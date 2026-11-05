---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:22
    summary: The change description says the --subpath check still runs after the clone and that a mismatched --subpath under --reset can still remove the hub, which the diff contradicts.
    fix: In _lym/landing/summary.md, replace the first bullet under "Design decisions" and the "What changes" and "Tests" sections so they match the diff. State that checkSubpathBeforeReset in internal/fabricengine/resetsubpathcheck.go checks an explicit --subpath against staged shallow clones of both remotes before --reset removes an existing hub, so a --subpath that disagrees with the recorded anchor or is absent from the warp leaves the hub in place (decision record entry on --subpath). Also state that the weft-branch fetch is shared with the probe (fetchWeftBranchTip) and list the new TestCloneHub_ResetRefusalKeepsExistingHub and TestCloneHub_ResetWithSubpathPresentInWarp next to ResetWithDisagreeingShortnameKeepsHub. Delete the sentence that says the issue names only the shortname and the binding.
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/resetsubpathcheck.go:71
    summary: recordedAnchorOnWeft adds three branches no test reaches, namely the prime branch fetched from the remote, the fallback to the default HEAD when that branch lacks a marker, and the legacy marker read gated on the default branch equalling the warp branch.
    fix: In internal/fabricengine/warpbinding_clone_integration_test.go, add rows to the table-driven subtests of TestCloneHub_ResetRefusalKeepsExistingHub (or one new table test beside it), each with a sentinel in the existing hub and Subpath disagreeing with the recorded anchor. Row one puts the marker only on the remote's main-weft branch, so the FETCH_HEAD read supplies it. Row two has main-weft on the remote without a marker and the marker on the default branch main, so the fallback read supplies it. Row three has a weft whose default branch is not the warp branch and a marker on it, which must not be adopted, so the hub is removed and the clone succeeds with a --subpath that exists in the warp. Assert the refusal text names the recorded anchor and the sentinel survives, or in row three that the clone succeeds.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/fabricengine/shortnamebinding_clone_integration_test.go:71
    summary: The comment that explains the serial subtests names only ResetWithDisagreeingShortnameKeepsHub and ResetKeepsRecord, but SecondCloneDerivesShortname and DisagreeingShortnameRefuses also omit t.Parallel with no stated reason.
    fix: Make the comment above ResetWithDisagreeingShortnameKeepsHub say why every serial subtest of TestCloneHub_ShortnameRecordLifecycle omits t.Parallel (they all read the one weft fixture and hub that ResetKeepsRecord later tears down), and move it above SecondCloneDerivesShortname, the first serial subtest. Alternatively add t.Parallel to the two read-only subtests that use only a fresh t.TempDir and weftURL, which is safe because parallel subtests resume only after the parent body returns.
  - id: F4
    severity: NIT
    class: consistency
    location: internal/fabricengine/livestate_doc_test.go:183
    summary: The neutering log for row (8) still says clone.go's resetHub own pre-flight, though the looksLikeHub refusal now lives in checkResetTarget, which CloneHub calls directly in the two-argument form.
    fix: In internal/fabricengine/livestate_doc_test.go, change "Neutered clone.go's resetHub own pre-flight (the `if !looksLikeHub(hubPath)` guard" to name checkResetTarget's `if !looksLikeHub(hubPath)` guard, and change "what the pre-flight was neutered out of catching" so it no longer attributes the guard to resetHub.
---

### [MEDIUM] Change description contradicts the diff on --subpath

**Location:** `_lym/landing/summary.md:22`
**Issue:** The summary's design decision says the `--subpath` check against the recorded anchor "still runs after the clone, and so after the reset" and that "a mismatched `--subpath` under `--reset` can still remove the hub before it is refused".
The diff does the opposite: `CloneHub` calls `checkSubpathBeforeReset` before `resetHub`, and the decision record's entry for `--subpath` requires exactly that.
Its "What changes" and "Tests" sections also omit the staged check, `resetsubpathcheck.go`, and the new tests.
A claim the diff does not bear out is a finding under the rubric.

### [MEDIUM] Untested branches in recordedAnchorOnWeft

**Location:** `internal/fabricengine/resetsubpathcheck.go:71`
**Issue:** `SubpathDisagreesWithRecordedAnchor` reaches only the case where the remote has no `main-weft`, so the default HEAD supplies the marker.
No test reaches the `FETCH_HEAD` read, the fallback to the default HEAD after a prime branch without a marker, or the legacy read gated on `strings.TrimSpace(stdout) != warpBranch`.
Each mirrors a decision the post-clone code makes with `suffixWeftPrimaryBranch` and `ensureBoardWorktree`, so a drift between the two would pass every test and either refuse a valid `--subpath` or admit one the clone then refuses after the hub is gone.
This is a behavior the diff adds with no coverage.

### [LOW] Serial subtests without a stated reason

**Location:** `internal/fabricengine/shortnamebinding_clone_integration_test.go:71`
**Issue:** The new comment at line 94 correctly explains why `ResetWithDisagreeingShortnameKeepsHub` and `ResetKeepsRecord` share one hub.
Its inner `TwoArgument` and `OneArgument` subtests are covered by it, since they sit inside the commented subtest.
`SecondCloneDerivesShortname` and `DisagreeingShortnameRefuses` are serial too and carry no comment, so the requirement that a non-parallel test say why is met for only part of the function.

### [NIT] Stale resetHub wording in the neutering log

**Location:** `internal/fabricengine/livestate_doc_test.go:183`
**Issue:** The log is a record of a past run, so its outcome stays true, because neutering the guard in `checkResetTarget` gives the same gate refusal.
Only the attribution is stale.
The other comments the focus names (`livestate_matrix_test.go`, `livestate_refusal_selftest_test.go`, `livestate_verbs_test.go`) were updated and are accurate.

## Rejected

- Whether every refusal before the removal leaves the hub in place: the shortname, warp binding and old-order guard refusals run in the probe block, the non-hub refusal in `checkResetTarget`, and the `--subpath` refusals in `checkSubpathBeforeReset`, all before `resetHub`, in both argument forms. Not a defect.
- The `CloneHub` doc comment and the doc.go sentence describe the staged `--subpath` check accurately: it runs only when an existing hub is present and a `--subpath` was explicit, which is the only case where a removal could follow.
- Probe-time refusals are each pinned by a sentinel or board-record assertion that fails if the removal moves back before the probe: shortname in both forms, binding mismatch, old-order guard, and both `--subpath` refusals. No missing test there.
- The stale `.fabric-anchor` refusal in the create path still runs after the removal. It depends on the weft's content, not on an argument, so the decision record's entry on arguments does not reach it.
- `os.MkdirTemp(cwd, warpProbeDirPrefix)` in `resetsubpathcheck.go` follows the existing probe in `recordsprobe.go`, and the destructive-guard allowlist entry for its removal is present.
