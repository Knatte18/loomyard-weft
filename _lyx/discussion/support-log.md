# Support log — shuttle-open-permission

## Interview

### Turn 1 — exploration (autonomous)

Read the board entry, the board list, `lyx shuttle open`'s CLI and engine code, `claudeengine`'s permission-mode validation and open launch, and the shuttle package docs.
Findings: `OpenSpec` has no permission field; `buildOpenCmd` passes no permission flag; `validatePermissionMode` already resolves an interactive mode with `bypass` and `prompt`; `Spec.PermissionMode` is documented as caller-owned engine vocabulary.
`creel-stage1-fixes`, which the board entry waits on, has landed (`27887027b`).
Board entries touching shuttle (`codex-engine`, `creel-agents`, `rename-binary-lyd`) do not touch the open permission mode; the engine-vocabulary split (D2) leaves a future engine its own mapping.

### Turn 2 — self-decided batch (autonomous, no operator, no advisors)

- Q1 flag vs config default → flag only (D1).
- Q2 where the mode is validated → in the engine, via the existing rule (D2, D3).
- Q3 resume line → carries the flag on both branches (D4).
- Q4 empty default → stays prompt (D1).

### Turn 3 — lens review by forks

All six lenses returned a report; none found a BLOCKING or MEDIUM issue.

- premises: strand name `ly:<role>` holds only in a `ly` prime → fixed, Goal names `<shortname>[:<slug>]:<role>`.
- premises + scenarios: pause, engine and opener refusals precede the mode refusal → fixed, stated in D3.
- premises + scenarios: D5's deny rules and hooks in bypass rest on Claude Code behaviour → fixed, marked as an external premise in D5.
- premises: `OrElse` single-pipeline premise holds → no change needed.
- goal-scope: R1's `ly:cli` premise unverified → fixed; `ly:cli` is `lyx help index`, which lists commands not flags, and R1 now says so.
- goal-scope + leanness: AC6 (strand name) restates unchanged behaviour → fixed, cut.
- goal-scope: D1 rationale rests on an unsourced caller claim → fixed, now rests on the unchanged default and YAGNI.
- board-fit: `codex-engine` would need to honour the open mode → fixed, D2 says a `SessionOpener` honours or refuses it.
- board-fit: `rename-binary-lyd`, `creel-agents`, `orch-split` ordering notes → rejected, no conflict and nothing for this record to do.
- pattern-fit: help text must name mode words, never Claude's flag → fixed, D2.
- pattern-fit: log the mode on the opened-session line → fixed, Scope and D5 (`PATTERN-spawn-observability`).
- pattern-fit: `PATTERN-no-denied-recovery` unnamed → rejected, the refusal names flag values, not a denied command.
- scenarios: resume by another reed process after a crash → rejected, `ResumeCmd` is persisted and AC2 covers it.
- scenarios: Claude Code's first-use bypass dialog → rejected, informational; the operator answers it in the watched window.
- scenarios: no AC for bypass under a covering pause → rejected, the existing pause row in `open_test.go` covers it regardless of mode.
- scenarios: no new concurrency → no change needed.
- leanness: A1 and A3 repeat D1 → fixed, cut; the remaining assumption is renumbered A1.
- leanness: generic `PATTERN-comment-line-breaks` and `PATTERN-test-economy` constraints → fixed, cut.
- leanness: docs constraint repeats Scope and AC → fixed, reduced to the bare rule citation.
- leanness: Scope Out's board-entry bullet is process, not scope → fixed, cut.
- leanness: test-budget note repeats CLAUDE.md → fixed, cut.
- leanness: D3's refusal sentence repeats AC4 → fixed, replaced by the refusal-ordering sentence.
- leanness: R1 hid a requirement for the help example → fixed, moved into AC6.

## Rejected alternatives

- A `shuttle.yaml` key for the open permission mode: provider vocabulary in the engine-neutral config, and a resolution layer for a per-call choice (D1).
- A `claude.yaml` key `open_permission_mode`: engine-scoped and so seam-clean, but still a default nobody asked to persist; the flag covers the operator's case.
- Making bypass the default for `open`: changes every existing caller's behaviour and removes the dialog without an explicit request.
- Validating the mode in `shuttlecli`: duplicates the engine's rule and puts provider vocabulary outside the engine package.
- Passing shuttle's `--settings` guards to an opened session in bypass: changes what an opened session is (it takes the worktree's own settings by design) and is not asked for.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): flag or config default? → flag only; the caller states the mode per session, and the entry offers either.
- Q2 (resolved, auto-pick): who validates the mode? → the engine, through `validatePermissionMode(mode, true)`, matching `Spec.PermissionMode`.
- Q3 (resolved, auto-pick): does the resume line carry the mode? → yes, both the resume branch and the fresh fallback, since reed replays `ResumeCmd`.
- Q4 (resolved, auto-pick): default when the flag is absent? → prompt, today's behaviour.
- Q5 (resolved, auto-pick): flag values? → `bypass` and `prompt`, `Spec.PermissionMode`'s vocabulary.
