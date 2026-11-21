```yaml
lens: code-affected-set
outcome: reported
```

Task: review the code-affected-set lens. For each change the diff makes, find everything that calls, reads, asserts or describes it, and flag any of it the diff leaves wrong.

**Per change, the set found:**

1. **`rootProcesses` also matches the root with its links resolved** (`internal/hardenshed/reap.go:182-205`).
   - The signature is unchanged. Its only caller is `Reap` (`reap.go:128`), which still reads it correctly.
   - The production wiring is `HardenReapDeps` (`internal/loomcli/hardenwiring.go:183-184`). It passes `SandboxRoot(location.WorktreePath())` as an absolute path, which `EvalSymlinks` takes as is. Correct.
   - The output shape (`StrayEntry`, its JSON keys and its argv) is unchanged, so no consumer is affected.
   - `internal/hardenshed/doc.go:63-65` is updated in the diff. Correct.
   - `internal/hardencli/doc.go:27`: "It ends only processes whose cwd, executable or command line names the root." This stays true, because the resolved root is the same directory. Correct, nothing forced.
   - `pattern/PATTERN-child-processes.md`, the `EndProcess` bullet: "bounded to processes under a `hardener` run's sandbox root that no record names." A link-resolved cwd under the root is still under the sandbox root, so the bound still holds. Correct.
   - Other comments that describe the matching are unchanged and still accurate: the `StrayProcess` constant (`reap.go:37`), the `rootProcess` type (`reap.go:88`), `Reap`'s doc (`reap.go:102`) and the error text (`reap.go:191`).
   - Tests: `TestReap/EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget` is new, and the hardencli clear subtest is changed.
2. **The hardencli clear test's orphan is now started with `Dir = root` and no root argument** (`internal/hardencli/cli_integration_test.go:127-133`).
   - The subtest name at line 125 ("clear ends a process naming the root") still holds.
   - The assertions on the `ended` report and on the process exiting are unchanged.
3. **The `doc.go` sentence on how a process names the root** (`internal/hardenshed/doc.go:64-65`).
   - Its only reader is the package documentation. It is consistent with `namesRoot`'s doc (`reap.go:234`).
4. **Rubric checks:**
   - **Commit messages:** the `Root-Cause:` trailer on 19d52874 names the comparison of a link-resolved cwd against the unresolved root, and the diff removes that comparison. Correct. The range carries no `Pattern-Departure:` or `Loosened-Check:` trailers. The hardencli test change narrows what the orphan names rather than loosening any check.
   - **Declared PATTERN changes:** the directive declares none, and the diff touches no `PATTERN.md` or `pattern/` file. Nothing to check.
   - **Stencil wording:** no stencil is changed. Not applicable.
   - **Repository rules:** the test creates its link with `fslink.CreateDirLink` (PATTERN-lym-files) and names `.lym` through `lymdirs.DotLymDirName`. Covered.

**Findings:** none.

**Tool calls:**
- One grep over `*.go` and `*.md` for prose that describes root-naming or the sandbox clear behavior. My context had no search for docs or specs beyond `hardenshed/doc.go`.
