# Fixer report — discussion, round 1

The record's section "Added after Discussion (operator, 2026-10-21)" replaces D7's zero-test handling: zero discovered passes, no floor, no skipped outcome.
The review's floor and drop-rule findings read D7 as it stood before that decision, so the fix folds the decision into D7 and removes `min_tests`, `build_tags`, the drop rule and the skipped outcome from the Goal, D1, D3, D7, D11, D12, Auto-mode, Open risks and the acceptance criteria.
The operator's section itself is unchanged.

## Changes

- F1 — fixed as proposed (second option): the card `**Verify:**` runner keeps one gate slot, and its timeout from the slot, for any line that is not one `lyx gate test --wait` call (D8); such lines are non-tier holders in a new D13, and the `PATTERN-gate-slot` and `PATTERN-test-tiers` texts name them; acceptance criterion added.
- F2 — fixed differently: the floor half dissolves under the operator decision, so no floor rule for `none` is written; D3's `none` bullet now makes computed, `--base` and card scopes whole and passes explicit targets verbatim (D2); acceptance criterion for explicit-target and filtered `none` runs added. Not made: "takes no floor" wording, since no floor exists.
- F3 — fixed as proposed: D8's Merge-In/catch-up/Finalize and Publish rows carry darn's tree-scoped path, matching `darnVerifySource` (`internal/loomcli/darnwiring.go:53-71`): `tree_scope_verify` in place of `landing`, and Publish skips `publish`; acceptance criterion added.
- F4 — fixed as proposed (first option): darn runs no command, logs it, and passes the site when the rendered `{targets}` is empty (D9); acceptance criterion added.
- F5 — fixed as proposed: `tree_scope_verify` must be one `lyx gate test --wait` call (D9); `verifytree` drops its 60-minute timeout since every command it runs is a gate call, and the card runner runs a gate-call line outside its 10-minute timeout (D8); the plan gate refuses a `**Verify:**` gate call lacking `--wait` (D10); D2's `--wait` bullet names both; busy-hub acceptance criterion added.
- F6 — fixed as proposed: D1 says a `{targets}`-less command ignores any narrower scope and runs whole, except that an empty computed, `--base` or card scope runs no listed tier's command; D3 and the acceptance criterion match. "Skipped" became "runs no command and passes with zero discovered", since the operator decision leaves no skipped outcome.
- F7 — fixed differently: the drop rule is gone with the operator decision, so `go vet {targets}` vets every impacted package as today; D3's Bound names the one remaining extra narrowing, an empty set skipping `tier1`'s `go build ./...`; acceptance criterion added.
- F8 — fixed differently: D1 names the required tier keys; a tier is written whole through a new flow-mapping `--set` for an open-map entry, since `--set` writes open-map entries only as scalars (`internal/configcli/configcli.go:466-467`) and the self-check found the orch cannot drive the editor the first fix named; D12 states the provisional `timeout_min: 60` (`internal/verifytree/verify.go:28`'s ceiling) and drops `min_tests`; Scope gains the `--set` form; acceptance criterion added. Not made: making missing per-tier keys a use-time refusal, since a whole-tier write makes the key-by-key path unneeded.
- F9 — fixed as proposed: D5 makes the guard step a run of `guard_tier` with its slots, `timeout_min` and `prebuilt_lyx`; D2's timeout bullet and D11's record match; acceptance criterion added.
- F10 — fixed differently: under the operator decision a confirm-the-fix run discovering zero passes; D8 says so and the acceptance criterion names it. Not made: a "skipped" outcome, which the operator decision rules out.
- F11 — fixed differently: the test-holding definition is removed with the drop rule; a Benchmark-only package discovers zero and passes, named in the acceptance criterion. Not made: the narrower `Test`/`Fuzz`/`Example`-with-output definition, since no rule reads it any more.
- F12 — fixed as proposed: every loomyard tier's `env` is `GOFLAGS: -trimpath -p={parallel}` and the command drops its own `-p` (D12); acceptance criterion added.
- F13 — fixed as proposed: new D13 lists the card runner, `internal/planglyph`'s type load and `cmd/testtiming` as non-tier holders setting their own GOFLAGS, with `-p` from `tier0`'s `parallel`; `cmd/testtiming` is named as an exception in both PATTERN texts; Scope and D6 point at D13.
- F14 — fixed as proposed: D8 says Merge-In and Publish name the round's base command on Site.
- F15 — fixed as proposed: `-C` with another scope form is refused before any slot, naming explicit targets (D2).
- F16 — fixed as proposed: D5 checks `guard_tier`'s placeholders only for a guard step that is not skipped.
- F17 — fixed as proposed: D2's Bound reworded.
- F18 — fixed as proposed: D10's Bound defines command position as the shell guard's (`internal/shellguard/rows.go:63-71`, launchers included) and lists what passes: a script, `bash -c`, `go -C dir test`, and an assignment prefix.
- F19 — fixed as proposed: `tree_scope_verify` may hold no shell control operator (D9); acceptance criterion updated.
- F20 — fixed as proposed: Open risks names pre-deploy processes acquiring single slots past the drain.
- F21 — fixed as proposed (first option): a machine-layer `parallel` for an undeclared tier is dropped at load with a warning naming the operator's machine-layer editor session.
- F22 — fixed as proposed: D2 says the watchdog watches Go test binaries only, so a `vstest` tier is bounded by `timeout_min` alone.
- F23 — fixed as proposed: D9 says the Webster fork and recovery prompts stop rendering the plan's verify section.
- The support log records the round, updates the zero-test ledger entry, and adjusts three rejected alternatives the fixes made stale.

## Self-check

- fix-consistency, MEDIUM: D1 required `all_targets` for NORCE tiers that hold none — fixed as proposed: required only for a command holding `{targets}`.
- fix-consistency, MEDIUM: D1's "no tier command at all" contradicted D3's guard step — fixed as proposed: "no listed tier's command".
- fix-consistency, LOW: the Goal ignored D13's exceptions — fixed as proposed.
- fix-consistency, LOW: darn's doc example lacked `--wait` and the template comment was stale — fixed as proposed (D9).
- fix-consistency, LOW: Auto-mode said no operator answered — fixed as proposed.
- fix-consistency, LOW: the trimpath constraint covered tier env only — fixed as proposed.
- fix-claims, LOW: D13's "keeps today's flags" overstated — fixed as proposed: names the `-p` source change and the no-`tier0` case.
- fix-way-forward, MEDIUM: the editor-save route cannot be driven by the orch — fixed as proposed: flow-mapping `--set` (D1, D12, Scope).
- fix-way-forward, LOW: the machine-layer warning named a command that cannot remove the entry — fixed as proposed: names the operator's editor session.
- fix-proposal, MEDIUM: duplicate of the `all_targets` finding — merged, fixed above.
- fix-proposal, LOW: the guard step now carries tier1's `lyx` prebuild unstated — fixed as proposed: D5's Cost names it.

## Disputed

None.

## Deferred

None.
