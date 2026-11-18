```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the reviewer's proposals the fixer carried out as proposed.

**Findings**

1. **MEDIUM** — `contracts/stencils/darn/darn-fragment-make-change-gated.md` step 3: "run the same gate call once more with the property the report's line names".
   - The report line (`internal/gatecli/tierrun.go:486`) says "it names no property". It tells the reader to take the property from the project file's `IsTestProject` condition, and to run the tests by hand with `dotnet test <project> -p:<property>=true`.
   - The sentence "the property the report's line names" is therefore false, so the writer is sent to a line that names nothing.
   - The writer also has no way to put `-p:<property>=true` into the gate call unless the tier's command carries `{args}`. The fix reproduces the reviewer's assumption ("a tier whose command takes `{args}` can carry the property") without stating the condition.
   - Fix direction: say the property comes from the project file's `IsTestProject` condition, and that it goes among the args after `--` only when the tier's command holds `{args}`. Otherwise name the tests to the parent.

2. **LOW** — the same stencil, step 3, `--gate darn-write <target> -- <args>`.
   - Explicit targets replace the key's scope, but the key's tier list, lints and `reports: [redundancy]` still run on every measuring call. The writer's "run once" call then also runs the redundancy report.
   - The args after `--` fill every tier the key lists. A key with more than one tier hands the class-selecting args to tiers whose runner may not understand them.
   - With the template's `darn-write: {tiers: [0]}` this does not bite. It does for a hub whose `darn-write` lists several tiers.
   - Fix direction: add a clause saying that when the key lists more than one tier, the args must suit each of them. This is a caveat, not a reason to drop the proposal.

3. **LOW** — `docs/dotnet-hub.md`, the added sentence "On an inert-only branch the Parent-Align and Finalize gates run no tier."
   - The sentence is unconditional. It holds only when the hub sets `inert_paths` and the parent branch is readable.
   - A tier with `carry: false` still runs whole. The review's Rejected section says this for `carry: false`, and `allScope` at `scope.go:334` and `scope.go:342` shows both conditions.
   - Fix direction: state the conditions, namely `inert_paths` set, and tiers with `carry: false` excepted.

4. **LOW** — `internal/gateslot/template.yaml` line 21, "or no tier when every changed file is inert".
   - The template ships `inert_paths: []` (line 48), and `allScope` returns whole when `len(cfg.InertPaths) == 0`.
   - The comment reads as unconditional.
   - Fix direction: add "once inert_paths is set".

F2, F5 and the F7 doc edits: no findings.
- F2: `runLints` needs only the root, and the tierless branch builds `request.root` first.
- F5 and the F7 doc edits state the same behaviour as the code.

**Tool calls**
- `grep -n "by hand\|-p:" internal/gatecli/tierrun.go`: read the report line the F3 fix points the writer at. My context held only the review's description of it.
