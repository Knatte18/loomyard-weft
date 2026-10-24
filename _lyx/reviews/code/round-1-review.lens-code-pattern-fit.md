```yaml
lens: code-pattern-fit
outcome: reported
```

Task: check the diff's fit with the repository's rules (PATTERN.md and `pattern/`), the declared PATTERN changes and the commit trailers.

**Rules checked**
- **Declared PATTERN changes landed — held.** I compared the `PATTERN.md` diff line by line against the declared-changes list. The `entry_gate`, `command-row`, `shed-recipe-registry`, `trimpath-safe-root`, `role-skills-typed`, `template-overlay`, `shell-guard-table`, `lyxdirs-single-declarer`, `treadle-runner-seam` and `binary-change-reconcile` lines are present. So are `shed-producer-seam`, `parent-lock`, `agent-filed-issues`, `fabric-write-containment`, `origin-reconcile`, `never-force-add`, `recipe-owns-failures`, `hub-store-gc`, `owner-machine` and `pane-binary-resolution`. In total 23 declared entries land with the exact declared text: `test-tiers` is changed, `tier-tags`, `gate-carry` and `row-entry-gate` are added, and the rest are shortened.
- **`PATTERN-documentation-lifecycle`, same-diff docs — held.** The diff updates `pattern/PATTERN-test-tiers.md`, `PATTERN-test-speed.md`, `PATTERN-verified-tree.md` and `PATTERN-recipe-setting-lookup.md`, plus `docs/overview.md` and the `doc.go` files of `gatecli`, `gateslot`, `shedengine`, `shedbuild`, `darnengine` and `landingshed`. The `entry_gate` bullet is added to the recipe-setting-lookup background file.
- **`PATTERN-test-speed` and `PATTERN-tier-tags` backgrounds — held.** The tier table, the disjoint-tags rule, the guard `tierN || integration` transition rule and the one-tag enforcement pointer (`cmd/lyx/tiertags_test.go`) are written in.
- **`PATTERN-test-tiers` — held.** The background file now says a site names a gate key and nothing else, with the darn tree-scoped targets as the one addition.
- **`PATTERN-shed-producer-seam` — held, from the diff only.** `EntryGate` is a plain function type on `ProducerDef` in `internal/shedengine/producer.go`. `entry.go` imports only stdlib and `state`.
- **`PATTERN-failure-stop-visible` — held.** `failEntry` writes the status `failed` before it returns the error (`entry.go`).
- **`PATTERN-tolerant-records` — held.** `EnteredRow` and `Reason` are `omitempty` fields on the status, and `readGreen` treats an absent, unreadable or malformed file as an empty record.
- **`PATTERN-gate-slot` and `PATTERN-gate-carry` — held.** In `internal/gatecli/run.go`, `runGreenAware` returns through `reportCarried` before `runWithSlots`, so a carried tier takes no slot. The record is read only when the worktree is clean (`openGreen`), and `stillGreen` rechecks the tree before a write.
- **`PATTERN-refusal-way-forward` — held for the refusals I saw.** The `--gate` refusals, `gateKeyRefusal`, the empty-tier failure and the chain-length and evaluation-error failures each name a way forward.
- **Commit messages and departures — held.** The grep over the range's commit messages returned no `Pattern-Departure:` trailer, so there is nothing to validate.
- **Not verified:**
  - Whether the green record's directory is named through `internal/lyxdirs`. `green.go` uses `gateslot.WaitDir(location.AnchorPath())`, and the `lyxdirs` indirection is not in my context.
  - `PATTERN-sole-parsers` for the `shedbuild` `entry_gate` parser.
  - `PATTERN-binary-change-reconcile` and `PATTERN-hub-wide-config` for `gateslot.MigrateConfig`.
  - `PATTERN-comment-line-breaks`, which the rubric leaves to `lyx loom lint-comments`.

**Findings:** none.

**Tool calls (1 of 6)**
1. `git diff` of `PATTERN.md` and `pattern/` over the review range, plus a grep of the commit messages for `Pattern-Departure`. My context held no `PATTERN.md` diff and no commit-message trailers.
