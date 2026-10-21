# Holonomic multi-user: how it will be built (2026-10-19)

Answer to `holonomic-multi-user-request.md`. Built in holonomic, in this order. Defaults picked where the
request or `persona/multi-user.md` left a choice; each can change.

## The contract Hermai stamps
- A turn's author id `person:<id>` names who spoke (`person:kayla`, `person:elliot`, ...). Hermai stamps it only
  for a confident voiceprint match; an unsure or unknown voice is stamped `person:guest` (any id that is not a
  known person works the same).
- Turns with any other author (none on Desktop, a phone number on iMessage) open the owner's store, as today.
- A session that has had one `person:` turn is a shared session from then on: a later turn in it without a
  `person:` author opens no store, so a missed stamp can never fall back to Kayla's memories.

## Settings (holonomic.json; Hermai's People page writes them)
- `owner`: the person id of the existing store, `"kayla"`.
- `people`: `{"elliot": {"remembered": true}, ...}`. Empty (the default) means one person and nothing changes.
- `remembered: false`: nothing written for that person and nothing recalled; treated as a guest. (Request's
  proposed default; recall is not kept.)

## Stores
- The owner's store stays at `holonomic/`. Each other person's is `holonomic/people/<id>/`, a separate
  store with its own memories, profile, images, faces, fingerprints, conversations and dreams.
- A helper agent (subagent) gets no memory once more than one person is set up: it is not told who it works for.
- Libraries are per store: each person has their own. (A shared library could point into the owner's memories, so they are kept apart.)
- Her own words that holonomic takes in (thymos accounts, melete thoughts) stay in the owner's store for now.
- Isolation test: with one store open, nothing from another comes back from recall, profile, images, faces,
  fingerprints or dreams.

## Per turn
- The store follows the turn's speaker. Recall, writes and the memory tool use only that store. A guest turn
  gets a one-line note that memory is closed for this speaker, and nothing is stored.
- With `people` set, the profile and latest dream move out of the system prompt (built once per session,
  before anyone has spoken) into each turn's recalled context, for that turn's speaker.

## Backup and restore
- `hermes holonomic backup --audience ID` and `restore --audience ID SRC`; command line only, never by voice.
  A per-person backup is refused for any other person's store. The present store is set aside, and every
  other store is left byte-for-byte as it was. A full backup still takes every store.
- No restore tool for her (the request rules out voice). She is told at that person's next turn that their
  store was restored, from which backup, and how many memories are gone.

## One idle scheduler
- Person stores do no background work of their own. The owner's worker takes them in turn, after its own,
  with one quiet clock: nobody talking in any conversation.
- Person stores dream in words only (persona/multi-user.md section 7); the owner's keeps its current setting.
