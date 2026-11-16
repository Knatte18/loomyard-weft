# P1 A Codex session gets exactly what a Claude session in the same role gets

Rendered 2026-11-16 14:05 UTC; every age below is as of then.
Under: [P2 Codexengine: every task, trial and note about the Codex engine](project-codex-engine-trial.md).

[`codex-parity`](design-codex-parity.md) · Age 9m · Owner `neve`

Groups the entries filed from the Sketch `codex-same-as-claude`: prompt-engine-per-launch, codex-standing-instructions, codex-wording-engine-blocks, codex-question-deny-nudge, launch-manifest-parity.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (5)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (5)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | `draft` **Every producer renders its prompt and edit directive for the engine its launch resolves to** | [`prompt-engine-per-launch`](design-prompt-engine-per-launch.md)<br>[Sketch](decision-prompt-engine-per-launch.md) | darn · solid | T5 | 9m<br>9m | codex, shuttle, stencil, heddle |
| T13 | `draft` **A Codex session gets Claude's standing items, user setup, instruction files and subagent terms in its developer instructions** | [`codex-standing-instructions`](design-codex-standing-instructions.md)<br>[Sketch](decision-codex-standing-instructions.md) | tapestry · solid | T6 | 9m<br>9m | codex, shuttle, stencil, heddle |
| T17 | `draft` **Claude-only wording moves into Claude engine blocks, each with a Codex copy, held by a term scan** | [`codex-wording-engine-blocks`](design-codex-wording-engine-blocks.md)<br>[Sketch](decision-codex-wording-engine-blocks.md) | darn · solid | T13 | 9m<br>9m | codex, shuttle, stencil, heddle |
| T18 | `draft` **A Codex session gets the question-tool deny and the batching nudge Claude runs** | [`codex-question-deny-nudge`](design-codex-question-deny-nudge.md)<br>[Sketch](decision-codex-question-deny-nudge.md) | darn · solid | T13 | 9m<br>9m | codex, shuttle, stencil, heddle |
| T19 | `draft` **Each launch writes its manifest, a parity test compares the engines per seat, and lym shuttle manifest prints both** | [`launch-manifest-parity`](design-launch-manifest-parity.md)<br>[Sketch](decision-launch-manifest-parity.md) | tapestry · solid | T17<br>T18 | 9m<br>9m | codex, shuttle, stencil, heddle |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
