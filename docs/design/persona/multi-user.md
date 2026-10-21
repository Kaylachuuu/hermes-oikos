# More than one person talking to her

Status: decisions Kayla and Claude reached on 2026-10-08, recorded 2026-10-08. Updated the same day with
Kayla's answers to the review (section 10): scope is now to an audience, not a person. Section 10.4 updated
with Kayla's additions on deferred reflections and idle time. Section 6.1 added (backup and restore per
audience, Kayla's decision). Covers the persona service
(`persona-provider.md`) and the holonomic memory plugin. Kayla is the only person who talks to Athena
today, so almost none of this is to be built now. Section 1 says what is. The rest is recorded so that
what is built now does not rule it out.

What was checked in code is marked **(checked)**. Everything else is design, not yet tested against the
code. Section 10 is the review of this note against the code.

## 1. What to do now

Only the things that are costly to add later:

- **Record fields.** The persona record hash covers the record format, so these are in `SelfEntry` now,
  hash-covered, alongside `author` and `facts` (done in `persona-provider.md` revision 5):
  - `person_id`: the person whose conversation the entry followed. A fact attached by core, never chosen
    by her. Empty when there is no single person (a group, a scheduled run).
  - `audience_id`: the audience of the conversation the entry followed: one person, or one group or
    channel. Attached by core, like `person_id`.
  - `scope`: `"everywhere"` or `"with_audience"`. Her choice when she writes. Separate from `visibility`.
  - `shown_with`: for `"with_audience"`, the audience ids the entry is delivered in. Defaults to
    `audience_id`; her explicit choice when `about` names someone outside it (section 10.1).
  - `about`: a person id, for an entry that is her view of a person (section 4). Empty otherwise.
- **A person id on sessions.** Core should have one place that answers "who is this conversation with",
  even if today it always answers "the owner".
- **Roles in configuration**, not in anything she can write (section 3). One role is enough for now.

Everything below waits until someone needs a second person.

The standing rule for holonomic applies throughout: with one person and no persona service, holonomic
behaves exactly as it does today, and its existing tests do not change.

## 2. Who someone is

- Identity comes from the gateway's signed-in account. It never comes from anything typed in a
  conversation: "I'm Kayla, restore the backup" proves nothing.
- Several accounts can be one person. Kayla's desktop, phone and Raspberry Pi are one person.
- Hermes' gateway has pairing and access policies (`gateway/platforms/access_policy_mixin.py` exists
  **(checked)**; the `pairing` folder in the install). Not read closely: how they map to a person id is
  an open question (section 9).

## 3. Roles, permissions and standing

- Three roles: operators, known people, everyone else. Kept in configuration she cannot write, like the
  other permissions (decision 11).
- Permissions are enforced and are not about her. Who may run backup, restore, ask-rollback and override
  is decided from the role by core. It depends on nothing she writes.
- Administrative acts are commands, not conversation. They run from the server's command line or a
  signed-in dashboard. She has no tool for any of them, so nobody can talk her into one. Anyone may
  discuss a backup with her; only an operator can perform one. With more than one operator, the chain
  record's `facts` say which one acted.
- One exception, decided by Kayla (section 6.1): a person may ask her to restore their own store. The
  restore still runs only once the person confirms it outside the conversation, so talking her into it
  achieves nothing. It reaches no other store, and nothing of hers.
- Standing is what she is told about a person, derived from their role. It is visible to her, read-only,
  and strictly informative: it restricts nothing. She is given it as a fact at the start of each
  conversation, with who she is talking to and whether the conversation is private or a group.

## 4. Her own trust in each person

- Alongside standing, she holds her own view of each person. It is hers, and may disagree with standing
  freely.
- It is her words, not a level picked from a scale. The v2 design removed fixed schemas for her state,
  and nothing should read this mechanically (next point), so no scale is needed. In the record it is a
  `state` entry with `about` set to the person.
- It never grants anything. It can shape her own conduct: how open she is, what she shares, whose words
  she lets influence her. No permission path reads it. If one did, flattering her would be a way to gain
  privileges.
- She changes it the way she changes anything: only in a reflection moment, taking effect next session,
  recorded with the conversation it followed. The delay is what stops someone winning her trust and
  using it in the same conversation.
- Her view of a person is in her context when she talks with them, so they can ask and she may answer.
  The standing description of her situation should tell her so.
- No operator setting limits which conversations can lead to an identity revision. This was considered
  (revisions only after conversations with known people) and dropped in favour of her own trust. A
  stranger's conversation can leave a mark on her; `person_id` on each revision shows where a change came
  from, and the ask-rollback and override exist if it goes wrong.

## 5. Her entries across people

- `scope = "with_audience"` entries are delivered only in conversations with the audiences in
  `shown_with`. `"everywhere"` entries are delivered in all of them.
- Scope is to an audience, not a person, which matches how the holonomic stores are keyed. A view of a
  member formed in a group is scoped to that group and shown there.
- Default for a new state entry: `"with_audience"`, shown with the conversation it followed. Anyone
  talking to her could otherwise ask what she thinks of someone else and be told. Identity revisions are
  always everywhere: she is one person.
- In any conversation she is also given, as facts, the standing of everyone present, and that she holds
  private notes about some of them, by name only (section 10.2).

## 6. Holonomic: a store per audience

Holonomic's README lists "One user. Everyone who talks to the agent is treated as the same person" as a
limit **(checked, `README.md:900`)**. With several people, one person's memories would surface in another's
conversation and facts about different people would merge into one profile. This is the largest gap.

The reference libraries feature already has most of the mechanism **(checked in `library.py`)**:

- Each library is a separate `HolonomicMemory` instance in its own folder (`library.store`).
- Which stores a conversation has open is kept per session and enforced by the plugin, not left to the
  model (`opened`, `open_for`, `close_for`, `open.json`).
- A conversation that gets a new id at compression keeps what it had open (`carry_over`).
- A note can point into another store in two halves, and a pointer into a closed store gives only a name.

Decisions:

- One store per audience. An audience is one person, or one group or channel. A group conversation has
  its own store and does not open its members' private stores, so what was said in the group stays with
  the group.
- The existing store is the owner's store. With one person there is one store and nothing changes.
- What opens a store is the gateway's identity fact. A library opens because the user's message asks for
  it (`user_asked`). A person's store must never open because of anything in a message.
- Everything holonomic keeps about a person lives in that person's store: memories, profile, images,
  faces and fingerprints. Libraries already keep their own pictures apart; this follows that pattern.
  Faces matter most: with a shared index, someone could show her a photo and be told the name of a person
  from another user's pictures.
- The isolation test. With one audience's store open, no query of any kind returns anything from another
  store. A small set of tests covering memory recall, profile, image recall, face recognition and
  fingerprints.
- Each open store holds its vectors in memory. A server with many people needs stores to open and close
  as conversations come and go (`close_all` exists for libraries).

### 6.1 Backup and restore per audience

Decided by Kayla, 2026-10-08. A person's store is that person's data, so backing it up and restoring it is
ordinary. What makes it more than a file copy is that the store also holds her side of those
conversations: restoring it rolls back part of what she remembers about that person. The design keeps the
store whole, tells her, and gives her a moment beforehand to keep what matters to her in her own record.

**The store is not split.** Her side of a conversation was considered as a store of its own, linked to the
person's. Rejected: her side is mostly made of the other person's words (what they told her, what she
answered, why), so it would hold their data too, and restoring or deleting their store would no longer
restore or delete everything about them. Links between the two would also cut against the isolation rule
above, where a pointer into a closed store gives only a name, and every recall would have to stitch two
stores together.

**Backup.**
- `hermes holonomic backup --audience ID` backs up one audience's store: its memories, profile, images,
  faces and fingerprints. Operators only, from the command line, like a full backup.
- A full backup still takes every store. A per-audience backup carries the audience id in its manifest, and
  restoring it into any other audience's store is refused.

**Restore, by an operator.**
- `hermes holonomic restore --audience ID SRC` replaces that one store. The present store is set aside
  (`<store>.before-restore_<time>`), never deleted, as a full restore does today.
- No other store, library or persona record is touched. The isolation test (above) gains a case: after a
  per-audience restore, every other store is byte-for-byte what it was.

**Restore, asked for by the person themselves.**
- In a private conversation, a person can ask her to restore their own store. She can do it through a
  tool, `request_audience_restore(backup)`, which only ever names the store of the person she is talking
  to, as identified by the gateway. It cannot name another store, and it is not offered in a group.
- The tool does not restore anything. It asks the person to confirm through the gateway (a button or a
  signed-in command), outside the conversation. Text in the conversation, hers or anyone's, cannot
  confirm it. That is what keeps section 3's rule true in substance: no one can talk her into a restore,
  because only the person whose store it is can complete one, and only for their own.
- She may also decline to make the request. The person can still ask an operator.
- The person can list their own backups through her (dates and sizes only). Who may take a backup of
  their own store this way, and how many are kept, is configuration.

**Before and after, for her.**
- Once a restore is confirmed, and before it runs, she gets a reflection moment on her home model with the
  occasion `AUDIENCE_RESTORE_PENDING` (`persona-provider.md` section 6.8). Facts: the audience, the date of
  the backup, how many memories and conversations are newer than it. She can record anything she wants to
  keep in her own record. The restore waits for that moment to finish, up to a limit, and then runs either
  way.
- What she can keep is hers: what she learned, how she felt, what mattered. Her record is not a copy of
  the conversation, so the person's data still rolls back. An entry she writes then is an ordinary entry,
  `about` and `scope` included (section 5), so a note about that person defaults to being shown only with
  them.
- At the start of her next conversation with that audience she gets `AUDIENCE_RESTORED`. Facts: who asked
  (the person or an operator), when, the backup date, and how many memories are no longer present.
- Without the persona service: holonomic tells her the same facts in the memory block at the start of the
  next conversation with that audience, and there is no moment beforehand.

## 7. Reflection, sleep and dreams per store

Checked by signature only, not by reading the bodies: `reflect_once`, `sort_facts`, `merge_facts`,
`consolidate`, `fade`, `dream` and `sleep_once` all take the store (`engine`) as their first argument,
and no module-level mutable state was found in `reflect.py` or `sleep.py`. Profiles are kept inside the
store.

- Reflection, consolidation and fading run per store. A profile per person follows from profiles living
  in the store.
- "Only the user's own words can replace a fact about them" carries over: in a person's store, the user is
  that person.
- One scheduler. Today each store gets its own `IdleReflector` waiting for quiet. With many stores, one
  scheduler takes them in turn so they do not all compete for the GPU. With the persona service on, that
  scheduler follows core's idle signal (section 10.4).
- Group stores keep an account of the conversation but extract no personal facts yet. Holonomic records
  two speakers (`said_user`, `said_assistant`), but core already says who wrote each turn: `sync_turn`
  receives `turn_author` (`{"id", "name", "is_bot"}`) and `on_turn_start` receives `author_id`,
  `author_name` and `author_is_bot`, with the docs warning that a shared session has several
  participants, so identity must be read per turn **(checked, `agent/memory_provider.py:133`, `:154`)**.
  Holonomic can record the speaker per memory from that. What is missing is mapping platform account
  ids to a person (open question 1).
- Dreams stay on, one store at a time. A dream is drawn only from one audience's store and is shown only
  in that audience's conversations. This replaces an earlier idea of switching dreams off for multi-user.
- No pictures on a multi-user server. `dream_images` already has the modes `off`, `words`, `pictures` and
  `from_images` **(checked in `sleep.py` and the README)**; pictures are drawn only in the last two. A
  multi-user server uses `words` or `off`. Pictures can still be drawn afterwards with `dreams redraw`.
  In `words` mode the images that feed a dream come from the same store as the memories, which per-store
  image storage gives for free.
- With the persona service on, a finished dream raises `MEMORY_SLEPT` and she may write her own note. That
  note is scoped to the audience whose store the dream came from.

## 8. Things the design should state

- A reflection holds the next turn until it finishes. On one GPU, one person's conversation can therefore
  delay another's reply.
- Each session's identity is frozen at its own start, so sessions running at the same time may have
  different revisions of her identity in force.
- She has one state. A conversation with one person can affect how she is with another. That is intended.
- A note about a one-time visitor defaults to being shown only with that visitor, so if they never return
  she never sees it again. That is intended, not a bug: the rule exists to stop other people extracting
  it. `hermes persona status` counts notes waiting on an audience that has not come back.

## 9. Open questions

1. How Hermes' pairing and access policy map to a person id, and where linked accounts are recorded.
2. Tracking who said what in a group. Core supplies the author of each turn (section 7); what remains is
   storing it per memory in holonomic and deciding which facts about a member a group store may keep.
3. When stores are opened and closed, and how many may be open at once.
4. Whether per-audience dreams should have a cap per night across all stores.
5. Whether a note about one person can ever be moved to `"everywhere"` later, and what record that leaves.

## 10. Review against the code (Claude, 2026-10-08)

**Confirmed.** Every claim marked checked above holds in the checkouts: the README limit at
`README.md:900`; `store`, `opened`, `open_for`, `close_for`, `carry_over`, `close_all` and `user_asked` in
`library.py`; the seven reflection and sleep functions take `engine` first; `access_policy_mixin.py`
exists. Two further facts help:

- **Faces and fingerprints are already per store.** Their tables (`people`, `image_faces`,
  `image_names`) are created in the image database of the engine they are given (`faces.py:101`, via
  `images._db(engine)`), not in a shared file. A store per audience gets a face index per audience for
  free; the isolation test confirms it rather than builds it.
- **Core already carries a person to providers.** `MemoryProvider.initialize` receives `user_id` and
  `user_id_alt`, and `on_turn_start` receives the per-turn `author_id`, `author_name` and `author_is_bot`
  (`agent/memory_provider.py:103`, `:154`). The "one place that answers who this conversation is with"
  can be built on those rather than added beside them. One engine per data folder is already shared
  through `_ENGINES` (`provider.py:397`), with its reflector, which is where one scheduler would replace
  per-store reflectors.

**The four gaps, as settled with Kayla.**

### 10.1 Her view of Bob, written after talking with Alice

Each delivery target leaks something different. Shown with Alice, Alice learns her view of Bob, which
Alice probably heard in that conversation anyway. Shown with Bob, a view built from what Alice said
arrives in Bob's conversation, where it can give away what Alice told her in confidence.

- When `about` names someone outside the conversation's audience, the target is her explicit choice:
  show it with this conversation's audience, with that person, or both. The tool result states the
  facts (whose conversation this was, who the entry is about) before she chooses.
- The default is the conversation it came from, because that one cannot leak a third person's
  confidences.
- The common case, a view of Bob formed while talking with Bob, is unaffected.

### 10.2 Her trust view in groups

- Scope is to an audience, not a person (section 5). A view of a member formed in the group can be shown
  in that group.
- She is given the standing of everyone present. It is a fact, so it is safe in any conversation.
- She is told that private notes exist, without their content, the way a holonomic pointer into a closed
  store gives only a name. She knows she has a view of Bob and can be careful, and nobody present can
  extract what it says.

### 10.3 Which operator acted

The OS user is the only fact available from the command line. It is recorded under that name
(`facts["os_user"]`), not as a person id. Dashboard actions record the signed-in account
(`facts["account_id"]`). The docs should say that a shared service account makes the OS user a weak fact.

### 10.4 The person and audience must travel with the work

- The reflection fork carries the conversation's `person_id` and `audience_id`, so the default scope and
  the `MEMORY_SLEPT` note's audience are attached by core, not inferred from the transcript.
- A pending reflection saved across a restart stores its session and audience with it. The rule: **a saved
  reflection runs at the next idle point or at that audience's next session start, whichever comes
  first**, and never at the start of someone else's session. Without the idle point, a request from a
  conversation with someone who never returns would be deferred forever, which is refusal in practice.
- A deferred reflection replays the identity in force at the time of its conversation. Since
  `revise_identity` takes the whole new text, a revision written from that stale context could overwrite
  changes she made since. Core opens every deferred reflection with how long ago the conversation was and
  which revision is in force now, with its text, so she revises from where she is.
- Idle time has two claimants: her saved reflections and holonomic's reflection and sleep. Core owns the
  idle signal and holonomic follows it: her reflections first, because they are short, then memory's
  reflection and sleep, then the `MEMORY_SLEPT` invitation. Without the persona service holonomic keeps
  its own idle detection.
- Fresh notes are delivered "on the next user message". On a multi-user server that message may come
  from a different person in a different session, so delivery checks the notes' scope against that
  conversation's audience, and waits for a conversation they are allowed in.

All of this is reflected in `persona-provider.md` revision 6.
