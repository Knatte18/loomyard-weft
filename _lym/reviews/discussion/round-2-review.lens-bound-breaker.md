```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guarantee and existing-code claim in the decision record. Three claims break: one BLOCKING, two MEDIUM. One more holds only in part (LOW), and one gap is left unstated (LOW).

**Claims attacked**

- **D1, coverage never adds a package `diff` would not run:** held. `withGateScope` returns early on explicit targets (`scope.go:108`).
- **D1 bound, `finalize` (scope `all`) runs every test before a landing:** held. The hub's `finalize` is `scope: all`. A tree-scoped darn Finalize uses explicit targets, but those runs are never coverage-narrowed either.
- **D2, a map's recorded hashes are the content at its `Commit`:** held only in part. See finding 4.
- **D2 with the Constraints line, the marker's `Commit`/`BuiltAt` tell apart a map an older binary rewrote:** held. `writeMap` always stamps `BuiltAt = time.Now()` (`testmappackage.go:286`), so a rewrite never keeps a matching marker.
- **D3 bound, the spawn scan's limits (tier0 empty verdict, no cross-package follow, git hooks):** held as stated. It matches `spawnscan.go:168-174`.
- **D4, function-level selection is a superset of block-level selection:** held for body edits. An empty function body still gets a cover counter.
- **D4, functions changed vs. non-body changes:** broken. See finding 2.
- **D4, a change on main since the map was built can only add tests:** held for body changes. Non-body changes are the stated Open risk.
- **D5, a new plain function is reached only through a changed caller:** held. Var-initializer references are non-body misses, and a test calling it changes the importing package.
- **D5, comments are no miss:** broken. See finding 3.
- **D5, the non-Go file rule covers state the map cannot speak for:** broken. See finding 1.
- **D5, an added standard-library or covered import is no miss:** held. The bound names the stdlib `init` case.
- **D6, the selected entry carrying `-run` never serves as a base or a whole run:** held. `newestWholeAncestor` needs empty args (`green.go:206`), and `argsMatch` needs exact args for a non-empty entry (`green.go:224-229`).
  - Also held: an empty-args package entry on the same tree carries a selected run. `isGuardRunFilter` accepts the anchored `-run ^(...)$` shape (`green.go:232-234`), which is a superset and so safe.
- **D6, an over-budget pattern runs whole; `--stop-on-fail` and the rerun act on both invocations:** held.
- **D8, the scope only reads maps, through `ReadMap` under its lock:** held (`coveragemap.go:61-64`).
- **Constraint, the measured cost split:** held. (191+516)/975 is 72.5%.
- **Constraint, the strict loader refuses `coverage` on an older binary:** held as stated.

**Findings**

1. **BLOCKING, design.** `decision-record.md` D5, last bullet (lines 124-125), with D2 (lines 50-51).
   - **Summary:** a change to a non-Go file a covered package embeds from a subdirectory is neither a changed function nor a miss. The importing packages' tests run nothing.
   - **Counterexample:**
     1. A diff edits `contracts/stencils/loom/loom-template-plan.md`. `contracts/stencils/loom/` holds no Go file, and the hub sets no `inert_paths`.
     2. impactset maps the path to `contracts/stencils`, the nearest package directory up the tree, so the package counts as changed. Its importers enter the `coverage` set.
     3. D4 finds no Go file changed, so no function changed.
     4. D5's non-Go rule checks only "a non-Go file of a covered package's directory". D2 records "every file its directory tracks", and testmap lists a directory's files through `CodeDirAt`, which returns only direct files.
     5. The subdirectory file is never recorded, so no miss fires. An importer's tests that render the embedded stencil are not selected, which silently narrows the run.
   - **Evidence:**
     - `contracts/stencils/stencils.go:81` and the other `//go:embed &lt;subdir&gt;/…` lines
     - `internal/impactset/derive.go:99-117`
     - `internal/fabricengine/codereads.go:12-15` (direct files only)
     - `internal/gatecli/testmap.go:426-445`
     - `_bolt/_lym/config/gate.yaml` (no `inert_paths`)
   - **Fix:** the D2 record and D5's non-Go rule should cover every non-Go file the package owns under impactset's path-to-package rule: its subdirectories that hold no package, `go:embed` targets included.

2. **MEDIUM, design.** `decision-record.md` D4 (line 96) against D5 (line 118).
   - **Summary:** a function's signature or receiver is in neither D4's "body text" nor, on the natural reading, D5's "outside function bodies". A signature-only change neither selects nor misses.
   - **Counterexample:**
     1. In covered package A, `func (c Config) Apply()` becomes `func (c *Config) Apply()` with an identical body. Mutations now show to callers.
     2. D4 compares body text only, so the method is not changed.
     3. Block keys take their enclosing function's span from the whole `FuncDecl`, signature included (`blockkey.go:152-156`). A plan that reuses that span as "function" for D5 counts the signature as inside a function, so D5 raises no miss.
     4. The importing package's tests that called `Apply` are not selected.
   - **Fix:** D4 should compare the whole declaration, receiver, signature and body, or D5 should name the signature as a non-body change.
   - **Evidence:** `internal/testcover/blockkey.go:150-156`, `decision-record.md:96`, `decision-record.md:118`.

3. **MEDIUM, design.** `decision-record.md` D5 (lines 118-119), "other than comments".
   - **Summary:** compiler directives are comments, so a changed `//go:embed`, `//go:build` or `//go:linkname` line counts as no change.
   - **Counterexample:** a diff in covered package A retargets `//go:embed loom/a.md` to `loom/b.md` on an unchanged var. Or it edits a file's `//go:build` line so the file joins or leaves the build. Either way:
     - The text outside comments is identical, so D5 raises no miss.
     - No function body changed, so D4 selects nothing.
     - The importing package's tests that read the embedded value, or reach the swapped-in file, do not run.
   - **Fix:** D5 should count a changed `//go:` directive line as a non-body change.
   - **Evidence:**
     - `contracts/stencils/stencils.go:16-125` (embed directives on vars)
     - `decision-record.md:118-119`

4. **LOW, design.** `decision-record.md` D2 (lines 57-60), "A map's recorded hashes are therefore the content at its `Commit`."
   - **Summary:** this holds for the hashes but not for the coverage they are meant to vouch for.
   - **Counterexample:**
     1. The dirty check reads `Status` once, before the walk (`testmap.go:270-277`).
     2. The changed-during-run check compares HEAD with HEAD through `CodeFileAt` (`testmappackage.go:204-226`).
     3. Blocks are keyed by reading the working tree after the tests ran (`KeyBlocks` reads `pkgDir`, `blockkey.go:94`).
     4. So an uncommitted edit to a covered package in the prime's working tree, made after `prepare` and reverted before the next landing, is built and keyed into the map. The map still records `Commit`'s hashes.
     5. If the edit dropped a call from F to G, test T is not recorded as covering G. A later change to G does not select T.
   - The record extends this window from one directory to every covered package's directory.
   - **Evidence:** `internal/gatecli/testmap.go:270-277`, `internal/gatecli/testmappackage.go:204-226`, `internal/testcover/blockkey.go:94`.

5. **LOW, consistency.** `decision-record.md` D4 (line 95) and the D5 miss list.
   - **Summary:** neither says what happens when a changed covered file, in the working tree or at the map's `Commit`, does not parse with `go/parser`.
   - Under widen-never-narrow it should be a D5 miss, with a reason in the scope report. As written, the plan writer could make it a scope error that refuses the call, or skip the file and select nothing.
   - **Evidence:** `decision-record.md:95`, `decision-record.md:111-125`.

**Tool calls**
- `grep` for `//go:embed` lines that reach into subdirectories: the manifest held no embed site, and finding 1 needed one.
- `grep` of the hub `gate.yaml` for `inert_paths`, a Go file under `contracts/stencils/loom/`, and the count of importers of `contracts/stencils`: the manifest showed only the hub's gates section, and the counterexample needed the rest. No `inert_paths` and no Go file under `loom/`. Five non-test packages import `contracts/stencils` directly.
