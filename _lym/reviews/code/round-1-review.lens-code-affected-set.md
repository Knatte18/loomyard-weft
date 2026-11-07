```yaml
lens: code-affected-set
outcome: reported
```

Lens `code-affected-set`: what the diff's changes force outside the lines it touches.

Findings: none.

Per change, the set I found:

1. **`gitkit.initRepo` and `initBareRemote` now write config through the new exported `WriteFixtureGitConfig`.**
   - **Callers:** only `internal/gitkit/gitkit.go` and `internal/hubforge/hub.go`. The grep for `initRepo`, `initBareRemote`, `initScratchRepo` and `initBareRepo` found no other Go caller.
   - **Tests:** `TestCopiedRepoScenario` in `internal/gitkit/gitkit_test.go` and `TestInitRepos_WriteFixtureConfigKeys` in `internal/hubforge/hub_test.go` read the keys back. Covered.
   - **Docs:** the Layer A paragraph in `internal/gitkit/doc.go` is updated in the same diff. Covered.

2. **`hubforge.initScratchRepo` and `initBareRepo` now call `gitkit.WriteFixtureGitConfig`.** The only other readers or assertions of these helpers are the new `hub_test.go` rows. Covered.

3. **The config file's shape: sections `[user]`, `[core]`, `[maintenance]` and `[gc]` are appended after `git init`'s own `[core]`.**
   - I found no other writer or reader of these fixture keys among the initialisers.
   - The remaining `git config user.name` calls (in `internal/loomshed`, `internal/fabriccli`, `internal/websterengine` and others) all target repositories those tests build themselves.
   - D4 names only the four initialisers, so those calls are out of scope, and the diff does not license touching them.

4. **Rule documents.**
   - The diff changes no `PATTERN.md` or `pattern/` file, and the constraints section declares no PATTERN changes, so there is nothing to land.
   - `PATTERN-test-isolation` (hermetic git) is unaffected: the per-repository layer stays, as D4 asks.
   - `PATTERN-gitrepo-client-boundary` concerns running git. The new code runs no git, so it is not affected.
   - `internal/gitkit` already imports `os`, `fmt` and `filepath`, and `hubforge` already imports `gitkit`. No new import edge appears.

5. **Commit messages.** The three commits carry no `Pattern-Departure:` trailer, and the diff differs from the decision record nowhere that needs one.

6. **Stencil wording.** The diff changes no stencil, so there is nothing to check.

Noted, not a finding: `docs/benchmarks/fixture-copy.md:104` says "`lymtest.initRepo` and `initBareRemote` set" the keys. It was already stale before this diff (`lymtest`), and it stays true that those initialisers set the keys, so the diff does not force an edit there.

Tool calls (1):
- One grep across `.go` and `.md` files for the initialiser names and the quiet-config wording, for the other `git config` writers of these keys, and a `git diff --stat` of `PATTERN.md` and `pattern/`. My context held no repo-wide caller or doc search.
