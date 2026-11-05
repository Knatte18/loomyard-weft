---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/claudeengine/command.go:404
    summary: The OpenLaunch doc comment still says an opened session's settings.json holds creel's hook delivery alone, but the file now also carries attribution; the writeOpenFiles comment at line 464 is incomplete the same way.
    fix: In command.go, rewrite the OpenLaunch sentence at line 404 to say settings.json holds creel's hook delivery and the empty attribution, and nothing else, matching the wording of doc.go line 312 and settings.go line 198. Rewrite the writeOpenFiles comment at line 464 to say it writes settings.json holding creel's hook delivery over the flag directory creelFlagDirPosix and the empty attribution.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/settings_test.go:75
    summary: TestCreditNoSession duplicates coverage; its RunStandalone row repeats TestBuildSettings_MatchesGolden and its Opened row repeats checkOpenCreelFiles, which TestOpenLaunch runs. Only the RunInAHub row is covered nowhere else.
    fix: In settings_test.go, delete the RunStandalone and Opened rows of TestSettings_CreditNoSession. Keep the RunInAHub row, since the golden test never passes a creel flag directory, and keep the test only as that one case (a single-case test, with the table dropped). Correct the claim in _lym/landing/summary.md that the opened document had no test of its own content, since checkOpenCreelFiles already asserted its hooks.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: commit 16bac6c9
    summary: The commit message of 16bac6c9 ends with a Co-Authored-By trailer crediting a Claude engine, which the change's own Goal says no commit should carry.
    fix: When the commit is next rewritten, drop the Co-Authored-By Claude trailer from the message of 16bac6c9. No file edit is needed; if history is not rewritten, defer to the run's parent.
---

### [MEDIUM] OpenLaunch and writeOpenFiles comments contradict the settings document

**Location:** internal/shuttleengine/claudeengine/command.go:404, command.go:464
**Issue:** The doc comment of `OpenLaunch` says the opened session's `settings.json` holds "creel's hook delivery alone".
After this diff the document `buildOpenSettings` writes also carries `attribution`, so the sentence states something the code no longer does and contradicts `doc.go:312` and `settings.go:197-203`.
The `writeOpenFiles` comment names only the hook delivery and is incomplete for the same reason.
The Wanted list asks that the documentation state the setting beside the other settings each document carries.

### [LOW] TestSettings_CreditNoSession duplicates existing coverage

**Location:** internal/shuttleengine/claudeengine/settings_test.go:75-105
**Issue:** `RunStandalone` repeats `TestBuildSettings_MatchesGolden`, which pins the whole run document, attribution included, byte for byte across all deny combinations.
`Opened` repeats `checkOpenCreelFiles` (command_test.go:577-582), which `TestOpenLaunch` runs against the file `OpenLaunch` really writes.
Only `RunInAHub` pins a path no other test reaches, since the golden test passes no creel flag directory.
The change description's claim that the opened session's document "had no test of its own content" is not borne out: `checkOpenCreelFiles` asserted its hooks content before this diff.

### [LOW] Commit 16bac6c9 credits an engine as co-author

**Location:** commit 16bac6c9
**Issue:** The message ends with `Co-Authored-By: Claude Sonnet 5.5`.
The decision record's Goal states the operator does not want an engine credited as a co-author, and this change exists to stop sessions crediting themselves.
Commit 711aec52 carries no such trailer.

## Rejected

- Change description claims the open-launch paragraph no longer says the creel-only document holds hooks alone, while the overview diff is one added sentence (lens:code-affected-set): the claim reads as the `doc.go` open-launch paragraph at line 312, which the diff did change, and it never says the overview carried that wording.
- Scope against the Wanted list: the diff touches only `settings.go`, `doc.go`, `docs/overview.md`, tests and the golden file, and no landing code, so nothing beyond the Wanted list is changed.
- Every settings document is covered: `buildSettings` has one caller and `buildOpenSettings` one caller, so no other path composes a Claude settings document the diff leaves without `attribution`.
- Overview wording: the added sentence names `attribution` and both documents, so the Wanted list's overview item is met.
- Declared PATTERN changes: the directive declares none, and the diff touches no PATTERN file.
