```yaml
lens: code-affected-set
outcome: reported
```

The code-affected-set lens leaves exactly one member of the affected set wrong: a test doc comment, at LOW. Every other caller, doc, stencil, spec and rule text matches the new behaviour.

**Per change, the set found and whether it is covered**

1. **`commentlint.Lint` now reads the working tree against a base (empty base is HEAD).**
   - `internal/loomcli/lintcomments.go:51` calls `Lint(c.env.WorktreeRoot, base)`. Covered: its file header (line 2), Long help (lines 31–39) and `--base` flag text (line 70) now describe the working tree against HEAD or against `--base`.
   - `internal/gatecli/failurereport.go:118` calls `Lint(root, "")`. Covered: an empty base still means the working tree against HEAD, untracked files included, as before.
   - `internal/gatecli/doc.go:350–357` ("Each runs over the worktree against HEAD, untracked files included", "the round and darn gates lint their own range through `loomshed`") is still correct.
   - `internal/gatecli/failurereport_test.go:189` points the reader to re-run `lym loom lint-comments`. Still correct, because the verb without `--base` runs the same check.
   - `docs/overview.md:1305–1308` is updated. Covered.
   - `docs/overview.md:406` (the package-tree line: "a diff creates, over a told worktree and base") is generic and still correct.
   - `internal/commentlint/doc.go` is updated. Covered.
   - `pattern/PATTERN-gate-slot.md:64` is unchanged and still correct.
   - Stencils and specs naming the lint are all still correct: `contracts/stencils/webster/webster-body-implementer.md:103` (verb allowed from any strand), `contracts/stencils/loom/loom-rubric-code-review.md:73` ("the gate's comment lint"), and `contracts/specs/shed-recipe-spec.md:150–155` (the verify gate's lint, with no range wording).
   - Verb-name tables need nothing: `internal/loomcli/cli.go:325`, `arm_seed_test.go:55`, `wiring_test.go:698`.
   - **Not covered:** the `TestLintCommentsCmd_OverARepository` doc comment (see finding 1).

2. **New `commentlint.LintCommitRange`, called by the verify gate.**
   - `internal/loomshed/verifygate.go:146` is the only production caller.
   - `pattern/PATTERN-gate-slot.md:65` is updated. Covered.
   - `pattern/PATTERN-gate-slot.md:60–61` ("goes through `verifytree.Verify`: it runs the comment lint") is still correct.
   - `internal/loomshed/seam_enforcement_test.go:65–67` pins the package import, not the function, so nothing changes.
   - `shed-recipe-spec.md:150` is still correct.
   - `PATTERN.md`'s `PATTERN-gate-slot` one-line entry names neither function, so it is still correct.

3. **Internal signatures of `changedGoFiles`, `readChange` and the new `lintChanges`.** All callers are inside `internal/commentlint` (`lint.go`), and the diff updates them. Covered.

4. **Repository rules, declared PATTERN changes, commit trailers.**
   - The directive declares no PATTERN changes, so nothing is owed.
   - The range carries no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed.
   - The `Root-Cause:` trailer on e1f36ce names the defect the diff removes: `readChange` read the new side from HEAD whenever a base was given, and the old `diff base HEAD` did the same.
   - No stencil changed.

**Findings**

1. **LOW, consistency.**
   - **Location:** `internal/loomcli/lintcomments_integration_test.go:22`.
   - **Summary:** the test's doc still says "At each step it asserts lint-comments and lint-pattern agree on the same tree". The step this diff adds ("lint-comments --base lints uncommitted edits too") runs only `lint-comments`. The two verbs can no longer agree there, because `lint-pattern --base` still compares `--base` with HEAD and ignores the working tree.
   - **Evidence:**
     - The test's doc comment, at `internal/loomcli/lintcomments_integration_test.go:21–23`.
     - The added subtest, which calls `lint("lint-comments", "--base", base)` alone (diff piece 003, lines 119–133).
     - `docs/overview.md:1309`: "`lint-pattern` … at `--base` with HEAD's".
   - **Fix:** limit the doc's "each step" claim to the steps that run both verbs, or say the last step checks `lint-comments` alone on an uncommitted edit.

**Tool calls**

1. grep of `.md`, `.tmpl`, `.txt` and `.yaml` files plus the loomshed seam test for `lint-comments`, `LintCommitRange` and `commentlint`: my context held no doc, stencil or seam-test readers beyond the diff.
2. grep of Go files for `lint-comments`, and of `contracts` for comment-lint wording, plus `git diff --stat`: to find Go-embedded and stencil text naming the verb that my context lacked.
3. grep of the three matched contract files: to read the matching lines.
4. `sed` of `contracts/specs/shed-recipe-spec.md:145–162`: to check whether the spec described the verify lint's range.
5. `sed` of `internal/loomcli/lintcomments.go:36–44`: the rest of the Long help, which the earlier reads cut off.
