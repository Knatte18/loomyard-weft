```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I walked each function and branch the diff adds or changes with one concrete input.

**Per function or branch: the case I ran, and the verdict**
- `configreg.PrepareRunSettings` with the key move gone. I ran this hub's live profile block `tapestry: {loom: {plan_max_bounces: 0, …}, tapestry: {…}}` (`_bolt/_lym/config/profiles.yaml:20`) through it. `checkLocalDocument` calls `ValidateLocal`. `plan_max_bounces` is not in loom's template, which holds plan keys only in `tapestry.yaml`. It is also not among loom's `RetiredKeys` (`internal/loomengine/config.go:697-708`). The load stops with `key "plan_max_bounces" is not in the loom template` (`internal/configengine/layers.go:260-262`), and the message names no way forward. At the base, `moveRunSettings` moved the key to `tapestry` before validating (base `internal/configreg/keymove.go`). This hub self-heals on its next hub-wide reconcile: `yamlengine.FollowTemplate` drops a unit the template dropped while the file still holds the template base's value, and this hub's values match `profiles.template-base:31,46,62,78`. A tuned value, or a board entry's `run_settings` carrying `loom.plan_*`, stays and is refused. **Not correct** (finding 1).
- `runprofile.heldFormerName`, `Profiles.Settings` and `Profiles.Default`. Cases: a profile holding only a `loom` block, a `defaults` entry `loom: light`, and recipe `darn` with no default. Results: refusals naming the hand rename, and `"", nil` for `darn`. **Correct.**
- `shuttleengine.Runner.reconstruct`. Case: a candidate with `Outcome: running` and `PromptOffset: 40`. `state` is a value copy, the pid is set, the record is saved, and the replay offset is 40. A save failure is logged, as at the base. **Correct.**
- `dispositionCandidate`, tracked and live strand, with an `asking` outcome: the run is respawn-eligible. The decision record licenses removing the legacy asking outcome. **Correct.**
- `isTerminalOutcome`, with `""` (false), `stopped` (true) and `asking` (false). **Correct.**
- `fabricengine.moveWeftContent`'s new staged-only branch. Case: name `_lym` under `from`, whose last tracked files the caller staged for deletion. `PathTracked` reports false, `diff --cached --name-only` lists the deletion, and the path joins `specs`. `git commit -- <path>` matches against the HEAD tree overlaid on the index, so the deletion is committed. A failing `diff --cached` returns `committed false` before anything moves. **Correct.** One aside outside this lens: the doc comment's "A name that is not tracked under from is left alone" is now stale (`internal/fabricengine/anchormove.go:52`).
- `shedrun.runSegment`. Case: only `_lym/shed/self/` exists. It returns the slug, so the old directory is no longer read, which the decision record licenses. **Correct.**
- `tendercli.commitRunIntent` and `recordPauseIntent` without `returnOwed`. Case: a push is owed, and `owedIntentPush` answers it on stderr. **Correct.**
- `batcher.retiredKeyError`. Cases: key `weights.master_base` in the worktree layer and in `LayerDefault`. Both texts name the key and both ways forward. **Correct.**
- `shedbuild.checkRecipeShape` and `Parse` with a top-level `retired:` map. The strict decode refuses it as an unknown key with a line number. **Correct.**
- `verifytree.LandingFailure` with an old `kind: plan_verify` record. It decodes as an unknown kind string with no special mapping, which the decision record licenses. **Correct.**

**Findings**

1. **MEDIUM, design.** Location: `internal/configreg/runsettings.go:63-80`, reached through the deleted `moveRunSettings`.
   - Summary: a run-profile block or a board entry's `run_settings` that sets a plan key under `loom` is now refused, with no way forward, where it used to be moved to `tapestry`. This hub's `profiles.yaml` still holds `loom.plan_*` in every `tapestry` block. The hub reconcile removes those keys only where they equal the template base, so a tuned value or a board entry's `run_settings` value still breaks Seed-Child and the profile checks.
   - Evidence: `_bolt/_lym/config/profiles.yaml:20,33,47,61`; `internal/configengine/layers.go:260-262`; `internal/loomengine/config.go:697-708`; base `internal/configreg/keymove.go` (`moveRunSettings`).
   - Fix: add the moved `plan_*`, `plan` and `implementer` keys to loom's `RetiredKeys` with a `RetiredWayForward` naming the `tapestry` module, so the cheap refusal the decision record keeps applies. Or confirm that no board entry or tuned profile holds them.

**Tool calls**
1. Read loom's template plan keys, HEAD `runprofile.go` `Settings`/`Default`, and the attach hunk of piece 081: the manifest held only that piece's added lines.
2. Grepped for template-base handling: needed to know whether a reconcile clears the hub's `loom.plan_*` keys.
3. Grepped `configsync.go` for the seed-only follow path: same question, to locate the code.
4. Read the seed-only branch of `configsync.go` and `FollowTemplate`'s contract: how a dropped template unit is handled.
5. Ran `git show 467b8093` on the fabricengine production files: the fabric gap fix was not in context.
6. Read `moveWeftContent` whole at HEAD: needed the commit and failure path around the new branch.
