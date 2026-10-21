# Holonomic, after thymos: recall by task, and memory for how to do things

Status: parked. Agreed by Kayla and Claude (Cowork session) on 2026-10-08, recorded the same day. **Not for
now.** Kayla wants the persona work finished first. Both changes touch `provider.py`, which the persona work
also changes, so doing them afterwards avoids editing the same code twice. Section 4 is Claude Code's check of
the "to check" items against the code.

Why they matter to her: the reason for holonomic is that a model should not have to hold everything in its
context, any more than a person holds everything in mind. It should be able to bring back the piece it needs
when it needs it. These two are the next steps in that direction. Do the first before the second: the second
depends on it.

What was read in the code (holonomic v0.24.0) is marked. The rest is a proposal.

The standing rule applies: with the new settings at their defaults, holonomic behaves as it does today and its
existing tests do not change.

## 1. Recall by what she is doing, not only by what was just said

**Today (read, `provider.py`, `prefetch`).** The recall cue is the latest message, plus the names of anything
recognised in an attached image: `engine.recall(words[:2000], k=k * 3, ...)`. A trivial message ("yes", "go
on") recalls nothing. So in the middle of a long piece of work, a short message brings back little, even though
what she is working on has not changed.

**Proposal.** A second cue for what the conversation has been about, recalled separately and merged in.

- **Where the cue comes from.** Facts only, nothing that needs a model to judge:
  - the last few user messages of this conversation (the provider already keeps the latest in `self._said`);
  - a `/goal`, if one is set and Hermes exposes it to providers (it does not; section 4);
  - the names of the libraries open in the conversation.
- **How it is used.** `engine.recall` already takes a text query, discrete `keys`, a prepared `vector`, or a mix
  (read, `engine.py`). Either a second `recall` call with the task cue, or one call with both; a second call
  keeps the two sets of scores apart, which makes the next rule simple.
- **It never crowds out the message.** Hits from the task cue fill at most `recall_task_k` of the `recall_k`
  places, and take the place of the weakest message hits, never extra places. The library notebook already
  does this for followed pointers (`library_follow_notes`: "in place of the weakest piece and never in
  addition").
- **It lets go when the subject changes.** If the latest message is far from the task cue (their vectors are
  already to hand), the task cue is dropped for that turn. Without this, an old task keeps dragging its
  memories into a new conversation topic.
- **Cost.** One more embedding call per turn, about 15 ms by the README's figures.
- **Settings.** `recall_task_k`, default `0` (off). `recall_task_messages`, how many recent user messages make
  the cue.
- **Seeing it work.** `hermes holonomic context` should say which hits came from the message and which from
  the task.

**Build it as a general "extra cues" path.** The persona design has an optional second cue too: the text of her
latest entry, for mood-congruent recall, off by default. Same mechanism, different source. One list of
`(cue text, most places it may take, label)` in `prefetch` serves both.

## 2. Memory for how to do things

**Today.** Holonomic keeps what was said and what is true: conversation, facts about the user and projects,
accounts of past conversations. The one place it keeps *how something was done* is the library notebook (read,
`notebook.py`): her notes on what worked, what failed and the correction, bound on the plates to the passages
they are about. Kayla built the notebook for exactly this reason and wants it extended to her ordinary memory.

**How Kayla puts it:** skills are procedures, libraries are knowledge, lessons are experience. (With the
persona work there is a fourth: who she is.) That is also why lessons need a rule the other two do not. A
procedure or a reference is right or wrong when it is written. Experience is provisional: the next attempt can
show that what was learned was incomplete, or true of one case only. Worth using in the README.

**Proposal.** The same kind of note, in her own store, bound to situations in place of passages.

- **Reuse what the notebook already settles** (all read in `notebook.py`):
  - a realm of its own (`notebook` there), so these notes are recalled with ordinary memory or without, and
    never mixed into factual recall by accident;
  - where a note came from, carried into recall: `inference` (her reasoning, not checked), `tested` (must say
    what was run and what happened), `user` (must carry the user's words, checked against what they wrote).
    She cannot mark a guess as a result;
  - a better note replaces the old one, which is retired and kept on record; a note that says what another
    already says is refused.
- **What a note is bound to.** A library note is bound to passages. A lesson is bound to two things:
  - the memories of the conversation it came out of, on the plates, so either brings up the other;
  - a short statement of *when it applies* ("writing a large file through Git Bash on Windows"), with its keys
    (`extract_keys`: tool names, file types, error text).
- **When it comes up.** By the task cue from part 1, matched against "when it applies". This is why part 1
  comes first: a lesson has to be found by what she is doing, and what was just said is usually "ok, try it".
- **Who writes it.** She does, through the memory tool, in her own words, as with library notes. The
  reflection model does not write lessons for her.
- **It does not fade with time.** A lesson is not conversation. It changes by being contradicted: a `tested`
  note outranks an `inference`, and a result that goes against a note marks it as questioned until she revises
  or confirms it. Two things like this exist today: facts about the user are replaced when the user contradicts
  them, and a library note is flagged when its passage changes. "Questioned" for lessons is new.
- **She marks a lesson as questioned herself,** through the memory tool, the way she writes one. The plugin
  does not infer it from a failed command: that would mean guessing why something failed, which is the kind of
  judgement kept out of the framework elsewhere. A wrong lesson recalled often misleads her often, so this rule
  matters as much as the lessons do.
- **How it is shown.** In the injected block, labelled with where it came from, in its own small budget, as
  library notes are (`library_notes_k`, `library_notes_context_chars`).
- **Settings.** Off by default.

**The first thing to check: Hermes skills.** Hermes already has procedural memory of a kind. Skills are whole
procedures in files, and background review (`agent/background_review.py`) asks after a turn whether a skill or
memory should be saved. This must not become a second copy of that. The intended difference:

| | Hermes skill | Lesson |
|---|---|---|
| Size | A whole procedure | One correction or finding, a few sentences |
| Found by | Name, from the skills index | Association with what she is doing |
| Says where it came from | No | Yes: inference, tested, or the user's words |
| Tied to | Nothing | The conversation it came out of |

If reading the skills code shows the gap is smaller than this table claims, say so before building.

### Lessons are how a skill finds out it has stopped working

A procedure that works one day may not work the next. What Hermes has for that today (from the docs,
`website/docs/user-guide/features/curator.md` and `skills.md`; not run):

- She can create, patch and delete her own skills with `skill_manage`.
- The curator runs in the background when she has been idle (about weekly). It tracks how often each
  agent-created skill is viewed, used and patched. A skill unused for 14 days is marked stale; after 30 it is
  archived, never deleted. A periodic model review proposes merging near-duplicates and patching drift.

**The gap:** stale there means unused. A skill she uses daily that stopped working last week looks healthy.
Nothing records that a procedure was followed and did not work.

That record is what a lesson is. So:

- **A lesson can be bound to a skill.** When she follows a skill and a step fails, she writes a lesson tied to
  it: what she ran, what happened, what worked instead, marked `tested`. In the record this is the skill's name
  among the lesson's keys, so the plates bring the lesson up whenever the skill is in play.
- **The lesson comes up with the skill.** The next time she uses that skill she is given the lesson with it,
  and does not repeat the failure, even before the skill itself is fixed. What triggers this: see section 4.
- **Several lessons against one skill are a reason to rewrite it.** One is a footnote. When a skill has more
  than a few standing lessons, she is told so as a fact ("this skill has 4 lessons against it"), and she decides
  whether to revise it with `skill_manage`. Holonomic never edits a skill.
- **After she revises a skill,** the lessons that led to the change are not deleted. They are marked as
  answered by that revision and stop coming up, the way a retired note stays on record.

To check: whether the curator's model review could or should be shown the lessons against a skill. It would make
that review better informed, but it is another model changing her procedures, and with the persona service on
the preference is that she does it herself.

## 3. Open questions

1. **Several people.** A lesson learned while working with one person is useful with another, but may carry the
   first person's project details. Default under the audience-store design (`persona/multi-user.md`): it stays
   in the store of the audience it was learned with. Whether she can mark one as general is undecided.
2. **Does a `/goal` reach memory providers?** It does not (section 4). The task cue is recent messages and open
   libraries only, unless core is changed to pass it.
3. **How far is "far"** for dropping the task cue when the subject changes. Needs trying on real conversations.
4. **How many lessons against a skill** before she is told it may need rewriting.
5. **Lessons and the persona service.** A lesson is knowledge, not identity, so it stays in holonomic and is not
   one of her persona entries. Stated here so it is a decision and not an accident.

**Also noted, no code needed.** Kayla may watch how full Athena's context gets over a few typical sessions
(`/context` in a conversation) to see whether the 64k setting can come down. Where she would miss something at a
smaller size is a list of what recall is not bringing back, which is the best input for part 1.

## 4. Checked against the code (Claude Code, 2026-10-08)

Holonomic v0.24.0 and `NousResearch/hermes-agent` main.

**The task cue does not half-exist.** `unfinished_business` (`sleep.py:287`) is consolidation's list of
conversations that have gone quiet with memories not yet summarised; it says nothing about what she is doing
now. The "libraries the last conversation used" line (`library.py:1191`) is a hint in the system prompt about
which libraries to offer, not a recall cue. Part 1 is new work.

**Core skips memory entirely on bare acknowledgements, before holonomic is asked.**
`_memory_turn_start_and_prefetch` (`agent/turn_context.py:895`) calls `prefetch_all` only when
`is_trivial_prompt(query)` is false. Trivial means the whole message is one of a fixed list ("yes", "ok",
"continue", "go ahead", "do it", "proceed", "next", ...), optionally with punctuation
(`agent/memory_provider.py:66`). "ok, try it" is not trivial and does reach holonomic; "do it" and "continue"
do not. So part 1 cannot help on exactly the turns that motivate it unless core changes. Two ways:
- core calls `prefetch` on trivial turns too, for providers that opt in (an attribute like the existing
  `pre_compress_checkpoint_api_version`), passing a `trivial=True` flag so holonomic recalls by the task cue
  alone; or
- holonomic implements `queue_prefetch` (core calls it after every turn, `run_agent.py:916`) and caches a
  task-cue recall for the next turn. That still needs core to deliver the cache on a trivial turn, so it is
  the first option with extra steps. The first is the one to propose upstream.
Holonomic's own `prefetch` also returns early on trivial prompts (`provider.py:669`); that line changes too.

**Decided after the check (Kayla):** do not propose the upstream flag yet. Core calls `on_turn_start` with the
message on every turn, before the trivial check, so holonomic can keep the task cue current on every turn. What
it cannot do on a trivial turn is inject anything, and that loss is small: on "yes" or "continue" she is carrying
on with the same work, and what was recalled the turn before is still in her context. Propose the flag only when
a case shows it clearly hurts. The same gap was a live bug in the library "yes" path; fixed in v0.24.1
(PR #1), which keeps the message in `on_turn_start`.

**`/goal` does not reach memory providers.** Neither `MemoryProvider` nor `MemoryManager` mentions goals, and the
turn passes only the message text. Using a goal as a cue needs core to pass it (for example as a kwarg on
`prefetch`, delivered by signature like `turn_author`).

**The skills table holds up, and the gap is real.** Skill telemetry (`tools/skill_usage.py`) records counts and
times of use, views and patches (`use_count`, `view_count`, `patch_count`, `last_used_at`, ...), nothing about
outcome. Nothing records that a procedure was followed and failed, which is the gap the note describes.

**What triggers a lesson to come up with its skill.** `on_skill_lifecycle` fires after a skill is `loaded`,
`patched`/`edited`, `created` or `installed` (`tools/skill_usage.py:508-542`), with `action`, `skill_name`,
`provenance`, `task_id`, `session_id`, `use_count`, `reused` and `reuse_after_patch`. A `loaded` event is the
trigger. Two cautions:
- A skill is usually loaded mid-turn, after `prefetch` has run, so a lesson found then reaches her next turn,
  not in the turn where she follows the skill. Delivering it in the same turn would need the lesson attached
  to the skill load's tool result, which is core's to offer.
- The hook is registered with `register_hook`, and a memory provider's hooks are the fallback group that the
  general loader replaces with its own pass, bound to a second, inert provider instance (the same two-instance
  problem as the idle signal, `persona-provider.md` section 8). The same fix applies: an optional
  `MemoryProvider` method (`on_skill_event`) that core calls on the active provider. Until then, the skill name
  in the task cue (it appears in the conversation when she loads one) is the fallback, as the note suggests.

## 5. Backing up one library (Kayla, 2026-10-08)

`hermes holonomic backup --library NAME` and `hermes holonomic restore --library NAME SRC`. Each library is
already its own store in its own folder (`library.py:96`), so this is the existing backup and restore pointed
at that folder. A library is nobody's memory, so restoring one needs no occasion and tells her nothing beyond
what a changed library already does. Restoring into a library of another name is refused unless asked for.
It does not touch `provider.py`, so unlike the rest of this note it need not wait for thymos.

The per-person version is designed in `persona/multi-user.md` 6.1 and waits for multi-user.


## 6. Ownership and access rights for libraries (Kayla, 2026-10-08)

An idea to revisit, not designed yet. Libraries could have an owner and access rights: who may open a
library, who may add to or change it, and who may back it up or restore it. This matters once more than one
person talks to her (`persona/multi-user.md`), where today any conversation can open any library. Kayla wants
the focus on thymos first, so this waits.

## 7. A smaller context, with recall standing in for it (Kayla, 2026-10-08)

Kayla's goal: keep her context window small by clearing it often, while she can still bring back what was
discussed recently when it bears on the current subject. Not for now.

**Most of it exists (read, holonomic 0.24.1).** Recall skips memories of the current conversation that are
still in the context window (`provider.py:652`, `_visible`). When Hermes compresses the conversation,
`on_pre_compress` marks the time (`provider.py:1120`), and from then on the older turns of that conversation
can be recalled again like any other memory. So a smaller context setting (Hermes' compression threshold,
or the 64k model setting coming down) already means: compress more often, and let recall bring back the parts
that matter. No separate conversations are needed: every message is stored as it happens, whichever
conversation it is in.

**What would make it work well:**
- Part 1 of this note, recall by what she is doing. After a compression, a short message ("ok, and the
  other one?") recalls little by itself; the task cue is what brings the earlier part back.
- A summary of the stretch that was compressed, written at `on_pre_compress` as an episode of its own, so
  that what left the window has a short form that recall can find, not only the raw turns. Today episodes
  are written only after 30 minutes of quiet (`sleep.py:287`).
- Hermes' own compression summary is written by its auxiliary model. With the persona service on, that is
  another model summarising her conversation into her context (persona design, section 9.6). To check when
  this is built.

To try first, with no code: lower the compression threshold in Hermes' settings and see what recall misses.
That list is the best input for the items above.

## 8. Small fix: a closed Hermes can show as still sleeping (found 2026-10-08)

`hermes holonomic sleep status` counts a worker as running if its last state says it was in the middle of
something (`cli.py:345`, `or st.get("doing")`), however long ago that state was written. A worker does not
write its state while a sleep step runs, so this keeps a long sleep from looking dead. But a Hermes that is
closed during a sleep then shows as "it is sleeping now" until its file is a day old. Seen on Kayla's PC: a
process up 2 minutes, last heard from 113 minutes earlier, shown as sleeping for 113 minutes.

Nothing is harmed: the lock that keeps two sleeps apart is per process (`reflect.py`, `_busy`), and the next
sleep runs in whichever Hermes is open. Fix: write the worker's state from a timer during long steps too, then
drop the `doing` exception; or cap it at the longest a sleep may take.
