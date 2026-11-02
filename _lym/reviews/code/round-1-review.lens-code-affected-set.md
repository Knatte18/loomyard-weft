```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's `{test_timeout}` / `test_timeout_min` change forces outside the lines it touches.

**Per change, the set found:**

1. **`{test_timeout}` placeholder, rendered in `renderTierCommand` (`internal/gatecli/tiers.go:302-326`).**
   - The only renderer of `tier.Command` is `renderTierCommand`. The other `tier.Command` readers (`tiers.go:97,154,189,198`, `green.go:315,359,419,470`) only test for `{targets}` or `{args}` and never render the command.
   - `build.go:64` passes the build step's command, not a tier's, so it is not a bypass.
   - The `strings.NewReplacer` replaces `{test_timeout}` as `&lt;n&gt;m` on the single render path.
   - Covered.

2. **`Tier.TestTimeoutMin`, `DefaultTestTimeoutMin`, `TestTimeoutPlaceholder`, `decodeTier` validation (`internal/gateslot/tiers.go`).**
   - Readers of `TestTimeoutMin` are `gatecli/tiers.go:325` and the tests. The decoder is the only writer.
   - Docs:
     - `internal/gateslot/doc.go:66,77` carry the key and its refusals.
     - `template.yaml:6,8` carry the placeholder and key.
     - `internal/gatecli/doc.go:32-33` carry the placeholder and its rendering.
   - The `Tier.Command` field comment is updated.
   - Covered.

3. **Green fingerprint (`internal/gatecli/green.go:64-96`).**
   - `TestTimeoutMin` is not in the hashed `declaration`, so it is excluded. `Command` is hashed with the placeholder unrendered, so editing the command invalidates carried passes and editing only the minutes does not.
   - The `green.go` comment, the `gatecli/doc.go:183` list and the `green_test.go` subtest are updated.
   - Covered.

4. **Repository docs and PATTERN.**
   - `docs/overview.md:498`, `docs/dotnet-hub.md` and `README.md` list no placeholder set or tier key set that would need `{test_timeout}`.
   - The `timeout_min` grep found only the shed, webster, loom and verifytree modules, which are unrelated tier timeouts.
   - `PATTERN-gate-slot` (a gate and its self-check call one function) is respected, since there is one renderer.
   - The directive declares no PATTERN changes, and none are in the diff.
   - Covered.

5. **Commit message / `Pattern-Departure`.** The commit carries no trailer and the diff departs from no entry. Covered.

6. **The repository's own tier1 command (the real goal).**
   - No in-repo file holds `{test_timeout}` or `-timeout`. The only matches are `internal/gateslot/template.yaml` and the diff's own tests and docs.
   - The hub's `gate.yaml` is not in this checkout. `find` found no `gate.yaml` within three levels above it and no `gate.yaml` in the repo tree.
   - The placeholder is therefore a mechanism only, and the panic persists until the hub's tier1 command writes `go test -timeout {test_timeout}`.
   - Whether the hub config falls in this diff's scope is for the handler to decide. I record the gap and raise no finding, since the hub file is outside this worktree.

**Findings:** none from this lens.

**Tool calls:**
1. `grep` for `{filter}` and `tier.Command` readers, to find every site that reads or renders a tier command or documents the placeholders. My context lacked callers outside the diff.
2. `grep -l timeout_min` across go, md and yaml, to find docs or specs that list tier keys. My context lacked a repo-wide list.
