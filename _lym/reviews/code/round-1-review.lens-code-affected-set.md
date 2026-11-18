```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the merge resolution and the `Names` restore force outside the lines they touch.

**Per change, the affected set:**

1. **Merge e6e36704 keeps `idBytes` and renames `maxNameLength` to the exported `MaxNameLength`.**
   - In-file uses: `CheckName` and its error message (`internal/machinename/machinename.go`) read `MaxNameLength`. Covered.
   - The only `idBytes` use is `NewID`, which reads it. Covered.
   - External callers: `internal/creelengine/crossmachine.go:52,62` read `machinename.MaxNameLength`. Covered, and `go vet` passed.
   - Nothing in `.md` files, `pattern/` or `docs/overview.md` names `maxNameLength` or `MaxNameLength`. Nothing is stale.
   - `doc.go` has no mention to update. Covered.

2. **26b29bb8 restores `Names() (current string, former []string, err error)`.**
   - Callers: `internal/creelcli/slackbridge.go:36` assigns it to a field of type `func() (string, []string, error)` (line 28), and the signature matches. `internal/slackcli/wiring.go:381` sets `MachineNames`, which `internal/slackengine/bridgein.go:407` reads as `current, former, err`, and that matches too. Covered.
   - The body is the same as `main`'s and as the version 6005496f removed. Covered.
   - Tests: the logic lives in `resolveNames`, which `TestResolveNames` and `TestResolveNames_UnreadableFileIsAnError` already cover. The wrapper itself is the code-tests lens's subject.
   - Docs and patterns: `pattern/PATTERN-owner-machine.md:42,73` and `pattern/PATTERN-leaf-packages.md` (`internal/machinename` section) describe the package's role and imports. Neither is changed by restoring `Names`, which adds no import and reads no new source. `docs/overview.md:366,895` is likewise unaffected. The only `.md` mentions of `machinename.Names` are in `docs/research/claude/code-mode-bench-2026-11-16/data/grade-small.md:53,67`. That is a dated research record, not a rule document. Covered.
   - Rule documents: no PATTERN entry forbids a thin exported wrapper. `PATTERN-test-only-code` holds because `Names` is reachable from a main package through creelcli and slackcli.

3. **Commit messages and departures.**
   - No commit in the range carries a `Pattern-Departure:` trailer, and the directive declares no PATTERN changes.
   - The 26b29bb8 message ("dropped once it had no caller") matches 6005496f.
   - The summary says `Names` was restored "unchanged" and that "every use of the length in the file already reads `MaxNameLength`". The diff and file read bear out both claims.

No stencils are touched, so the "Stencil wording" check does not apply.

**Findings:** none.

**Tool calls:**
1. Grepped `.md` files and `pattern/`, `PATTERN.md` and `docs/` for `machinename.Names`, `maxNameLength`, `MaxNameLength` and `ReadIdentity`. The manifest held Go callers only.
2. Grepped the `machinename` mentions in `pattern/PATTERN-leaf-packages.md`, `pattern/PATTERN-owner-machine.md` and `docs/overview.md`, to check whether any of them describes the changed symbols. The manifest held no pattern or doc text.
