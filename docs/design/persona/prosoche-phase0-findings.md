# Prosoche phase 0: what was measured, 2026-10-11

What the first measurements on Kayla's machine showed, and what was decided. Kept so the question of request
size can be picked up again without measuring from scratch.

Setup: Hermes fork (main at aeff051a18), model `gemma4-64k:latest` on Ollama (window 65,536), prosoche 0.1.1
from `claude/project-thread-9oz272` with `context.engine: prosoche`. Sources: `hermes prosoche status` output,
`plugin-data/prosoche/measure.jsonl`, and two request dumps (`HERMES_DUMP_REQUESTS=1`) taken 2026-10-11 03:38
(new chat) and 03:45 (chat `...850e56`) local time. Token figures for parts are estimates at about four
characters a token; totals marked "provider" are Ollama's own count.

## The answer to phase 0's question

The fixed part of every request (system prompt and tool definitions) is most of it.

| | Tokens |
|---|---|
| A typical request (provider) | about 16,900 |
| Fixed part: system prompt | about 9,000 (36,300 characters) |
| Fixed part: tool definitions | about 5,100 to 7,100 by the provider; about 8,100 by characters |
| Fixed part as a share of a typical request | about 90% |
| The conversation itself, in the chats measured | 2,800 to 7,600 |
| Memory blocks (recall attached to turns) | up to 3,200, about a quarter of the estimate |

No conversation measured so far reached `window_high` (16,000 tokens of earlier turns), so the window never
would have stepped. It only matters in long conversations. With Hermes compressing at half the window (about
32,000), the window would step at roughly the same point compression fires today, which is what it is meant to
replace: release whole turns instead of summarising them.

## What fills the system prompt (new chat, 03:38)

| Part | Characters | About (tokens) |
|---|---|---|
| Her identity, SOUL.md (Who You Are through Core Principles) | 11,400 | 2,850 |
| Holonomic: memory header, profiles, what the user is working on, latest dream, reference libraries | 7,800 | 1,950 |
| of which the latest dream | 2,800 | 700 |
| of which the reference libraries note | 2,050 | 500 |
| Skills section and the list of available skills | 6,200 | 1,550 |
| Hermes' operating rules (finishing the job, parallel calls, tool-use enforcement, operational rules) | 5,600 | 1,400 |
| Thymos section | 4,000 | 1,000 |
| Hermes runtime environment | 1,900 | 480 |

The system prompt was the same size in both dumps. An earlier status showed 6,490 tokens for the head of one
resumed chat; most likely a resumed session keeps the system prompt stored when it began (inferred, not checked).

## What fills the tool definitions (14 tools, 32,400 characters)

| Tool | Characters |
|---|---|
| holonomic_library | 6,300 |
| holonomic_memory | 6,200 |
| terminal | 4,850 |
| skill_manage | 3,700 |
| search_files | 2,300 |
| tool_search | 2,150 |
| read_file, write_file, patch | 1,000 to 1,200 each |
| skill_view, tool_call, vision_analyze, and the rest | under 1,000 each |

## Decided, 2026-10-11

Kayla: no reduction for now.

- **Keep the coding tools** (terminal, files, skills). She plans to use them with Athena.
- **Do not shorten holonomic's tool descriptions or prompt sections.** Several were tuned so Athena would use
  them correctly; shortening risks undoing that.

If size has to come down later, the candidates, roughly in order of least risk:
1. The latest dream in holonomic's section (700 tokens): could be shown on request rather than every time.
2. The reference libraries note (500): only needed when libraries exist or are in use.
3. The tuned tool descriptions, only with care and with Athena checked against the same cases they were tuned on.
4. Toolsets off for conversations that do not need them, if that is ever wanted.

## Also found

- `compression.threshold_tokens: 12000` was left in config.yaml from an earlier test. With the fixed part at
  about 16,000, every request was over it, so Hermes tried to compress nearly every turn (one chat went from 14
  to 12 messages mid-reply). Kayla removed it on 2026-10-11; Hermes now compresses at 0.5 of the window.
- Prosoche 0.1.0 compared the floor with an estimate that leaves tool definitions out, so it read over 100%.
  Fixed in 0.1.1.

## A long conversation, 2026-10-11 (chat `...b3e080`)

Kayla's long philosophical conversation reached 11 turns before this was read: 9,619 tokens of earlier turns,
24,299 prompt tokens by the provider on the last request. The window had not stepped (it starts at 16,000).

- **Recall is more than half of the conversation part.** 5,472 of the 9,619 tokens are memory blocks. Each turn
  adds about 650 tokens of recalled memories, which stay attached to their turns (design section 4). The words
  said were about 4,100.
- **The request grows about 1,000 tokens a turn.** At that rate the window would step around turn 17, and
  Hermes' compression (about 32,000 provider tokens) would fire around turn 19: the window gets there first,
  as intended.
- A question for later phases: since recall blocks are the larger share, whether a turn that leaves the window
  should leave with its memory block (as now planned) is worth confirming with the measurements of a step.

### The first step, turn 18 (11:51 UTC)

- Earlier turns reached 16,011 tokens: the window would have stepped. **10 turns would have left (8,374
  tokens) and 8 stayed (7,637).** The request would have been about 16,600 estimated tokens instead of 25,000,
  so roughly 24,000 by the provider's count instead of 32,446.
- 32,446 is right at Hermes' own compression point (half of 65,536). The window steps one turn before Hermes
  would summarise, as intended.
- The oldest block would have been marked for offering at turn 16 (past `offer_at`, 14,000), two turns
  before the step. In one continuous conversation she gets no idle moment in those two turns, so in phase 2
  this is exactly the case design section 7 step 4 covers (the block stays until she has been offered it,
  up to `window_hard`).
- **Hermes' background review also went through prosoche.** After turn 17 Hermes forked the conversation to
  review it for memories and skills (`agent/background_review.py`); the fork's compressor is unbound
  (session id ""), so it shows as a conversation with no id. It made four requests, its current turn growing
  to 9,670 tokens with tool calls, the largest 38,575 provider tokens. Phase 1 must decide what the fork sees:
  it has its own engine instance, so it would make its own window decision on the same history.

## Still open for phase 0

- How long the window stays put after a step (the first step was seen at turn 18; keep the conversation going
  to see the next).
- Whether the window sizes (8,000 / 16,000 / 28,000) suit a 64k window with a 15,000-token floor.

## Correction, 2026-10-12

Hermes' compression point on Kayla's machine is **55,705 tokens (0.85 of 65,536)**, as logged by prosoche 0.2.0
(`threshold` in measure.jsonl), not half the window. The "0.5" and "about 32,000" above were inferred, not
read. So the window's own marks decide when it steps (16,000 tokens of earlier turns, about 32,000 by the
provider's count), well before Hermes would compress.
