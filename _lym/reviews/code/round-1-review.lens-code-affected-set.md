```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines they touch.

**Per change, the set found and whether it is covered**

1. **Removed `RaddleFiling.Release`, `raddleFiled` and `Board.ReleaseTasks`.**
   - Set: `internal/boardengine/{board.go,raddlefile.go,raddlefile_test.go,doc.go}`, `internal/loomcli/raddlewiring.go`, `internal/loomshed/{raddlefile.go,raddlefile_test.go,doc.go}`, `internal/shedrecipe/{entries_raddle.go,fixture_test.go}`, `internal/testkit/envkit/envkit.go` and `internal/battencli/wire_seams_test.go`.
   - Covered. A repo-wide grep for `ReleaseTasks`, `raddleFiled\b`, `RaddleFile.Release`, `filing.Release`, `Release:` and "release the filed" finds no remaining reference. The hits that remain are unrelated lock `.Release()` calls and the "filed notice" helpers.
   - The `released` variable in `boardCriticalSection` still serves the raddle promotion; only the filing branch was removed.
   - The comment on the draft rule in `internal/boardengine/board.go:135` is about promotion only, so it is still correct.

2. **Beam-shape release step removed; the Raddle-File steps are renumbered 1–6.**
   - Set: the `Call` and `finish` comments in `internal/loomshed/raddlefile.go`, the Raddle-File paragraph in `internal/loomshed/doc.go`, the beam step-order row in `TestRaddleFile`, and the stuck-step table row for `release`.
   - Covered; all four were updated.
   - `contracts/specs/raddle-proposal-spec.md:94` mentions only the re-check, not the steps, so it needs no change.
   - `dependencyOrder` is still used by `filing` in `internal/loomshed/raddlefile.go`, so it is not orphaned. I did not re-read it, so this rests on the `finish` diff alone.

3. **The filed notice now says the tasks wait as drafts.**
   - Set: `raddleFiledNotice`, its assertion in `TestRaddleFile`, and the `raddleFiledNotice` caller.
   - Covered. The grep finds no other reader or asserter of the old notice string.

4. **Draft wording in docs.**
   - Set: the two `draft` rows in `docs/reference/weaving-glossary.md`, `internal/boardengine/doc.go` (the draft-rule paragraph and the `FileRaddle` paragraph) and `internal/loomshed/doc.go`.
   - Covered. The grep for raddle plus release in `*.md`, `*.go` and `*.yaml` finds no stale "released parts" wording.
   - `internal/boardcli/cli.go:175` and `internal/boardengine/sketch_test.go:207` describe the promotion's release, which still exists.

5. **New optional proposal key `order_departures` and the `OrderDeparture` type.**
   - Set: `internal/raddleparser/proposal.go`, `contracts/specs/raddle-proposal-spec.md`, the stencil, and `pattern/PATTERN-sole-parsers.md`.
   - Covered.
     - The type is a plain struct with json tags. The unknown-key refusal reads the tags of the receiving type (`internal/raddleparser/doc.go:10`, `proposal.go:155-200`), so the new key and its nested keys are accepted and validated with no further change.
     - `PATTERN-sole-parsers` names the parser and the spec, not the keys, so it needs no edit.
     - The spec carries the key, and the stencil points to it in line 51.
   - The `packages-shared` check at `internal/raddleparser/check.go:34` and spec line 84 are unchanged, so two unordered parts on one package are still refused.
   - The spec's example proposal (about line 129) omits the optional key. That is acceptable for an optional key.

6. **Interactive mode rules in `raddleModeRules`: dependencies table, departures with costs, drafts notice.**
   - Set: `internal/loomengine/raddletable.go`, `raddletable_test.go:84,96`, and the stencil's `{{.mode_rules}}` marker (stencil line 63).
   - Covered.
     - The test compares against the `raddleModeRules(...)` output itself, so it needs no change.
     - The wording follows `PATTERN-stencil-ownership`, which says interactive questions come from the `{{.mode_rules}}` marker that Go fills, not from stencil text (`pattern/PATTERN-stencil-ownership.md:53-54`).
     - The stencil carries the chain-order rule and the `order_departures` rule in Step 3.

7. **Declared PATTERN changes and commit messages.**
   - The diff touches no `PATTERN.md` or `pattern/` file. The directive's constraints section lists no declared changes (the rendered entries are existing ones only), so nothing is owed.
   - Neither commit message carries a `Pattern-Departure:` trailer, and the diff departs from no PATTERN entry, so none is needed.
   - Both commit messages match the diff.

**Findings**

None. Every member of the affected set that I found is updated or unaffected.

Tool calls (2):
1. Repo-wide grep for raddle plus release wording and `ReleaseTasks`, to find stale docs, comments and callers outside the diff. My context held only the diff.
2. Grep of the stencil and spec for dependency and table wording, plus `raddletable_test.go` uses of `raddleModeRules`, to find readers and asserters of the changed mode rules.
