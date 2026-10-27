```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. I looked for everything the diff's changes force outside the lines they touch.

**Per change, the set found:**

1. `commitWeftAt` now seeds excludes before staging (`recordsgit.go:376`).
   - **Production callers.** The only production caller is `Bolt.Commit`, at `bolt.go:38`.
     - Its callers are `fabriccli/clone.go:77`, `configcli/hubwide.go:145` and `configcli/hubwide.go:201`, `orchcli/rules.go:70` through `boardRulesBolt`, and `boardengine.Sync`.
     - Seeding only appends the lines `seedWeftArtifactExcludes` already owns. For a board whose gitdir was already seeded the file comes out byte-identical, so those callers see no change. Covered.
   - **Other `commitWeftAt` callers.** The only other callers are the three tests in `commitweftat_test.go`.
     - Two of them (`:57`, `:87`/`:91`) still hold: seeding does not touch the assertions on the committed file, the message, the no-trailer rule, or the clean no-op.
     - The `SkipGit` test (`:116`) still holds because the seed sits after the early return. Covered.
   - **Seed errors.** `seedWeftArtifactExcludes` already wraps its error with `fabricengine: seed weft artifact excludes in %s: %w` (`recordsgit.go:131`). The bare return at `:377` is therefore not an unwrapped sibling error. Covered.
   - **Tests that assert on the exclude file.** `weftgit_exclude_test.go` asserts byte-identical re-seeding through `Fabric.Commit`, and `hubscratch_integration_test.go:57` asserts the exclude after `CloneHub`. Neither goes through `Bolt.Commit`. `hubscratch_integration_test.go:101` calls `Bolt.Commit` after the seed is already present, so it is unaffected. Covered.
   - **Bolt tests.**
     - The new `LsFiles` assertion in `TestBolt_DirtyRepo_CommitsAndPushes` is the only one that pins seeding through `Bolt.Commit` on an unseeded fixture. No other test covers that.
     - Removing the push-lock cleanup in `TestBolt_CleanRepo_CommitAndPushAreNoOps` is sound. `Push` runs first and leaves `.gitrepo-push.lock`, `Commit` then seeds the `.gitrepo-push.lock` exclude, and the stage-all skips the file. Covered.

2. Comments and docs that describe the commit path.
   - `clone.go:331-335` is rewritten truthfully. The clone-time seed still comes before `WireJunctionsWith` and before the commit, and the "seeds again" claim matches `commitWeftAt`.
   - `doc.go:133` and the `bolt.go` and `recordsgit.go` comments agree with what `commitWeftAt` does.
   - `reconcile.go:373` says the excludes are "self-healing (every weft-git verb re-seeds them)". That is still true, and not contradicted, though it no longer mentions Bolt. `reconcile.go:405` ("Bolt.Commit is stage-all") only concerns the dirty-board gate. Neither is now wrong.
   - `commitweftpaths.go:50-55` mentions `commitWeftAt` as a stage-all with a caller-responsible lock. Both statements are still accurate.
   - `spawn.go:234` and `doc.go:294` describe a warp-side asymmetry. The diff does not touch it.
   - I found no remaining text claiming Bolt's commit seeds nothing. The one text that said so was the `clone.go` comment this diff replaced.
   - The `bolt_integration_test.go` header (lines 10-14) explains why the fixture avoids a nested bare remote. It is unaffected.

3. The `fabriccli` bypass-push test now runs through `RunCLIIn(h.BoardDir(), …)`.
   - The `RunCLIIn` seam exists at `fabric.go:479`.
   - The bypass branch resolves the hub from cwd through `lymcwd.ResolveWorktree` and `HubLogsDir(hubLocation.HubPath)` (`weft_verbs.go:253-265`).
   - The branch is covered by the new `logger.TraceDir()` assertion, so the redirect cannot silently land elsewhere.
   - The `t.Cleanup` reset is registered in the step, and the steps run serially under one parallel parent, so the sink reset does not overlap another step. Covered.
   - I found no other test or fixture that reads the changed step's behavior.

4. Repository rules and stencils.
   - The diff changes no stencil, so "Stencil wording" does not apply.
   - `PATTERN-fabric-git` and `PATTERN-durable-vs-ephemeral-state` hold: the seeding stays in `fabricengine` and uses `lymdirs.DotLymDirName`.
   - A new cross-cutting invariant needs a `PATTERN.md` entry. The diff adds none.
   - Docs land in the same commit through the `doc.go` sentence.

**Findings:** none.

**Tool calls (2 of 6):**
1. A grep for docs, comments and tests that claim Bolt seeds nothing or touch the exclude file. My context lacked the full-repo search.
2. A read of `reconcile.go:368-412`, the `bolt_integration_test.go` header, `spawn.go:228-240`, `doc.go:286-298` and `commitweftpaths.go:46-58`, to judge the comments that grep surfaced.
