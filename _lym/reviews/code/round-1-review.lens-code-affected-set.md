```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I checked everything outside the diff's lines that reads, names or describes the changed `--reset` behaviour.

**Per change, the set found and whether it is covered**

1. `checkResetTarget` and `hubWorkOriginLacks` now read the records store and the Bolt checkout, and count ignored files.
   - Callers `resetHub`, `CloneHub` (both forms) and `existingHubError`: updated.
   - Every path lym itself writes as ignored or untracked into the prime, the records store or `_bolt`: **not covered** (F1).
2. Refusal wording now names the work and `--discard-work` (`hubWorkWayForward`, `existingHubError`).
   - Tests asserting it, `clone_singlerepo_integration_test.go:400` and `warpbinding_clone_integration_test.go:496`: updated.
3. Plain `--reset` now refuses a hub whose records store holds unpushed commits.
   - The way forward of `RefuseHubWithoutBolt` (`clone.go:177-226`) and its descriptions in `fabricengine/doc.go:595` and `fabriccli/doc.go:32-33`: **not covered** (F2).
   - `reconcile_missingbolt_integration_test.go`: changed so it still passes, which hides F2.
4. The seed refusal in `pushclassified.go` now names `--reset --discard-work`.
   - Its doc comment, `doc.go:572` and the test constant at `clone_singlerepo_integration_test.go:845`: updated.
5. The new `DiscardWork` field and `DiscardWorkFlag` constant.
   - `cloneResetCommand`, the CLI flag, the help text, the usage string and `CloneOptions`: updated.
6. Sandbox docs that rely on `--reset` being idempotent.
   - F12 in `SANDBOX-FABRIC-SUITE.md`: updated, but its precondition conflicts with F1.
   - `SANDBOX-CORE-SUITE.md:244` (S6): **not covered** (F3).
   - The `tools/sandbox/suite.go:137` comment: still true.
7. `lymcwd.go:19` and the `livestate_doc_test.go:120` row: unaffected.
8. Repository rules. PATTERN-lym-files and PATTERN-code-conventions are followed: the skips use `lymdirs.DotLymDirName` and `fabriclayout.RecordsLockDirName`, not literals. PATTERN-refusal-way-forward binds only the webster, shed and loom packages, so fabric is outside it. This round declares no PATTERN changes.
9. Commit messages: no `Pattern-Departure:` or `Loosened-Check:` trailer is needed. The board entry is labelled `bug`, and no commit in the range carries a `Root-Cause:` trailer (F4).

**Findings**

- **F1 — BLOCKING, design.** Location: `internal/fabricengine/clone.go` `ignoredWork` (lines 1074-1098).
  - Summary: the filter skips only links, `.lym` and `.weft`. Lym writes other ignored files into every finished hub, so any hub that has been used refuses a plain `--reset` while holding nothing of the operator's.
  - Prime files: heddle's rendered `CLAUDE.local.md` and `AGENTS.override.md` (regular files, excluded by the `warp_exclude` default at `fabricengine/template.yaml:3`), plus `.claude/` and `.vscode/`.
  - `_bolt` files: `board.lock` and `board.push.lock` (`fabriclayout/hubpaths.go:16,20`), `.board-rendered.json` (`boardengine/render.go:22`), plus `.gitrepo-push.lock` and `board.json.swaplock`.
  - Evidence: `git ls-files --others --ignored --exclude-standard --directory` on the live hub `/home/knatte/Code/loomyard-LYMHUB` lists all of these, and `stat` shows the heddle files are regular files, not links.
  - The same omission makes F12's precondition ("no ignored file but lym's own links and `.lym` directories", `SANDBOX-FABRIC-SUITE.md:436`) impossible on a real hub.
- **F2 — MEDIUM, design.** Location: `internal/fabricengine/clone.go:212-218` (`RefuseHubWithoutBolt`), with `fabricengine/doc.go:595` and `fabriccli/doc.go:32-33`.
  - Summary: the refusal still names a plain `--reset` clone. A hub that stopped before its Bolt checkout holds the records store's unpushed anchor commit (two-repository layout) or its unpushed seed (`--single-repo` layout), so the clone it names is now refused. The operator gets a second refusal before reaching `--discard-work`.
  - Evidence: the diff had to add `push -q origin --all` at `reconcile_missingbolt_integration_test.go:136` (diff 001, line 136) for the named clone to apply. The stated reason is "A clone refusal names the plain --reset only for a hub whose records origin holds."
- **F3 — MEDIUM, consistency.** Location: `tools/sandbox/SANDBOX-CORE-SUITE.md:244`.
  - Summary: "Use `--reset` on the clone so repeated sandbox sessions are idempotent" is stale. The S6 scratch hub keeps the clone's unpushed module-config commit (`fabriccli/clone.go:139`) and heddle's ignored files, so a repeat `--reset` is refused. The step needs `--discard-work`, or a note that it is required.
- **F4 — MEDIUM, design (commit messages).** Location: commit range `d79a799e..HEAD` (f3b82b33d, cbb379580, fcf76fd71).
  - Summary: the entry has the `bug` label and its title calls removing unpushed work the defect, yet no commit carries a `Root-Cause:` trailer, which the rubric requires for a decision-record bug fix.
  - Evidence: `git log` output of the range, and the `lym board get` labels `["bug","fabric"]`.

**Tool calls**
- `grep` and `sed` over `livestate_doc_test.go`, `SANDBOX-CORE-SUITE.md:238-250`, `suite.go:130-140`, `fabricengine/doc.go:588-600`, `fabriccli/doc.go:14-35` and `lymcwd.go:15-22`: the manifest had only the grep hit lines, not the text around them.
- `grep` of the test files for the refusal wording: to find tests asserting the old or new messages, which the manifest did not cover.
