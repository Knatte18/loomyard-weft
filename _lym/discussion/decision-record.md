## Goal

```markdown
Imported from [issue #213](https://github.com/Knatte18/loomyard/issues/213).

Import note: the issue names files under `.lym/`, which is deleted with its worktree, so only what the issue quotes of them reached the board: `.lym/parent-review/asks.json`.

## Seen

Under autonomy, PR-Review deadlocked twice on 2026-11-23, in `hydraulics-gaps` and `steady-wob-nan-at-zero-rotation`.
708cdadc5 fixes the first link: the PR-Review agent's own `parent-review ask` is now delivered, and a refused post ends `undelivered` rather than `no-parent`.
The links below remain.

In `steady-wob-nan-at-zero-rotation`'s `.lym/parent-review/asks.json`:

- Ask 2 is the row's own ask at head 038cf197. The orch answered it `--satisfied`, and it is `done`.
- Asks 3 and 4 were sent from the agent's pane at the same head and did not reach the orch.
- `lym loom pr-continue` then refused with "no satisfied answer from your parent stands at 038cf197; ask your parent again with lym loom parent-review ask".

## Still open

1. `pr-continue` judges by the latest ask at the head.
   A later ask that did not reach the parent hides a satisfied answer the parent already gave at the same head.
   Any failed later ask now does this, including `undelivered` and `timeout`.
2. The parent has no verb to settle an ask it never received.
   `parent-review answer` refuses a `no-parent` ask, and presumably an `undelivered` one too, and `parent-review ask` refuses any caller but the PR-Review session.
   Once the agent's asks keep failing, only a hand edit of `"autonomy": false` in the weft's `_lym/shed/<slug>/pull-request.json` lets `pr-continue` pass.
3. 708cdadc5 notes that no test runs the ask from a real creel pane, so the strand claim creel accepts from a live PR-Review pane is unproven.

## Idea

- Let `pr-continue` pass when a satisfied parent answer stands at the current head, whatever the state of later asks at that head.
- Let the parent record a satisfied answer at the current head without a delivered ask, for example `lym loom parent-review answer <slug> --head <sha> --satisfied`.
- Add a test that runs the ask verb from a live non-orch pane.

Related: #200, #206, #211.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): Item 3 leaves this run

Decision: Drop item 3, the tmux-tier test of parent-review ask from a live non-orch pane, from this run; it is filed as its own board note. Items 1 and 2 stand as committed and delivered; no --head flag.

Rationale: A tmux-tier test this run cannot run locally would be written blind and iterated through the verify gate.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
