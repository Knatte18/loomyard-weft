```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set I found:**

- **`adoptDeps.StartSuite` (new field)**
  - Constructors: `hubDepsAs` (`deps.go`) and the test `world.deps()` (`adopt_test.go`). I found no other `adoptDeps{` literal in the repo. Covered.
  - The call site in `adopt()` calls `deps.StartSuite()` with no nil check, unlike `StartDobbies`. Every constructor sets it, so this is fine.
- **`outcome.Suite`, `outcome.SuiteWarning` and the `suite` / `suite_warning` envelope keys**
  - Writer: `cli.go` fields. Asserted: `adopt_test.go` rows. Documented: `cli.go` long help and the `doc.go` envelope paragraph. No other reader of `dobby_warning` or the envelope exists (search hits only `adoptcli`). Covered.
- **`lyxdirs.AdoptDirName`**
  - Only user: `deps.go`, through `fabricengine.HubScratchDir`, the same route `RunProcDirName` and `FailedPushDirName` take. No `lyxdirs` test or fixture enumerates the tokens. Covered.
- **The `lyx adopt` long help (new step 9, trimmed step 3)**
  - I found no golden or test that asserts the help text.
  - The removed rationale "this machine's dobby drives only runs whose parent is its own worktree" is still carried by `refusal.go:172` and `doc.go:30`. No loss.
  - Covered.
- **Ordering semantics: "StartDobbies runs last" becomes "runs after the resumes"**
  - Updated in the `adoptDeps` comment and in the `adopt.go` header. Not updated in `doc.go:61`. See F1.
- **`docs/overview.md` adopt row**
  - The new sentence was added. Covered.
- **`PATTERN.md` and `pattern/`**
  - The only mentions of adopt are unrelated. No declared change is expected. Covered.

**Findings**

- **F1 [LOW] [consistency] `internal/adoptcli/doc.go:61`**
  - The text says "Last, after the resumes, `StartDobbies` starts the dobby…". The diff makes the suite start the last step, and the new `# Step 6` section follows directly, so "Last" is now false.
  - Evidence: `doc.go:61` against `adopt.go:229` (`StartSuite` after `StartDobbies`).
  - Fix: change "Last, after the resumes," to "After the resumes,".

No stencil is touched by the diff, so I raised no stencil-wording finding.

**Tool calls (3 of 6):**
1. `grep -rln "adoptDeps{\|dobby_warning"`: to find other `adoptDeps` constructors and envelope readers outside my context.
2. `grep -rn "lyx adopt|adopt"` over the docs, `PATTERN.md` and `pattern/`, plus `AdoptDirName` users: to find other docs or consumers of the new token.
3. `grep` over `doc.go` headings and ordering phrases: to check the step numbering and the stale "Last".

(Before those, I ran one `grep` for "drives only" and for help-length-cap tests. That makes 4 of 6 in total.)
