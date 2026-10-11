## Interview

Autonomous run: no operator answered, so each turn below is the writer's own exploration and pick.

1. Read the board entry `twb-tw-om-2`, its linked issues (#505, #509, #517, #522, #511, #500, #507) and the touching entries `fork-wait-noise`, `proc-child-lifetime`, `weaver`, `convergence-plan` and `audit-reader-globs`; the base is `800a0e892`.
2. Gate shells: at base `background_shell_wait_min` already expires nothing; the remaining hole is `expiredTurnEnd` finishing the first fresh gated arrival at once when only unawaited shells are outstanding. Master awaits only `lyx webster recover-batch`. Picked: await `lyx gate ` too, matched at command-segment starts (D1).
3. Master self-fix: shuttle's generic re-prompt reads "read it, fix every finding, and end your turn", which contradicts Master's stencil at the moment it reads the failure. Picked: a per-entry re-prompt text for webster's verify gate (D2).
4. Recovery attribution: `loadRunWrites` audits Master sessions and their forks only; a recovery strand's session is never audited. Picked: record each recovery session on the batch record and audit it (D3).
5. `recover-batch` exit: a terminal non-done digest returns a normal envelope with exit 0. Picked: non-zero for every non-done terminal status, envelope unchanged (D4).
6. Recovery gates: the shared implementer body bans mutating `lyx fabric` verbs and the recovery prefix says nothing about gates. Picked: the recovery prefix names the card and batch gates as allowed and leaves `tmux`-tagged steps to verify, as the board asks (D5).
7. Create check: probed `lyx quarry resolve` (quarry v0.2.1, same as base `go.mod`) on an unexported function, a test-file function and the two #511 targets as now written; all answer `found`, and `internal/proc#StartTime` answers `ambiguous`. So exports and test files are not the limit; `tmuxAllowed` was a build-constrained duplicate, and `rusageStats`'s cause stays unidentified (D6, Open risks).
8. Build tags: DoneChecks, the Plan-time status policy and handle binding read `ambiguous` as unresolved. Picked: one disjoint-constraint rule applied to every positive reading, with still-exists readings unchanged (D7).
9. Lens review (six forks: premises, goal-scope, board-fit, pattern-fit, scenarios, leanness; all returned). Duplicates merged; each finding with its lens and disposition:
   - premises, board-fit: D1's bound cited `lyx gate test`'s CLI wait, which bounds only the slot wait. Fixed: bound is the run deadline, liveness and `go test`'s own timeout.
   - premises: the dirty-tree findings text tells Master to commit or remove paths. Fixed: D2 readdresses it to the fixer.
   - premises, pattern-fit, scenarios: D4's envelope `ok` and refusal status unstated. Fixed: stays `ok: true`, not a refusal, exit after the commit.
   - premises: D4's `failed` digest cannot occur; it is a `batch_failed` refusal. Fixed: dropped.
   - premises, goal-scope: D6 asserted #511's cause for `rusageStats` without evidence. Fixed: moved to Open risks.
   - premises: D7's status policy also covers Delete targets. Fixed.
   - goal-scope: D7 overclaimed Plan-time flagging for Create targets. Fixed: stated as existing targets only.
   - goal-scope: D3 widened reset's own-path set unrequested. Fixed: reset unchanged.
   - goal-scope: whether Master should run gates at all. Fixed: D1 says the hold applies whatever started the shell.
   - goal-scope, pattern-fit, scenarios: D5's compile-check named no allowed command. Fixed: dropped; `llm` added beside `tmux`.
   - goal-scope: multi-file Edit of a per-GOOS member at the scope guard unpinned. Fixed: acceptance criterion.
   - board-fit: `pane-inject-safety` touches D2's send path and shuttleengine. Fixed: named in Constraints.
   - board-fit: `fork-wait-noise` ownership of how gates run. Fixed: Out bullet states the split.
   - board-fit: `weaver` inherits D5's skip. Fixed: named in Open risks.
   - pattern-fit: D1 added a second quote-unaware splitter beside `refscanner.go`'s. Fixed: splitter moves to `internal/shell`, seam entry widened.
   - pattern-fit: D2's text location unstated. Fixed: Go string beside the entry, with the reason.
   - pattern-fit: D7's directory test via `glyph.UnitPath`. Fixed.
   - pattern-fit: fixed GOOS list is perishable. Rejected: no stdlib export exists, a `go tool dist list` test would spawn a process, and the list fails closed.
   - scenarios: a recorded recovery session with no transcript fails `AuditForks`. Fixed: no writes, Warn, criterion.
   - scenarios: Master's handling of a failing own gate-shell result. Fixed: template rule in D1.
   - scenarios: D7 on an unparsable or `+build`-only constraint. Fixed: `+build` read, unparsable is unconstrained.
   - scenarios: handle binding of a disjoint-ambiguous answer undefined. Fixed: binds as multipart.
   - scenarios: test files excluded without reason. Fixed: all-test candidates may be disjoint; only a mix stays ambiguous.
   - scenarios: newline, `&`, `(` and `$(` separators. Fixed via the shared splitter.
   - scenarios: attach path for D2 and D1, restart reclaim for D3. Fixed: criteria added.
   - leanness: Auto-mode assumptions restated decisions and hid the GOOS list. Fixed: list moved into D7, section cut to one line.
   - leanness: rejected-alternative and probe narrative in D3 and D6. Fixed: removed.
   - leanness: non-applicable Constraints and CLAUDE.md restatements. Fixed: cut.
   - leanness: bench base in Goal and misplaced corpus pointer. Fixed: moved to Notes.

## Rejected alternatives

- D1, stop the shell and record its result as discarded: lyx has no pid for a provider shell (shuttle doc.go), so a stop would be strand removal again; holding gets the result instead of a note about losing it.
- D1, hold every outstanding shell in a gated run: holds Master on unrelated long shells too; gate shells are the ones whose result decides the outcome.
- D1, exact prefix match only: misses `cd <pair> && lyx gate test …`, the shape agents write.
- D1, a second splitter inside shuttle: two parsers of agent shell text drift; `refscanner.go` already has a quote-aware one.
- D3, also counting recovery writes in reset's own-path set: not asked for, and it changes what a reset treats as its own.
- D5, compile-checking tagged steps: every compile form of a tagged `go test` is denied, and the board asks only that they be left to verify.
- D2, a `lyx webster` verb that spawns the fixer: a fork is spawned in-session through the Agent tool, which no CLI verb can do.
- D2, restating the rule only in the template's rules list: it is already there and was not followed; the message Master reads is the place.
- D3, start the second recovery from a clean tree with the first attempt's diff as reference: cleaning the tree discards any foreign uncommitted file too, and the diff hand-off adds a new artifact for the same outcome.
- D3, attribution by shuttle run directory: a respawn replaces the record's run directory, so the earlier attempt's would be lost.
- D4, non-zero only for `stuck` and `dead` without retry: the exit code then answers "is the run over", which nobody asked; "is the batch done" is the reading the issue expected.
- D6, extending quarry to resolve unexported or test-file names: the probe shows it already does.
- D7, make quarry honour build constraints: quarry is a separate module outside this repo; the fix in planglyph covers the plans that hit it.
- D7, let a `Create` handle reuse a name its own card deletes (#507's first option): with D7 a plan Edits a per-GOOS member, so the delete-and-recreate shape is not needed.
- D7, accept every `ambiguous` answer for Create: passes genuine duplicate declarations.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Hold gate shells or stop them? Auto-pick: hold (D1). Resolved.
- Q2 Which shells hold? Auto-pick: commands with a `lyx gate ` segment, plus the existing recovery prefix. Resolved.
- Q3 Where does the self-fix rule reach Master? Auto-pick: the gate re-prompt line itself (D2). Resolved.
- Q4 Attribute recovery writes by batch or by session? Auto-pick: recorded sessions on the batch record, carried forward (D3). Resolved.
- Q5 Which `recover-batch` outcomes exit non-zero? Auto-pick: every terminal status but `done` (D4). Resolved.
- Q6 Does a recovery run `tmux`-tagged steps? Auto-pick: no, it compile-checks them and leaves them to verify, per the board (D5). Resolved; residual risk in the record's Open risks.
- Q7 Does quarry resolve unexported and test-file names? Probed: yes (D6). Resolved; `rusageStats`'s #511 cause stays open (record's Open risks).
- Q8 Fix ambiguity in quarry or planglyph? Auto-pick: planglyph, one shared rule (D7). Resolved.
- Q9 Revive a dead batch through `goto` (#511)? Auto-pick: out of scope, not in the board entry. Resolved.
