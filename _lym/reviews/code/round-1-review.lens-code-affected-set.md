```yaml
lens: code-affected-set
outcome: reported
```

I checked what the doc.go trim forces outside the lines it touches, and found one finding.

**Per change, the set found, covered or not**

**fabricengine/doc.go**
- `cmd/lym/destructiveguard_test.go:115` and `cmd/lym/uncontainedwrite_test.go:80`: covered. Both allowlist reasons say the doc names the banned tokens, and it still does: `os.RemoveAll`/`os.Remove`, `git worktree remove` and `git branch -D` at doc.go:320; `os.WriteFile` at :387; `reset --hard` at :35 and :472.
- `destructiveguard_test.go` at :248, :265, :289 and :313: covered. "The mutation record" section is at doc.go:280, and the doc names the guard test at :314. "Mutation Record Invariant" is absent at the base too, so that is not this diff's doing.
- `pattern/PATTERN-lym-files.md:124`: covered. The Windows/fslink settling text is kept at doc.go:533–547.
- `docs/overview.md:271` (the suffix): covered at doc.go:11.
- `mergein_recovery_integration_test.go:672` (plain-git last resort): covered at doc.go:472.
- `docs/overview.md:2056` (state matrix): it dangled before this diff, so it is not a finding.

**adoptcli/doc.go**
- `internal/adoptcli/cli.go:47` (the `lym adopt` help text): **not covered**. See finding 1.

**websterengine/doc.go**
- `contracts/specs/webster-spec.md:13-15` (fork mechanism, bracket verbs, audit policy, crash/resume, verify-gate mechanics): covered. The sections are kept, condensed, at doc.go:112, :143, :200 and :220–237.
- The removed heading "The verify gate" is matched in `loom-plan-spec.md:318` and the recipe YAMLs only as generic prose, not as a citation.

**batcher/doc.go**
- `contracts/specs/loom-plan-spec.md:37`: covered, since the batching description remains.

**shedbuild/doc.go**
- `internal/shedbuild/build.go:35` (which constructor produces which effect): covered by the closing paragraph. The removed "Blocks" heading has only generic matches elsewhere.

**shedrun/doc.go**
- `contracts/specs/loom-status-spec.md:4` (seed.json): covered at doc.go:3–17.

**reportengine/doc.go**
- `internal/reportcli/doc.go:10` ("describes every figure"): covered. Every figure heading survives; Stops and Repairs are merged into "Wakes, stops and repairs".

**githubclient/doc.go**
- `githubclient.go:3` (resolution chain, cache, transport): covered at doc.go:9–31.

**gitexec/doc.go**
- `gitexec.go:17` (process-group kill, low-speed limit): covered at doc.go:18–54.

**shellguard/doc.go**
- `claudeengine/doc.go:250` ("documents what each row matches"): covered at doc.go:2–20.

**orchcli/doc.go and orchengine/doc.go**
- `orchengine/doc.go:204` (withdraw around a fresh launch): covered by the orchcli section at :88.
- `orchcli/doc.go:28` (`Continuing after a reboot`): the summary says it is kept; I did not re-grep it.

**Other docs**
- `configengine`: `docs/shared-libs/configengine.md:33` and `docs/overview.md:165/213/520` are covered by `# Layers` at doc.go:22.
- `planglyph`: `loom-plan-checks-spec.md:14` is covered; the canonical finding list is at doc.go:27.
- `shedcheckkit`: `docs/overview.md:1596` is covered by "The finding kinds" at doc.go:20.
- `state`: `PATTERN-tolerant-records.md:7` (merge rules) is covered at doc.go:23–27.
- `loomcli`: `docs/overview.md:1183` (recorded seed driver) is covered at doc.go:63–64.
- `hardenshed`: `docs/overview.md:1672` (the campaign's method) is covered by `# Method` at doc.go:65.
- `shuttleengine`: `docs/overview.md:1921` (economic constraint) is covered at doc.go:7.
- `reedengine`: `docs/overview.md:185` (`reed.json`) is covered at doc.go:79 and :201. The `display-message` contract cited by `attach_test.go:5` is covered at doc.go:204.

**Removed headings matched elsewhere**
- "Run settings", "The parent seat", "Store housekeeping", "Keys the template lacks", "Remote control", "The HardenFix row…", "Worktrees", "Imports", "Input", "Prompt", "Example", "Wiring" and the gatecli report headings all match unrelated text, not citations.
- I did not open `README.md`'s match for "The steps".

**Rule checks**
- PATTERN names the diff adds (config-layers, cwd-resolution, fabric-boundary, gitrepo-client-boundary, owner-machine, reed-panes, run-records, shed-driver, shuttle-provider-seam): every one exists in PATTERN.md.
- Declared PATTERN changes: none are declared, so there is nothing to land.
- `Pattern-Departure:` trailers: none are in the range.
- Stencil wording: no stencil is changed.

**Findings**

1. **MEDIUM**, class `consistency`, `internal/adoptcli/doc.go` (the `# The steps` section and the envelope field list are removed).
   - Summary: the `lym adopt` help text at `internal/adoptcli/cli.go:47` still says "The package doc holds the steps, the hold and every field of the envelope." The trimmed doc dropped the `# The steps` section, and its `# The envelope` section no longer lists any field.
   - The dropped fields, at base doc.go:152–166, include `slugs`, `prime_recorded`, `hold`, `fix_task`, `prime_check`, `check_lines`, `deploy`, `warnings`, `prime_beam_steps`, `prime_beam_warning`, `archives`, and per-slug `owned_by`, `check` and `beam_steps`.
   - The envelope's field set is the verb's output contract, which the answer key keeps. Commit 20ad9235's claim that "sentences other files cite stay" is false here.
   - Evidence: `internal/adoptcli/cli.go:47`, HEAD `internal/adoptcli/doc.go` "# The envelope" (2 lines), base `internal/adoptcli/doc.go:37` and :152–166.
   - Fix: put the envelope field list back in adoptcli's doc.go, or reword `cli.go:47` to match what the doc now holds. Changing `cli.go` would be a code change, which the task's "No code changes" rule forbids, so restoring the doc is the fix that fits.

**Tool calls**
1. Read the fabricengine allowlist reasons in destructiveguard_test.go and uncontainedwrite_test.go and grepped the doc for raw tokens: the manifest had only the hit lines, not the reasons' claims.
2. Grepped for section-level citations into package docs: the earlier grep's keyword filter was too narrow.
3. Read the context of each new citation: it was needed to learn what each one claims.
4. Read adoptcli and shedbuild at HEAD, plus the planglyph, loomcli, shedcheckkit, state and configengine lines: to check whether those citations' content is still there.
5. Read adoptcli's base envelope section, the websterengine, reportengine, githubclient and shellguard headings and lines: to check the remaining citations.
6. Read websterengine's verify-gate and crash/resume text against the base: to settle the webster-spec citation.
