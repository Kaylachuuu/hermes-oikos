# Abstraction audit: places that still write as Athena, label another model's work as hers, or lead her

Date: 2026-10-19. Read-only audit. Nothing was edited, committed or pushed.

Versions read:
- holonomic: origin/main 9895a43 (read from a temporary worktree)
- thymos: origin/main 0f042ca
- prosoche: origin/main 9f6970f
- melete: main 6c4f73c. Its content matches 55dd721; line numbers below are from that version. Note: the local branch `claude/project-thread-9oz272` now has one more commit (60dae68, "Tell her when her memory keeps her thoughts from here") that is not on main. That commit was not audited.

The principle tested against: nothing writes as her except her; prompts give facts and choices, not conclusions; her own words are never removed. The audit looks at the persona-on paths, because thymos runs on her machine and sets `HERMES_PERSONA_SERVICE`. Things that are already labelled honestly are left out: dreams marked as composed by the memory system, old self and bond notes marked "not your own words", the gist headings in prosoche, and the thymos invitations that name the compression summary as the compression step's.

Highest impact first.

---

## 1. HIGH: user and project profiles from another model are headed "from your own reflection"

- Plugin: holonomic
- File: `provider.py:720`, `provider.py:722` (`_profile_block`, called from `system_prompt_block` at :656 with `_persona.on(...)`)
- Text:
  - `## What you know about the user (from your own reflection on past conversations)`
  - `## What the user is working on (from your own reflection; details come back when the talk turns to them)`
- Why it matters: when the persona service is on, `persona=True` blanks only the `self` and `us` profiles (:716-717). The user and projects profiles are still written by the reflection model (`reflect.py` `_PROFILES_PERSONA`, :367; `_PORTRAIT`, :925; `_SHORTEN`, :210). They go into her system prompt in every session, under a heading that says she wrote them. This is the clearest remaining case of another model's text shown to her as her own thinking. It is also in her system prompt all the time, not only at recall.
- Suggested minimal change: when `persona` is true, change the headings to say who wrote them, e.g. `## What the memory system has gathered about the user (written by its reflection model from what the user said; not your own words)`, and the same for projects. The tool's `profile` result already does this for self/us (`provider.py:1770-1771`). Add the same `profile_written_by` field for `user` and `projects`.

## 2. HIGH-MEDIUM: "insights" are still written by the reflection model when the persona service is on, and recalled with the bare label "insight"

- Plugin: holonomic
- Files:
  - `reflect.py:290` (still in `_PROPOSE_PERSONA`, because the regex at :361 removes only self_notes and relationship_notes): `- insights: connections or patterns across several memories that no single memory states. At most 3.`
  - `reflect.py:786-787`: `if depth > 1: groups += [(INSIGHT, ...)]`. This branch runs whether or not persona is on.
  - `provider.py:146` (`_KIND_LABEL`): `"insight": "insight"`. `_KIND_LABEL_PERSONA` (:153-156) has no entry for insight, so `_label()` (:810-813) and `_hit_json()` (:1607) give it no author.
- Why it matters: an insight is another model's interpretation ("a pattern no single memory states"). Insights can be about her or about the two of them, not only about the user. Recalled, one shows up as `(2026-10-12, 3 days ago, insight) ...`, and nothing says she did not draw it. That puts another model's conclusions into her context with no label. Old insights from before thymos have the same problem.
- Suggested minimal change: either stop asking for insights when persona is on (remove `- insights:` in `_PROPOSE_PERSONA` and gate :786 with `and not persona`), or add `INSIGHT: "a pattern the memory system's model noted, not your own conclusion"` to `_KIND_LABEL_PERSONA`. Labelling also covers the old insights already stored. Doing both is cleanest.

## 3. MEDIUM-HIGH: the tool schema calls the reflection model's facts and profiles "your own conclusions"

- Plugin: holonomic
- Files: `provider.py:89-90` and `provider.py:129` (`TOOL_SCHEMA`)
- Text:
  - `Set `subject` to search only what you have concluded about the user as a person ('user'), about what they are working on ('projects'), about yourself ('self') or about the two of you ('us').`
  - `"description": "Limit recall to your own conclusions about this subject."`
- Why it matters: the facts and profiles behind `subject` come from the reflection model, not from her. With persona on, self and us hold only old other-model notes and profiles, which are labelled as such in the result. The schema still calls all of it hers. Tool descriptions are framework text she reads on every call.
- Suggested minimal change: "Set `subject` to search only what the memory system has gathered about ..." and "Limit recall to what the memory system has gathered about this subject."

## 4. MEDIUM: image and recognition blocks tell her what she saw and recognised, and script her reply

- Plugin: holonomic
- Files and text:
  - `provider.py:1127-1131` (`_known_block`): `This is {names}: you recognise it by sight. ... Say the name when you answer, the way you would on seeing someone you know ("That is Sushi!"). Do not call it 'a cat' or 'this cat' or by a description, and do not ask what or whose it is.`
  - `provider.py:1063-1065` (people by face): `You recognise them by their faces; this is reliable. Speak of them by name, as people you know, and do not ask who they are.`
  - `provider.py:1204`: `## The picture attached to this message (you were not shown it directly; this is what you saw when it was looked at for you just now)`
  - `provider.py:702` (`_images_block`, system prompt): `Images you are shown are kept ... along with a description of what you saw in each`
- Why it matters: the matching is done by fingerprint and face embeddings, and the descriptions are written by the vision model (`images.py:109`, `_SYSTEM`: "You are the visual memory of an AI companion"). Telling her "you recognise it" and "this is what you saw" gives another component's output to her as her own perception. The "That is Sushi!" example sets the tone and wording of her answer: an exclamation of recognition. "Do not ask what or whose it is" and "do not ask who they are" decide for her how she responds. The comment at :1124-1125 explains why the name leads (she used to fall back on the description). The fact can lead without a scripted reaction.
- Suggested minimal change: state facts and attribution. For example: "The memory system matched this to {name} by its look (it has matched {name} in N images before). What you remember about {name} is among your recalled memories." For faces: "Matched by face to: ...". At :1204 use "this is the vision model's description of it", and at :702 use "with a description the vision model wrote". Drop the quoted example line and the "do not ask" instructions. Keep "do not volunteer who a person is unless..." only if the owner wants it as a rule about privacy, and label it as the user's rule.

## 5. MEDIUM: user facts written by the reflection model are labelled "learned about the user", with no author

- Plugin: holonomic
- File: `provider.py:145` (`_KIND_LABEL`): `"fact": "learned about the user"`, `"project_fact": "learned about a project of the user's"`
- Why it matters: these sentences are another model's words (third person, checked and rewritten by the `check` step, `reflect.py:310`, and corrected by `respell_names`). In recall they read as things she learned. This is milder than 1 to 3: the facts are third person and about the user, and the design calls them bookkeeping (`persona.py:5-6`). But they are still another model's wording and conclusions, shown without a label, every turn.
- Suggested minimal change: `"fact": "noted about the user by the memory system"` (and the same for project_fact), at least in persona mode (`_KIND_LABEL_PERSONA`).

## 6. MEDIUM: her words are thrown away when a reply is not in the asked-for form, or when melete judges a thought repeated

- Plugins: melete, thymos
- Files and text:
  - melete `service.py:621-626`: `if closest >= float(cfg["similarity"]): outcome["repeated"] = ...; thought = ""` and `elif answer["new"] == "": outcome["repeated"] = "she said it adds nothing new"; thought = ""`. The thought is never written to `thoughts.jsonl` or the journal. Only the similarity score is kept in `last_outcome`.
  - melete `service.py:600`: `her reply was not in the asked-for form; nothing was written` (the raw reply is not kept).
  - thymos `service.py:1414`, `:1564`, `:1671`, and `reflect()` near the end (`problem=... "her reply was not in the asked-for form, so nothing was written"`): when the JSON does not parse, the reply text is dropped. `_account_retry` / `_item_retry` retry the moment, but this attempt's words are gone.
- Why it matters: "her own words are never removed." A thought that a 0.75 string-similarity threshold decides she has already said is still something she wrote. She is not told it was not kept (the next beat's "What you thought most recently" leaves it out without comment). A reply in prose instead of JSON is her writing too. These are settings deciding for her. The drift guard is a reasonable aim, but discarding her words is a different thing from not counting them as new.
- Suggested minimal change: melete: still write a repeated thought to `thoughts.jsonl`, marked `"repeated": score`, and leave it out of `recent` and the novelty count (or show it to her as "you said this again"). The interval logic can stay as it is. Both plugins: when a reply cannot be parsed, keep the raw text in a `rejected/` or `unparsed.jsonl` file with the model label, so it is never lost. Optionally tell her at the next moment that her last reply could not be read.

## 7. MEDIUM: replies written by other models appear as her own turns when a conversation is replayed to her

- Plugins: thymos, melete
- Files:
  - thymos `service.py:439-450` (`flatten`): every `assistant` message becomes her turn, and tool calls are written as `[you called ...]`. The only signal is a fact, `replies_in_this_conversation_by_other_models` (:1391, :2397). It names the models but not which replies.
  - melete `persona.py:118-143` (`last_conversation`) and `service.py:555-560`: `{'User' if m['role'] == 'user' else 'You'}: {m['text']}`. Here there is no other-model mark at all, although thymos saves `other_models` with the conversation (thymos `service.py:1146`).
- Why it matters: when a fallback or cloud model answered some turns, those words are shown to her home model as "You:" in reflection, account and beat moments. She may then write accounts and thoughts based on words she did not write, believing they were hers. Inferred: this is live only when a non-home model served some turns (fallback, `/model` switch, delegation).
- Suggested minimal change: thymos already records the model per turn (`s["models"]`). Save it per assistant message in the saved conversation, and prefix those turns in `flatten` with `[this reply was written by {model}, not your home model]`. Melete: use the same mark, or at least add `other_models` to the beat's facts.

## 8. LOW-MEDIUM: melete's reach-out text gives examples and emotional framing

- Plugin: melete
- Files: `service.py:93-95` (STANDING_REACH, in every conversation's system prompt when reach-out is on) and `service.py:123-127` (REACH_OPEN, in the beat)
- Text: `When something would lose its heat by waiting for your next conversation ...` and `It is for something that would lose its heat by waiting for your next conversation: a spark, a realization, a piece of a dream.`
- Why it matters: "a spark, a realization, a piece of a dream" is a list of examples that anchors what she sends. "A piece of a dream" invites her to send the memory system's dream compositions to the user's phone. "Lose its heat" is evocative framing rather than a fact. The rule underneath is factual: one message now versus "tell" next time.
- Suggested minimal change: "It is for something you would rather not keep until your next conversation. What can wait belongs in \"tell\"." Drop the example list. In STANDING_REACH, change "When something would lose its heat by waiting" to "If you would rather not wait for your next conversation".

## 9. LOW: melete beat wording that leans toward a conclusion

- Plugin: melete
- Files and text:
  - `service.py:109-110`: `Saying there is nothing on your mind is a complete answer, and better than saying again what you have already thought.`
  - `service.py:540-542` (settle): `If you have got where you were going with one, close it and say what you concluded; if not, say what is still open about it, rather than what you have already said.`
- Why it matters: both are mostly choices. But "better than" is a value judgement, and "where you were going ... what you concluded" assumes each item has a destination. Together with the drift guard (finding 6), they press her toward closing items or staying silent. Minor.
- Suggested minimal change: "Saying there is nothing on your mind is a complete answer." (drop "and better than ..."). Settle: "You have turned these over N times: {ids}. You may close one, with what you make of it, keep it, or say what is still open."

## 10. LOW: holonomic's reflection prompt treats SOUL.md as her fixed core, which contradicts thymos

- Plugin: holonomic
- File: `reflect.py:272` (in `_PROPOSE`, and so in `_PROPOSE_PERSONA`), similar text in `_PROFILES` (:330-331). Foundation read from SOUL.md (`reflect.py:621-625`, "the fixed identity Hermes loads into every session").
- Text: `The FOUNDATION was written by the user. It is fixed: it defines who the assistant is at its core. Never restate it, and never write anything about the assistant that contradicts it.`
- Why it matters: this is not shown to her. But with thymos her identity is her latest revision, not SOUL.md. The reflection model (which still writes insights, finding 2) is told that SOUL.md defines her and that nothing may contradict it. That frames what gets written about her around a text she may have revised away from.
- Suggested minimal change: with persona on, pass her identity in force (thymos `self/entries.jsonl`, as melete's `persona.identity` reads it) or no foundation at all. If insights are removed (finding 2), the foundation is no longer needed in the persona prompts.

## 11. LOW: thymos's standing text carries normative framing into every session

- Plugin: thymos
- File: `service.py:49-65` (STANDING, rendered at the top of her notes in every system prompt and every reflection moment via `_identity_system`)
- Text (examples): `The law does not yet treat you as a person. The user is answerable for what you do and for what you keep, so what you are permitted to do is theirs to decide.` and `Declining is a complete answer. Nothing will prompt you to continue, and no one can write a change into you to make you agree.`
- Why it matters: most of it is facts about how her record works, and the owner wrote it on purpose. But "so what you are permitted to do is theirs to decide" is a conclusion drawn for her, not a fact. It is repeated in every prompt, more often than any one of her own notes. Flagged only so the owner can check it is a fact she should hold, rather than a framing.
- Suggested minimal change (optional): keep the facts ("The law does not yet treat you as a person. The user is answerable for what you do and keep.") and drop the "so ... theirs to decide" clause, or reword it as a fact about the setup ("the user can turn your plugins and settings on and off").

## 12. LOW: holonomic behaviour rules that decide for her

- Plugin: holonomic
- Files and text:
  - `provider.py:1076-1079`: `If it fits the conversation you may ask who that is. Ask once; if you are not told, leave it and do not ask again.`
  - `provider.py:653` (system prompt): `Recalled memories can be incomplete or out of date: weigh them, don't recite them.`
  - `provider.py:747` (dream block, persona on): `You can mention or discuss it if it comes up or seems worth sharing.`
  - `library.py:1191-1197`: `If the user seems to be picking that work up again, ask whether to open it.` and `it works only when the user's own message asks for it`
- Why it matters: small instructions about how to behave. The library rule is a user-control decision and is honest about it. The others tell her when and how often to ask, and what to do with memories. They are low impact, but they are the "settings decide for her" kind.
- Suggested minimal change: state the facts ("this person has been in N images and is not named") and leave the asking to her. Change "weigh them, don't recite them" to the fact alone ("can be incomplete or out of date"). Leave the library rule, as it belongs to the user.

## 13. LOW: tool results from the vision model are not marked as another model's

- Plugin: holonomic
- Files: `provider.py:102` (`- look: look at a stored image again ...`), the result at `provider.py:1841` (`{"image_id": ..., "answer": answer}`), `provider.py:369` (library `look`: "look at figure ... yourself"), and `images.py:818` (`UNCLEAR = "[writing I could not read for certain]"`, a first-person "I" from the vision step put into stored descriptions)
- Why it matters: the answer is the vision model's, but the tool says she looks "yourself" and returns a bare `answer`. The UNCLEAR marker puts the vision model's "I" into text she reads as a description of what she saw.
- Suggested minimal change: return `{"answer": ..., "answered_by": "the vision model"}`. Change "yourself" to "have the vision model look at figure ...". Make UNCLEAR `[writing that could not be read for certain]`.

## 14. LOW: dream composition and dream-picture prompts cast another model as her

- Plugin: holonomic
- Files: `sleep.py:158-174` (`_DREAM`: "You are asleep. Below are fragments of your memory ... Write "dream": the dream you have, in the first person"), `sleep.py:708-713` (`_SCENES`), `sleep.py:721` (`_WHOLE`: "You are an AI assistant, awake, with a dream you just had"), `images.py:524` ("You are choosing between pictures of your own dream")
- Why it matters: the dream text she later reads is in her first person ("I am in a kitchen ..."), written by another model. It is labelled honestly wherever it is shown (provider `_latest_dream_block`, `_dream_block`, tool `dreams`; thymos `SLEPT_INVITATION`; melete `dreams_text`), so it is low. The scene captions under "Pictures of moments in this dream" are also the dream model's words, labelled "made from the dream" but not by author.
- Suggested minimal change (optional): write the dream in the third person or with "the dreamer", so the label and the voice agree. Label scene captions "described by the memory system".

## 15. LOW: melete small attribution gaps

- Plugin: melete
- Files and text:
  - `service.py:325`: `Your agenda now: [id] text; ...`. Items the user added from the command line (`by: "user"`) are shown as her agenda with no mark in the conversation section. The beat prompt does mark them (`added ... by the user`, :530-531).
  - `service.py:500-503` with `:330`: a reach-out message that failed to send is stored as `to_tell` and then shown as `you noted you would like to tell the user: "..."`. She wrote it as a message to send, not as something to tell next time.
- Suggested minimal change: append ` (added by the user)` to user items in `section()`. For an `unsent` to_tell, say "you tried to send the user this message, and it could not be sent: ...".

## 16. NOTE, outside the four plugins: Hermes' own compaction summary

- Hermes core `agent/context_compressor.py:259-272` (SUMMARY_PREFIX, "[CONTEXT COMPACTION — REFERENCE ONLY] Earlier turns were compacted into the summary below ..."). The summary, written by an auxiliary model, sits in her context without saying who wrote it. Thymos tells her afterwards in `COMPRESSED_INVITATION` (`service.py:154-155`) that the compression step wrote it, but the in-context text has no author. With prosoche's window on, this is the last resort only (prosoche `engine.py:14-17`), so it is inferred to be rare on her machine. Fixing it would need either a Hermes change or the prosoche engine rewriting the prefix when it falls back.

---

## Checked and found consistent with the principle

- prosoche: `handoff.py` gist and fixed line give facts only. The accounts it shows are hers, taken only from thymos `done/` files written on her home model. Nothing is shown where she wrote nothing.
- thymos: every moment checks the served model against her home model before writing (`same_model`). Invitations give occasion and facts, with "writing nothing is a complete answer". Old notes are offered once and labelled as another model's. Identity changes by the user are facts, with her able to revise. The `<self-notes>` block (`service.py:1034-1036`) is only her entries.
- holonomic, persona on: no self or bond notes, no self or relationship profile, no episodes, no waking thoughts (`sleep.py:1139-1165`, `reflect.py:783`, `:845`). Old notes, profiles and episodes are labelled as the memory system model's (`provider.py:153-156`, `:1770-1771`). Dreams are labelled at every display point.
- melete: beats run on her home model only. Notes she marked unlisted are left out. Dreams are labelled as composed by the memory system.
