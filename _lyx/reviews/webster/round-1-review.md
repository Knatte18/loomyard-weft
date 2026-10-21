---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-pattern-fit
    location: PATTERN.md:14
    summary: The declared PATTERN-sparse-hub-base entry line is 350 characters, over the 320-character cap, so TestOverview_RealFilePassesCheck fails at the branch tip (verify-8.log).
    fix: Keep the declared entry text verbatim in PATTERN.md line 14 and make the length gate admit it. In internal/pattern/check.go raise MaxEntryLineChars from 320 to 360, then update the cap sentence in internal/pattern/doc.go and any test row in internal/pattern that pins 320. Run lyx gate test ./internal/pattern and confirm TestOverview_RealFilePassesCheck passes.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-pattern-fit
    location: PATTERN.md:13
    summary: Commits 8b07cacf and c4360b46 shorten eleven PATTERN entries that no declared change lists, dropping clauses the overview currently carries.
    fix: In PATTERN.md restore the parent branch's text (git show 90f613277e539c98eb07698ed58b50f705e99a77:PATTERN.md) for PATTERN-hub-wide-config (line 13), PATTERN-transient-stop (58), PATTERN-driver-choice-single-site (60), PATTERN-batcher-registry (69), PATTERN-creel-store (76), PATTERN-stencil-ownership (77), PATTERN-spawn-observability (95), PATTERN-leaf-packages (104), PATTERN-test-isolation (116), PATTERN-markdown-link-integrity (122) and PATTERN-comment-line-breaks (123). Only PATTERN-sparse-hub-base is a declared change. If the restored overview exceeds the 16 KiB cap, recover bytes from the declared entry's own links or from prose outside the entries, never from an undeclared entry's text. Keep every restored line within the cap F1 sets.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/testkit/lyxbin/lyxbin_integration_test.go:243
    summary: RunWithEnv and the non-nil extraEnv branch of runUntilDone have no test in the lyxbin package, because TestRun_PassesTheEnvironment drives Run, which passes nil.
    fix: Add one row or assertion beside TestRun_PassesTheEnvironment that calls RunWithEnv with an extraEnv entry setting helperEchoEnv to a value different from the one t.Setenv sets, and asserts the child's output is the extraEnv value. Keep it in that test, not a new one.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/smoke_prebootstrap_integration_test.go:204
    summary: The local skipPush duplicates skipWalkPush from smoke_helpers_test.go, which the integration-tier file cannot reach because that helper file is tmux-tagged.
    fix: Define one shared variable in an untagged loomcli test file that both tiers compile, and delete the local skipPush at smoke_prebootstrap_integration_test.go:204 and the skipWalkPush definition in smoke_helpers_test.go, pointing the callers at the shared variable.
---

### [BLOCKING] Declared sparse-hub-base entry exceeds the entry-line cap

**Location:** PATTERN.md:14, internal/pattern/check.go:27
**Issue:** The declared text for `PATTERN-sparse-hub-base` is landed verbatim, as the declared changes require.
With its background link the line is 350 characters, and `MaxEntryLineChars` is 320.
`.lyx/verify/verify-8.log`, the latest verify run, fails `TestOverview_RealFilePassesCheck` with "entry line is 350 characters, over the 320 cap".
The declared text and the length gate conflict.
The declared text wins, per the constraints section, so the gate moves and the entry does not shorten.

### [BLOCKING] Undeclared PATTERN entries shortened

**Location:** PATTERN.md:13, 58, 60, 69, 76, 77, 95, 104, 116, 122, 123
**Issue:** Against the parent branch's `PATTERN.md`, eleven entries lost clauses in `8b07cacf` and `c4360b46`, for example "set only at a producer or step-bootstrap boundary", "a pane only via its `Delivery` seam", "tagged packages run through `tmuxkit.Main`" and "checked by `lyx loom lint-comments` alone".
The declared changes list `PATTERN-sparse-hub-base` alone, so every other entry is judged by its current overview text.
`500a805e` restored six entries to that text and `c4360b46` trimmed shorter variants again to fit the headroom.
Where a background file holds the dropped clause, the overview still loses the rule's current wording.

### [LOW] RunWithEnv's extraEnv path has no test

**Location:** internal/testkit/lyxbin/lyxbin_integration_test.go:243
**Issue:** `94bc0800` adds the exported `RunWithEnv` and the `extraEnv` branch of `runUntilDone`.
`TestRun_PassesTheEnvironment` drives `Run`, which passes nil, so no lyxbin test fails if `extraEnv` is ignored.
Only the loomcli smoke tests exercise it, and an ignored skip variable there shows up as a race, not a deterministic failure.

### [LOW] Duplicate push-skip variable across test tiers

**Location:** internal/loomcli/smoke_prebootstrap_integration_test.go:204
**Issue:** The integration-tier file defines a local `skipPush` equal to `skipWalkPush` in `smoke_helpers_test.go`, because the tmux-tagged helper file is not compiled in that tier.
Two copies of the same skip list can drift apart.

## Rejected

- Publish failure `TestLoomResumeReconcilesThenRefusesWithoutLiveDriver` (verify-5.log, TempDir `tmp_objdir-incoming-*: directory not empty`): its cause is the detached push that `loom resume` fires after reconciling an unstamped hub, which wrote into the bare weft repo during TempDir cleanup.
  Commits `4d34dffe` and `94bc0800` skip that push for the test through `skipWalkPush` (`smoke_resumearming_test.go:32`), and verify-8.log shows `internal/loomcli` passing, so no finding remains to fix.
- The tmuxkit leak notice for key `lyx-p61b6f42` in the same run: the key has the form `lyx-p%07x` that reed gives its capability probe (`internal/reedengine/probe.go:146`), and `76abcca8` exempts that form in `isReedProbeKey`. verify-7 and verify-8 no longer report it.
- Merge-in diff `860b0cd9^1..860b0cd9`: no change in it leaves a tmux server or push process that the failing test omits to register or wait for; the leak was the probe key and the push above.
- `TestLoomFailedReedUpRefuses` lacking `FABRIC_SKIP_PUSH` (lens:code-affected-set): its fixture commits no retired batcher key, so no reconcile commit is shown to fire a push, and verify-8.log passes it.
- Opt-in `skipWalkPush` per call site as a recurrence risk (lens:code-correctness): speculation about future tests, no present defect.
- Missing `lyx-pzzzzzzz` row in `TestCheckSockets` (lens:code-correctness, lens:code-tests): `lyx-pnothex` and the wrong-length case already pin the rejection of a non-probe key.
- Commit message of `07c15a65` says "fix F1" but adds only a test row (lens:code-tests): the commit message is not a defect against the plan, and the row pins the stated behavior.
- Test economy over the new `loomcli` smoke and resume tests: no duplicate found.
  `TestLoomResumeReconcilesThenRefusesWithoutLiveDriver` covers the half past the reed config load, and `TestLoomResumeReconcilesTheHubConfigBeforeArming` covers the half that refuses at it, as `smoke_prebootstrap_integration_test.go:182` states.
