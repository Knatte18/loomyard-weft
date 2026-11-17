```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, check whether everything outside the touched lines that depends on it (callers, tests, docs, stencils, rules) still holds.

**Affected sets, per change**

- **`secretpath` package (new leaf).** Its section in `pattern/PATTERN-leaf-packages.md` lands in the diff. `leaf_enforcement_test.go` follows the same shape as other packages' leaf tests (`internal/githubclient/leaf_enforcement_test.go`). `testmain_test.go` calls `tmuxkit.Main`, as PATTERN-test-isolation requires. Covered.
- **`fabric.yaml` key `secret_paths`.**
  - `template.yaml`, `ConfigSpec` (classified `CategoryGuard`) and the local-layer refusal row in `TestLoadConfig` are all updated, as are `TestConfigTemplate` and its keep reason.
  - The package doc in `fabricengine/doc.go` is updated.
  - No doc under `docs/` or `contracts/` lists the fabric.yaml keys; my grep for `warp_exclude` found none outside the template.
  - Covered.
- **`Fabric.Commit` refusal.** The doc comment, `fabricengine/doc.go`, the `lym fabric commit` help in `fabriccli/weft_verbs.go:190` and `PATTERN-fabric-boundary.md` are updated. `push` and `sync` reach it through `Commit`, and their help defers to `commit`. Covered.
- **`gitrepo.NewPathsIn` (runs git through `runChecked`).** It is pinned in `cmd/lym/gitrepoboundary_test.go`. Covered.
- **`gitrepo.AddedFilesBetween` (go-git through `readGoGit`).** No pin is needed. Covered.
- **`websterengine.Git.AddedPaths`, a new interface method.**
  - `realGit` and `fakeGit` implement it.
  - The weaver and webstercli stubs embed the interface, so they compile and panic only if called. Weaver never calls it.
  - Covered.
- **`RecordDeps.SecretPaths` and the webstercli wiring.** `wireHub` sets the loader, standalone mode leaves it nil, and the `record-batch` help is updated. Covered.
- **The new `record-batch` failure, whose way forward is reset to batch start, delete the files, then `recover-batch`.**
  - `contracts/stencils/webster/webster-template-master.md:177-181` is **not covered**.
  - `webstercli/reset.go:32-36`, the reset verb's documented effect, is **not consistent** with the new comment and way forward.
- **Commit messages.** There are no `Pattern-Departure:` trailers, and none is needed.
- **Declared PATTERN changes.** The directive declares none, so nothing is owed.

**Findings**

1. **BLOCKING · consistency · `contracts/stencils/webster/webster-template-master.md:177-181`.** The Master stencil was not updated for the new `record-batch` failure, so Master skips its reset and recovers on top of the secret commit.
   - On `{"batch_failed": true}`, rung :177 tells Master to run `lym webster recover-batch &lt;NN&gt;` at once. The only exception is a plan-edit way forward, which goes to stuck.
   - Rung :179-181 allows only `reset --to report-head`, at most once, "and no other reset". That forbids the `reset --to batch-start` this failure names as step 1.
   - Following its stencil, Master recovers the batch over the commit that holds the secret file. The summary's "Not covered" section says `recover-batch` does not run the check, so the guard comes down to one failed record before the secret stays in history.
   - The stencil needs a rung that sends this failure to stuck or to the parent. Without it, the way forward is one the receiving agent may not perform, which PATTERN-refusal-way-forward's "No denied recovery" section forbids.
   - Evidence: the stencil at lines 177-181; `internal/websterengine/recordbatch.go:640-645` (the steps `failOnSecretPaths` builds); the summary's "Not covered" section.
2. **MEDIUM · consistency · `internal/websterengine/recordbatch.go` (the `failOnSecretPaths` doc comment, ~:623-627).** The comment and the way forward contradict the reset verb's documented effect.
   - The comment says a reset to batch start "drops the commits but leaves each file on disk untracked", and step 2 tells the agent to delete the files.
   - The reset verb's help says it "moves the task worktree's HEAD, index and tracked files back to a commit the run recorded". A file the discarded commits added is tracked at HEAD and absent at the target, so the reset removes it from disk.
   - Step 2 therefore names files that no longer exist. The same claim recurs in `internal/websterengine/doc.go:409-410` and `_lym/landing/summary.md`.
   - Evidence: `internal/webstercli/reset.go:32-36`; `internal/gitrepo/reset.go:32-46` (the keep form used in standalone mode).

Tool calls:
1. Read the Master stencil's failure rungs (lines 170-182), grepped docs and yaml for listings of the fabric.yaml keys, and listed leaf-enforcement tests. My context held the stencil only as grep hits, not the rung text, and I needed to know whether a doc lists the keys or a test lists leaf packages.
