## Interview

Autonomous mode: no operator answers; questions went to one advisor, `ly:twb-combo-2:discussion-advisor-1`, in two rounds, and each pick was settled after weighing its answer.
All code was read at bench base `800a0e892`.
The advisor disclosed that it ran `git show 1e79825ff --stat` (message and file list only, no diff) after reading the bench paragraph; its answers were weighed with that in mind, and every claim it made was cited at base and checked against what the chair read.

### Turn 1 — exploration

The chair read the board entry, the related entries (`fork-wait-noise`, `proc-child-lifetime`, `weaver`, `glyph-practice-audit`, `verify-watchdog`, `audit-reader-globs`), issues #500, #505, #507, #509, #511, #517 and #522, and the base code each part touches.
Findings that shaped the questions:

- `background_shell_wait_min` no longer expires anything at base; the remaining path is `countsAtOnce`, which finishes a gated run `done` at once over an unawaited shell when the output files exist.
- `gateRepromptText` tells every gated writer to "fix every finding", Master included.
- `loadRunWrites` audits Master sessions and their forks only; a recovery strand's session is never recorded.
- A terminal `stuck` or `dead` recovery digest returns `output.Ok`.
- Quarry v0.2.1 resolves unexported and same-package test-file members; `internal/proc#StartTime` answers `ambiguous` across per-GOOS files.

### Turn 2 — round 1 (parts 1–3)

- Q1, gate shell at a turn end. Advisor (`ly:twb-combo-2:discussion-advisor-1`): hold only gate-shaped shells, matched by command shape as caller data in shuttle, since hold-all would pin Master on a stray `tail -f` until `master_timeout_min` (480); after the wait, webster demotes a `done` whose `EndedShells` holds a gate shell to `stuck`; a stencil line about Master's gates. Settled as D1 and D2 (record numbering after the lens review); the demotion was adopted, then dropped in the lens review.
- Q2, Master self-fix. Advisor: per-entry re-prompt text on `GateEntry` plus the same instruction at the top of the findings file; a spawning verb would be a cold strand. Settled as D3.
- Q3, recovery attribution. Advisor: record each recovery session id on the batch, audit them in `loadRunWrites` as a separate slice; `reset.go` and `contractevidence.go` read the same evidence; skip a transcript-less session. Settled as D4, keeping `contractevidence.go` unchanged.
- Q4, exit code. Advisor: exit 1 with `ok: false` and the digest fields for terminal `stuck` and `dead`; refusal-spec rows, Master ladder wording, the `Long` help cap and the refusal-note writer follow. Settled as D5.

### Turn 3 — round 2 (parts 4–6)

- Q5, recovery gates. Advisor: the text goes in the shared implementer body; nothing denies `lyx gate test --tags tmux`, so the doubt was textual; compile a tagged step through `lyx gate test … -- -run '^$'` to stay slotted, and say "left to Publish's `publish_verify`", since the plan verify only compiles the tmux tier. Settled as D6, with the tagged-test rule moved to the recovery prefix in the lens review.
- Q6, #511. Advisor: no resolver change; #511's `tmuxAllowed` is #500's member, `rusageStats` likely per-GOOS but unconfirmable at base; the refusal names key, status, unit and candidates with a way forward per case, and the same shape for delete and rename. Settled as D7.
- Q7, partitioned members. Advisor: rewrite a partitioned `ambiguous` to a multipart-shaped answer in the plan-side callers only, decided by `go/build.Context.MatchFile` over enumerated configurations, failing closed past four free tags; #507's delete-and-recreate becomes moot and is unsatisfiable anyway; optionally a `planparser` check for Delete and Create of one key. Settled as D8 without the extra check.

### Turn 4 — lens review

Six forks, all returned. Decision numbers below are the first draft's (D1–D9); the revised record renumbers after dropping draft D2.

- pattern-fit, scenarios, board-fit, premises — draft D2's `done`→`stuck` demotion halts a verified run, can fire on a stale `EndedShells`, and duplicates the verify gate (a deadline end still runs `evaluateGate(final)`; checked at base): fixed, demotion dropped; D1 now states the gate covers a deadline end, and the shell wording says "result never read".
- scenarios — no way forward for draft D2's `stuck`: fixed by dropping D2.
- scenarios — a held shell whose completion never wakes Master holds to the deadline: fixed, added to Open risks and an acceptance criterion for a completing held shell.
- premises, scenarios — handle binding is delta-based, not a resolve site: fixed, D8 states `BindHandles` binds on any created entry with the ID, Open risks names the unverified delta shape.
- premises, pattern-fit, board-fit — go/build exports no GOOS/GOARCH list: fixed, D8 derives configurations from the candidates plus an "other" value, with a no-spawn constraint.
- scenarios — free tags must exclude configuration-fixed tags or `_linux.go` and `!linux && !windows` overlap falsely: fixed in D8.
- premises — a Rename whose old side is partitioned still fails: fixed, named in Scope Out.
- goal-scope, scenarios — draft D7 changed fork behaviour on tagged tests and its "tagged card step" premise does not exist (card gates carry no tags, card `Verify:` refuses tags): fixed, D6 moves the tagged-test rule to the recovery prefix and speaks of tests in tagged files.
- pattern-fit — draft D7 restated the plan template's tier contract in three places: fixed, the recovery prefix points at the plan template; plan template and spec edits dropped.
- board-fit, goal-scope — D6 and D2 text overlaps `fork-wait-noise`: fixed, D6 says what may run and never how a call is launched.
- goal-scope — the incident's exit-0 path is not identified: fixed, D5 now states the rule for every call answering a terminal non-`done` batch and names the base paths it changes.
- pattern-fit — which friction note covers D5's outcome: fixed, D5 names `noteRefusals`'s `webster-refusal-recover-batch` note and no second note.
- goal-scope — "flags at Plan time" reinterpretation not marked: fixed, listed under Auto-mode assumptions.
- goal-scope — D4 attribution is run-wide, not by batch: fixed, listed under Auto-mode assumptions with its bound.
- goal-scope, scenarios — D7 misses the external `_test` package unit and path-shaped targets: fixed in D7 and an acceptance criterion.
- scenarios — an attaching call must not record a session, and the session id must be saved in the post-spawn commit: fixed in D4 and an acceptance criterion.
- scenarios — no other Go caller reads `recover-batch`'s exit code: no change needed.
- pattern-fit — reusing webster's splitter from shuttle is a reverse import: fixed, D1 and the notes say the splitter lives in `shuttleengine` or below.
- board-fit — D8's tests should pin quarry v0.2.1's candidate shape against `enclose-kickstart`'s bump: fixed, added to Constraints.
- board-fit — `discussion-premise-cites` and `kick-start-pack` are future resolve consumers: fixed, one line in the notes.
- board-fit — Weaver inherits D4 and D5: rejected, Weaver is an unbuilt idea and inherits whatever lands.
- leanness — rejected alternative in draft D4's rationale: fixed, cut.
- leanness — false premise in the D3 auto-mode line: fixed, restated as the chair's own pick.
- leanness — Goal repeats Scope: fixed, Goal shrunk and issue numbers kept in Scope.
- leanness — D6 rationale and D1 rationale repeat Constraints lines: fixed, cut from the decisions.
- leanness — the test-economy constraint is generic: fixed, cut.
- leanness — the `Long` cap is a note, not a constraint: fixed, moved to the notes.
- leanness — an acceptance criterion named a test: fixed, reworded to the behaviour.
- leanness — Open risk repeated draft D2's rationale: fixed, trimmed.
- leanness — `rusageStats` repeated in three places: fixed, kept in Open risks only.

## Rejected alternatives

- Part 1, lyx stops the shell explicitly: the provider reports no pid, lyx kills no shell today, and a kill would need a new provider seam.
- Part 1, every outstanding shell holds a gated Master turn end: a stray shell a fork left would pin Master until the run deadline.
- Part 1, adding `lyx gate` to the whole-label prefix list only: `cd X && lyx gate test …` and `… | tail` labels miss it.
- Part 1, teaching Master to run gates bare (no `cd`): Master's role runs no gate; D1 covers a lapse.
- Part 1, webster demoting a `done` with an unfinished gate shell to `stuck`: a `done` has passed the verify gate, a deadline end still runs it, and the demotion could halt a verified run on a stale shell list.
- Part 1, reusing webster's audit command splitter in shuttle: `websterengine` imports `shuttleengine`, so it would be a cycle.
- Part 4, the tagged-test rule in the shared implementer body: it would change fork behaviour the board did not ask to change.
- Part 4, restating the tier contract in the plan template and spec: the template already owns it; the recovery prefix points at it.
- Part 2, a webster verb that spawns the verify fixer: Go cannot make an Agent-tool fork, so it would be a cold strand with a different cost and design.
- Part 2, stencil-only rewording: the contradicting generic re-prompt would still be what Master reads first.
- Part 3, the second recovery starts from a clean tree with the first attempt's diff as reference: it discards work, needs a new lyx-performed step, and doubles the prompt.
- Part 3, non-zero exit only when no recovery is left: a dead recovery with a retry left still failed.
- Part 3, extending the contract-file check to recovery writes: no run has shown that case, and its message speaks of forks.
- Part 5, a quarry change for unexported or test-file members: quarry already resolves them.
- Part 6, quarry honouring build constraints: an external module and a version bump; lyx classifies from the candidates quarry already returns.
- Part 6, accepting every ambiguous Create: a genuinely duplicated declaration would pass.
- Part 6, normalizing inside `resolveTargets`: it also backs `lyx quarry resolve`, which should stay quarry's own answer.
- Part 6, a `planparser` check flagging a Delete and a Create of one key: not asked for, and `createFindings` already reports `create-already-exists` for that plan at Plan time.
- Part 6, #507's Create-reuses-a-deleted-name exemption: moot once a partitioned member is edited by glyph.

## Review rounds

_No rounds yet._

## Question ledger

### Round 1 — private picks, before advisors

- Q1 (part 1, gate shell at a turn end) — private pick, before advisors: Master's spawn declares `lyx gate test` as an awaited shell, matched per shell segment rather than as a whole-label prefix, so a running gate shell holds Master's turn end, bounded by the run's own timeout and the liveness check; every other shell keeps today's record (warning, friction note, summary section), reworded to say its result was discarded. Alternatives seen: (b) lyx stops the shell explicitly (no pid is reported, so lyx cannot); (c) every outstanding shell holds a gated Master turn end; (d) stencil-only: Master never runs a gate.
- Q2 (part 2, Master fixes a verify failure itself) — private pick, before advisors: the gate entry carries its own one-line re-prompt text as caller data, shuttle's generic text stays the default, and webster's verify entry names the fixer prompt file and says to spawn one fixer fork and edit nothing. Alternatives seen: (b) a webster verb that spawns the verify fixer (a cold strand, a different design); (c) stencil-only rewording.
- Q3 (part 3, second recovery treats the first's files as foreign) — private pick, before advisors: the batch record keeps every earlier recovery strand's session id, carried across spawns like `Recoveries`, and the run's write evidence audits them, so their writes read "Written by this run". Alternatives seen: (b) the second recovery starts from a clean tree with the first attempt's diff offered as reference.
- Q4 (part 3, `recover-batch` exit code) — private pick, before advisors: a terminal `stuck` or `dead` digest exits non-zero with the same digest fields on the failure envelope; `done` and the `running` snapshot stay exit 0; Master's ladder rungs are reworded to match. Alternatives seen: (b) non-zero only when no recovery is left (`recovery_retry: false`); (c) keep exit 0 and add a field.

### Round 1 — settled

- Q1: advisor agreed on matching gate shells by command shape and added two options: a webster demotion of `done` to `stuck` when a gate shell ended unfinished, and a stencil line on Master's gates. Settled (final record numbering): D1 (agreed); the demotion (added option) adopted, then dropped in the lens review; D2 (added option, adopted in the stricter form "Master runs no gate", an auto-pick). Advisor-informed, `ly:twb-combo-2:discussion-advisor-1`.
- Q2: advisor agreed and added the findings-file first line. Settled: D3 with both. Advisor-informed, `ly:twb-combo-2:discussion-advisor-1`.
- Q3: advisor agreed, added the transcript-less skip and named the other evidence readers. Settled: D4; `contractevidence.go` left unchanged and run-wide attribution are auto-picks. Advisor-informed, `ly:twb-combo-2:discussion-advisor-1`.
- Q4: advisor agreed and named the refusal-note, refusal-spec and help-cap consequences. Settled: D5; `dead` with a retry left exiting non-zero is an auto-pick. Advisor-informed, `ly:twb-combo-2:discussion-advisor-1`.

### Round 2 — private picks, before advisors

- Q5 (part 4, recovery prompt gates) — private pick, before advisors: the shared implementer body (which the recovery prompt composes) names the card gate, the batch gate, `lyx gate test` and `lyx loom lint-comments` as always allowed and says the fabric ban covers mutating `lyx fabric` verbs only; a `tmux`-tagged step is compiled, not run, and left to the plan's `## verify:` and Publish's `publish_verify`. Alternatives seen: (b) the text in the recovery prefix only; (c) let the strand run tmux-tagged steps through `lyx gate test --tags tmux`.
- Q6 (part 5, Create check on unexported targets) — private pick, before advisors: no resolver change, since quarry resolves unexported and same-package test-file members; the `create-not-done` detail names the key looked up, quarry's status and, for `ambiguous`, each candidate's file. Alternatives seen: (b) a quarry change.
- Q7 (part 6, partitioned members) — private pick, before advisors: a `planglyph` classifier treats an `ambiguous` answer whose candidates sit in files no single build configuration selects together as one partitioned member, accepted wherever found or multipart is accepted (Plan-time status policy, Create and Rename done-checks, handle binding); `delete-not-done` keeps blocking while any candidate remains; a non-partitioned ambiguity stays `glyph-ambiguous` at Plan time. #507's delete-then-recreate of the same name is out of scope once glyph Edit works. Alternatives seen: (b) change quarry to honour build constraints and bump it; (c) accept every ambiguous Create.

### Round 2 — settled

- Q5: advisor agreed, differed on wording ("left to Publish's `publish_verify`", not "to verify") and added the slotted compile form and `lyx quarry resolve`. Settled: D6 with the advisor's refinements, the tagged-test rule later moved to the recovery prefix. Advisor-informed, `ly:twb-combo-2:discussion-advisor-1`.
- Q6: advisor agreed and added unit status, build constraints and per-case ways forward, extended to delete and rename. Settled: D7. Advisor-informed, `ly:twb-combo-2:discussion-advisor-1`.
- Q7: advisor agreed and added the mechanism (multipart-shaped rewrite in plan-side callers only, `MatchFile` enumeration, four-tag fail-closed cap) and an optional `planparser` Delete-and-Create check. Settled: D8 without the check (rejected, see Rejected alternatives); the cap of four and the "flag at Plan time" reading are auto-picks. Advisor-informed, `ly:twb-combo-2:discussion-advisor-1`.

### Environment

- No tool failed during the interview.
