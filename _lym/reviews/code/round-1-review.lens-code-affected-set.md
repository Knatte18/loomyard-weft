```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set — for each change the diff makes, what reaches it from outside the touched lines, and whether the diff covers it.

**Per change, the set found:**

- **`lym gate testmap` verb removed (`cli.go` `testMapCmd`, `testmapaudit.go`, `testmapwait.go`, `testmaporder_test.go`, planning code).**
  - A repo-wide grep for `testmapwait`, `testMapCmd`, `testmapaudit`, `PlanUpkeep`, `Selector`, `declcompare`, `testselect`, `FailedAt`, `ProfiledFuncs`, `GoRoot(` and bare `testmap` finds no live reference.
  - The only hits are under `docs/research/`, which the record keeps as historical.
  - Covered.
- **Live text naming the verb.** The diff updates `README.md`, `docs/overview.md`, `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, `internal/dobbyengine/doc.go` and the `template.yaml` comments, and a grep finds no stale mention left. Covered.
- **PATTERN backgrounds.**
  - `PATTERN-gate-slot.md` now says "three places" and names the map run with the larger of tier 0's and tier 1's slot asks, where a tier asking for every slot ranks above any count.
  - `PATTERN-daemon-wakeups.md` loses the `testmapwait.go` bullet, and the `cmd/lym/daemonwakeups_test.go` row goes in the same diff.
  - `PATTERN-test-speed.md` rewrites its coverage-map section. Its overview line is unchanged, as the record requires.
  - `gateslot/doc.go` drops the testmap holder from its list of non-tier holders, as the constraint asks.
  - Covered.
- **`Pool.TryAcquire` removed (`gateslot/pool.go`).** Its `pool_test.go` tests and the doc line go too, and no caller is left. Covered.
- **`Map.FailedAt`, `stampDifferences` and `Toolchain.GoRoot` removed.** The `testcover/doc.go` sentences naming them are updated, and their tests are removed. Covered.
- **`ProfiledFuncs` removed and `funcsHolding` simplified (`testcover/blockkey.go`).** `doc.go` is updated. `CoveredFuncs` behaves as before, so old maps load unchanged. Covered.
- **`lym gate redundant` refusal text (`redundant.go`).**
  - `rebuild` now names `lym gate test --map <pkg>`.
  - The "other Go files" refusal adds the commit-or-revert advice, and its help text is updated.
  - `redundant_integration_test.go` is in the diff. Covered.
- **`--map` flag refusals and detach key (`run.go` `flagShapeRefusal`, `detachfront.go` `words()`).**
  - `cli_test.go` rows reach each refusal: `--tier`, `--upto`, `--gate`, `--guards`, `--stop-on-fail`, `--tags`, `-C` and words after `--`.
  - `detachfront_test.go` pins `--map` in the replay words.
  - Covered.
- **`prebuild.go` header.** It no longer names the testmap verb. Covered.

**Findings:**

1. **LOW, consistency** — `internal/gatecli/cli.go`, `testCmd` `Long` text, around lines 78–95.
   - The diff deletes help lines the answer key never asked to change.
   - Two rules go: "Inside a slot already held by an enclosing gate run it runs without acquiring another." and "Each tier prints a summary line to stderr and writes its record and log under the gate directory."
   - Two examples go too: `--wait --gate finalize` and `--upto 2`.
   - D1 only needs a `--map` line and an example.
   - `doc.go` still documents the rules, but the help is what an agent reads.
   - Evidence: `git diff` hunk of `cli.go` at `@@ -78,20 +78,18 @@`.
   - Fix: restore the two rules and the two examples beside the new `--map` lines. If a help-size limit forced the cut, record that in a `Pattern-Departure:` trailer.
   - No `Pattern-Departure:` trailer appears in the range's commit messages.

**Tool calls (3):**
1. `grep` for stale names, plus `git diff` of `pattern/`, README, overview, gateslot, dobbyengine and `daemonwakeups_test.go`. The manifest lacked the pattern and doc hunks.
2. Bare-`testmap` grep, `git diff` of `cli_test.go` and `detachfront_test.go`, a `Pattern-Departure` grep in commit messages, and a grep for help-size tests. The earlier grep was truncated and the test hunks were unread.
