```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set lens over F1–F10, rerunning each finding's own search for members the fix left out.

Findings:

1. **LOW** — `internal/shuttleengine/claudeengine/doc.go:544` still says "`shuttle.yaml`'s `deny_python` switches it off for a Python target repo". F7 and F8's premise is a set: every place that says false removes the python row. Lines 549–553 right below say the row is installed in every run and the hook reads the hub's value on each call, so the two sentences disagree. The fix updated `shellguard/rows.go:21`, `configcli.go:484`, `shellguard/doc.go:50` and `docs/overview.md:683`, but not this twin. `internal/languageconfig/doc.go:36` ("needs shuttle's deny_python off") is still true, so it is not a finding.

2. **LOW** — Four places describe the verb's config source as "the hub's `deny_python`" or the hub's layers, and none mention the standalone state dir the F2 fix added:
   - `claudeengine/doc.go:550`
   - `codexengine/doc.go:106`
   - `docs/overview.md:683`
   - `shuttlecli/cli.go:61` says only "resolves its own config", which is fine.

   The hub wording is true in a hub, but a standalone run now reads the state dir, so the docs under-describe the fix. `hookpython.go`'s own `Long` text and `shuttleConfigDirs` doc carry the full rule.

3. F1 (plain-text stderr) is covered for the new verb. The review calls the siblings `hook-batching` and `hook-edit` existing debt, so leaving them is correct.

4. F6 (the skip-stencil-seed annotation) was added to the new verb only. The review names no other member, so no gap.

5. F4 and F5 (keep markers) cover the two tests the review named. I did not check whether the redundancy report lists further contained tests, since it is outside my context.

6. F10 was a single site (the panic message). I found no sibling in `claudeengine`, which uses `shellGuardInstalled` and `shellGuardAnnounced`, whose unknown-switch handling is a plain false.

Tool calls:
- `grep -rniE` for stale text saying `deny_python` false removes the hook or row, across go, md and yaml, because I needed every site F7 and F8 share.
- `grep -rn` for `hook-python-allowed`, `PythonAllowedHookVerb` and `SwitchDenyPython` outside tests, because I needed every doc and caller of the verb and the switch.
- `sed` on `claudeengine/doc.go:540-556`, `codexengine/doc.go:102-110` and `shuttlecli/cli.go:58-63`, to read the doc wording at the sites the greps found.
