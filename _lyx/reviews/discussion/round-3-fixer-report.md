# Fixer report — discussion, round 3

The record on disk does not hold the text the review quotes.
D16 has no `origin/prod` sentence, D17 has no fallback check, and D11's first sentence reads "Every production `logger.Warn` call is checked ..., and so is every log line at Info or above inside a loop or a polled path", with no exemption for a Warn site outside a loop.
Round 2's fixer-report describes changes that are absent from the file.
Each finding was weighed against the file as it stands.

## Changes

- **F1 (fixed differently)**: the core premise holds, more strongly than stated: D17 had no fallback at all, so a dropped notice left the orch no durable signal on any hub.
  D16 now says the board entry turning `done` at teardown is what D17's wake check reads in the notice's place, and that `origin/prod` is the baseline for the deploy only.
  D17 gains two wake checks that read the board and do not depend on the `deploy` key: a running task whose entry is `done`, or pruned off the board by a later landing, is no longer running; and every ready task not yet started is started.
  D17 also gains the deploy-recovery check, scoped to an empty `deploy` key and to landings into `main`. It compares `origin/prod` against `origin/main`.
  The old "next wake after any landing" line gives way to it.
  Two acceptance rows pin the board checks and the deploy-recovery check.
  Not taken from the `fix` key: "or whose pair is gone" as a landed signal, because `internal/pairteardown/claim.go:31-33` clears an abandoned run's claim on teardown, so a gone pair does not mean the task landed.
  Also not taken: the `require_pr_to_base` scope, replaced by `main`, because `origin/prod` against `origin/main` sees only landings into `main` (`tools/deploy/main.go:154-163` requires HEAD on `origin/main`).
  D16 had no `origin/prod` sentence to replace, so the pointer was added, not swapped in.
- **F2 (fixed differently)**: the premise as written is false: no sentence exempts a Warn site outside a loop, line 134 gives such a site the keep verdict, and the Scope bullet, the first sentence, the rationale and Q7 all state the broad reach.
  What does hold is the focus question's mismatch: the first sentence checks every in-loop Info-or-above line, but the finish line and the acceptance row asked for verdicts on Warn sites only.
  Narrowing the reach, as the `fix` key proposes, was not done. It would drop the non-loop Warn sites that the board's ask for the other modules' Warn calls covers (D11 rationale, Q7).
  Instead, the finish line and the acceptance row now also give every Info-or-above line inside a loop or polled path a verdict.
- Support log: a Round 3 entry replaces "_No rounds yet._" under Review rounds.

## Self-check

- consistency: no findings.
- way-forward, MEDIUM, the deploy-recovery check could fire at every wake without progress (a deploy ships the prime's HEAD and refuses a dirty tree, `tools/deploy/main.go:50-57,147-163`): fixed as proposed. The check now pulls the prime before the deploy and reports a refused pull or deploy, or a `prod` still behind `main`, once, with no retry until `origin/main` moves. The acceptance row pins that report.
- way-forward, LOW, the comparison could read stale refs: fixed as proposed. The check now fetches first.
- claims, LOW, `done` or pruned does not always mean landed (`claim.go:45-46` marks a finished, unlanded run done, and an operator can delete an entry): fixed as proposed. The check now says "is no longer running" instead of "treated as landed", and its acceptance row matches.
- claims, LOW, "each reading a durable baseline" overstated the orch's own running set: fixed as proposed. The checks are now described as reading the board, which survives a refresh, compact or handoff.

## Disputed

None.

## Deferred

None.
