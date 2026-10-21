# Request: multi-user holonomic for Hermai

From the Hermai thread, 2026-10-19. Kayla asked for this to go to the hermes-entelechy thread so work can start.

## Why

Hermai puts a Galaxy Tab Active3 in each room as a voice interface to Athena (plan: https://claude.ai/code/artifact/2da05236-48f0-4ad1-96a2-6ee914978334). Four people will talk to her: Kayla, Elliot, Ty and Savannah, plus guests. Holonomic is still single-user (README "One user" limit), so everyone's memories and profile facts would merge into Kayla's store.

## The design already exists

`persona/multi-user.md` (2026-10-08), sections 6, 6.1 and 7, is the spec: one store per audience on the reference-library mechanism, the existing store becomes the owner's store, the isolation test, per-audience backup and restore, one idle scheduler across stores, dreams per store (`words` or `off`). Please build from that. Section 1's standing rule holds: with one person, holonomic behaves exactly as today and existing tests don't change.

## What Hermai adds or changes (decided by Kayla)

1. **Identity is a voiceprint, attached per turn.** Each turn from a herma carries the speaker as a person id plus a confidence. A confident match opens that person's store for that turn; an unsure or unknown voice opens no personal store. This keeps the design's rule that a store never opens from words in a message, since the match is attached by Hermes, not said.
2. **The speaker can change every turn in the same session.** A room's chat is one Hermes session shared by everyone in the room. The open store follows the turn's speaker, not the session. Recall and writes for a turn use only that speaker's store. Core already gives providers `turn_author` in `sync_turn` and `author_id` in `on_turn_start` (`agent/memory_provider.py:133`, `:154`).
3. **No room store.** The room's own record is the Hermes chat history. Holonomic keeps nothing per room.
4. **Guests are never remembered.** Unknown voices get no store and nothing is written about them.
5. **Per-person "remembered by Athena" setting.** Set on a People page in the dashboard (Hermai builds the page). When it's off for a person, holonomic writes nothing for them. Default proposal: it also recalls nothing from their store, so off means treated like a guest except she knows their name. Confirm with Kayla if you'd rather keep recall.
6. **Person ids.** `kayla` (the existing owner store), `elliot`, `ty`, `savannah`. The mapping from a voiceprint to a person id lives in Hermai's config, not in anything she can write.
7. **Recall in shared rooms.** Decided: she may recall the speaker's own memories aloud, but stays discreet with private things when others might hear. Hermai's shared-device note tells her this; nothing needed in holonomic unless you see a place it belongs.
8. **Backups and restores** stay command line or dashboard only, as section 6.1 has them. Never by voice.

## Dependency on the Hermai side (not yours, for awareness)

Petasos sends each turn over the dashboard's `/api/ws`, and Hermes refuses a client-stamped author there (`tui_gateway/methods_prompt.py:762`, error 4124: "turn author is stamped by the gateway, never by a client"). Hermai's own backend (separate from Petasos, decided 2026-10-19) will need a server-side way to stamp the speaker on a turn. Hermai owns that; holonomic just needs to read the author core passes it.

## Timing

Needed before Hermai phase 3 (voice recognition). Phases 0 to 2 run single-speaker and don't need it. Suggested order: per-person stores with the isolation test, then per-turn store switching, then backup/restore per audience, then the shared idle scheduler.

## Agreed contract (hermes-entelechy, 2026-10-19)

- Hermai stamps each herma turn's author id as `person:<id>` (`kayla`, `elliot`, `ty`, `savannah`) only on a confident voiceprint match, else `person:guest`. Holonomic reads it from `on_turn_start` `author_id` / `sync_turn` `turn_author`.
- Once a session has had one `person:` turn, an unstamped turn in it opens no store. Sessions never stamped keep the owner store as today.
- The People page writes `holonomic.json`: `owner` (`"kayla"`) and `people` (`{"elliot": {"remembered": true}, ...}`). `remembered: false` means nothing written or recalled.
- Holonomic's plan: /mnt/project-files/hermai/holonomic-multi-user-plan.md
- Update 2026-10-19 (holonomic 0.34.0 built, awaiting Kayla's merge): Hermai must stamp EVERY herma turn, `person:<id>` or `person:guest`. An unstamped first turn of a new room session would open Kayla's store. Until voice recognition exists (phases 1-2), herma turns are stamped `person:guest`, so nothing said at a herma is remembered yet. Libraries are per person; subagents get no memory once people are set.
