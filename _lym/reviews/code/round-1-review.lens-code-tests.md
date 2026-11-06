```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: behaviour → pinning test**

1. Withdraw strikes a Discussion bullet, an appended bullet, or the last bullet (the heading stays), and adds the `Withdraws:` line.
   - Pinned by `TestAppendDecision_WithdrawnPatternStrikesBullet` rows `DiscussionBullet`, `AppendedBullet` and `LastBulletKeepsTheHeading`.
   - These rows also run `CheckAddedDecisionsKept` on the result.
2. Withdrawing an id the record has no bullet for, or a record with no section, returns `ErrPatternBulletAbsent` and writes nothing.
   - Pinned by the same test, rows `NoSection` and `NoBulletForTheID`.
3. A `Withdraws:` line exempts its id from `pattern-change-dropped`.
   - Pinned by two new `TestCheckAddedDecisionsKept` rows: one where the line sits in an added entry, one where it sits outside an entry and does not exempt.
   - The fenced-block exclusion (`!fenced[i]` in `withdrawnPatternIDs`) has no row.
4. The `patternAt < at` ordering branch in `insertDecision` for a withdrawal has no row.
   - That is a PATTERN changes section before `## Decisions`.
   - The `patternEnd < len(content)` guard (a bullet on the last line with no trailing newline) also has no row.
5. `patternFormBroken` refuses text beside a withdrawal, in both flag form and `--from` form.
   - Pinned by `TestDecisionVerb_Refusals` rows `pattern text beside a withdrawal` and `from entry with text beside a withdrawal`.
   - A withdrawal with an entry but no action, or the reverse, falls under the existing partial-form rows.
   - Two `--from` entries naming one id, a withdrawal included, fall under the existing "both change" row.
6. `decisionVerb` passes `WithdrawnPattern` to the append and returns the id under `pattern_withdrawn`.
   - Pinned by `TestDecisionVerb_AppendsAndCommits`, row `parent label withdrawing a PATTERN change`.
   - Only the flag form is exercised. A `--from` entry with `pattern_action: withdraws` that succeeds is not.
7. `ErrPatternBulletAbsent` is refused with `decisionWayWithdraw`.
   - Pinned by `TestDecisionVerb_Refusals` row `withdrawal of an entry the record does not declare`.
   - It runs through a fake `appendErr`, not through a real append.
8. `MapPatternChanges` rewrites a bullet in place as `changes` (trimmed text), returns the record unchanged for no mappings, and errors on an id the record lacks.
   - Pinned by `TestMapPatternChanges`.
   - Two mappings in one call have no row.
9. `raddlefile.writeRecords` files a mapped change under the live entry, and the store keeps the Sketch's own record.
   - Pinned by `TestRaddleFile`, subtest `MappedPatternChangeIsFiledUnderItsLiveEntry`, for all three shapes.
   - It reads only `PartRecordPath("part-a")`. For the project shape, the project record (`proposal.Project.Slug`) is never asserted to carry the mapped bullet, though the spec says it does.
   - The `MapPatternChanges` error branch in `writeRecords` has no row.
10. `Part.PatternEntryIDs` replaces a mapped id by its entry for the spliced record.
    - Pinned only indirectly, by the tasks shape in item 9.
    - A part with both mapped and unmapped ids, or a mapping whose id the part lacks, has no row.
11. The `pattern-cluster-invalid` check, for a change the part lacks, a mapping written twice, empty text, two-line text and over-cap text.
    - Pinned by `TestCheck` rows `ClusterOfAChangeThePartDoesNotCarry`, `ClusterWrittenTwice`, `ClusterWithNoText`, `ClusterTextOfTwoLines` and `ClusterTextOverTheCap`.
    - `PatternTextOverCap` has no direct test, but the over-cap row covers it. Its boundary (exactly at the cap) is not pinned.
12. The `pattern-cluster-unknown` check.
    - Pinned by `TestCheck` row `ClusterOntoAnEntryPatternMdLacks`.
13. The `pattern-cluster-shared` check, against another mapping or another declared change.
    - Pinned by `TestCheck` rows `TwoPartsMapOntoOneEntry` and `ClusterOntoAnotherDeclaredEntry`.
    - A valid mapping passes (`ChangeMappedOntoALiveEntryPasses`).
14. `raddleBoardContext.LivePattern` answers from the worktree's `PATTERN.md`.
    - Pinned by `TestRaddleBoardContext` for a live and an absent id.
    - A worktree with no `PATTERN.md` (the `ErrNotExist` branch) and a non-`ErrNotExist` read error have no row.
15. The `pattern_clusters` JSON key is accepted and parsed by `raddleparser.Parse`.
    - Pinned by no test: nothing in the diff's tests mentions the key.
    - `Parse` goes through `rejectUnknownKeys` with reflection, and its handling of a nested slice-of-struct key and unknown keys inside a cluster is untested.
    - This is the only route by which a real proposal reaches the new field.
16. The 2000-byte `Long` cap on `lym loom decision add`'s help.
    - Whatever existing test enforces the cap covers it; the diff adds none. That is correct, since this is static text.

**Findings**

- **MEDIUM, design.** Location: `internal/raddleparser/proposal.go:60-100` (`Parse`, `Part.PatternClusters`).
  - The new `pattern_clusters` key, with `sketch_id`, `entry` and `text`, is never exercised through `Parse`.
  - Evidence: the grep for `pattern_clusters` across `*_test.go` returns nothing, and every test builds `PatternClusters` as a Go literal.
  - Fix: add a `Parse` row for a proposal carrying `pattern_clusters`. Add one that refuses an unknown key inside a cluster.
- **LOW, design.** Location: `internal/loomshed/raddlefile_test.go:218-239`.
  - The project shape's project record is not asserted to carry the mapped bullet.
  - Evidence: the test reads only `f.deps.PartRecordPath("part-a")`, while `writeRecords` writes `proposal.Project.Slug` from the mapped record and the spec claims it.
  - Fix: for `ShapeProject`, also assert the project record at `PartRecordPath(proposal.Project.Slug)`.
- **LOW, design.** Location: `internal/loomcli/decision_test.go:343-400`.
  - The `--from` path with `pattern_action: withdraws` has no success row.
  - Evidence: the new success row (`parent label withdrawing a PATTERN change`) uses the flag form only. The `--from` coverage is refusal-only.
  - Fix: add a `--from` row that withdraws and asserts `pattern_withdrawn`.
- **LOW, design.** Location: `internal/discussionparser/append_test.go:190-262`.
  - The `patternAt < at` branch and the unterminated-last-line bullet strike have no row. A fenced `Withdraws:` line has none either.
  - Fix: add a record with the PATTERN changes section before `## Decisions`. Add one where the bullet is the file's last line with no trailing newline. Add a fenced `Withdraws:` row to `TestCheckAddedDecisionsKept`.
- **LOW, design.** Location: `internal/loomcli/raddlecontext_integration_test.go:43-60`.
  - `LivePattern` against a worktree with no `PATTERN.md` is untested.
  - Fix: add a case with no `PATTERN.md` that asserts `LivePattern` is false for every id.

**Tool calls**
1. `grep` of `*_test.go` for `pattern_clusters`, `PatternClusters` and `PatternEntryIDs`, plus the `Parse` and unknown-key functions in `proposal.go`. My context held the diff hunks only, not whether `Parse` is tested for the new key.
