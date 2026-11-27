```yaml
lens: code-affected-set
outcome: reported
```

**code-affected-set lens: what the diff forces outside the lines it touches**

**Per change, the set I checked**

| Change | Set | Result |
|---|---|---|
| `Renderer.Passages` and `PassageReader` signatures, `Standing` | `cmd/lym` install, both `Prepare`s, `Show`, existing tests, package docs | covered |
| `Target.Engine` on the file rows | `promptTargetsFor`, `passagesOf`, `FileEntries`, `overview.go` `promptReach` (called for prompt targets only, so `HEDDLE.md` is unchanged) | covered; literals are finding 2 |
| New `localInstructionFileName` | the other spellings of `CLAUDE.local.md` | not covered (finding 1) |
| Codex passage item name `heddle passage <slug>` | refusal test; search for numbered names in docs and tests | covered |
| `manifests/` under the run-directory root | orphan sweep and strand lookup (skip it by name); `agentwait`, `waitmark`, `carryover`, `attach`, `stepstop`, `sessionread`, `enginesrun` and both `activity` scans (a directory with no `run.json` yields nothing); the refusal test that counts run-root entries | covered. I did not open the bodies of `agentwait`, `waitmark` or the first `activity` scan beyond their loop heads |
| Manifest copy in `Runner.launch` | engines that write no record, test fakes | covered |
| `UserSetup`, `InstructionItem`, `OmittedHook` fields; `skillListing` third result | the single caller, tests, Codex warning path | covered |
| Constructor and `build` signatures, `LaunchSeams`, `NewWithSeams` | all call sites, tests, `doc.go`, provider-seam background | covered |
| New verb `lym shuttle manifest` | `docs/overview.md` verb list and package tree, `cli.go` header and `Command` doc, positional-argument test; search found no other verb list | covered |
| "Writes nothing outside the temp directory but the Codex home" | Claude's creel flag-directory path (computes a path, writes nothing); the table-builder files I searched show no `MkdirAll` or `WriteFile` | covered |
| `testOnlyCodeAllowed` and `treeSitterAllowedLinks` edits | commit trailers `Pattern-Departure` and `Loosened-Check` | covered: each trailer names a real entry or test and a reason |
| Declared PATTERN change | `PATTERN.md` line (matches the declared text exactly), new background file | landed |
| Stencil wording | the diff changes no stencil | n/a |

**Findings**

1. **BLOCKING, class `consistency`**: `internal/shuttleengine/claudeengine/manifest.go:23`
   - The diff declares `localInstructionFileName = "CLAUDE.local.md"`, but the same package still spells the literal at `internal/shuttleengine/claudeengine/capabilities.go:35` (`return []string{"CLAUDE.md", "CLAUDE.local.md"}`).
   - `internal/heddleengine/target.go:68` also spells it, as the file heddle renders.
   - This breaks `PATTERN-code-conventions`: "one named constant, declared once and referenced everywhere, never a repeated literal".
   - The decision record's Code-Review carry-over lists `claude-local-instruction-name-redeclared`, and the tree still shows the redeclaration.

2. **BLOCKING, class `consistency`**: `internal/heddleengine/target.go:68-69`
   - The diff adds `Engine: "claude"` and `Engine: "codex"` literals to the two file rows. They repeat the literals already on the `claude-prompt` and `codex-prompt` rows (`target.go`, same table), with no named constant.
   - `PATTERN-code-conventions` forbids a repeated literal, and its background says a change touching such a literal never adds another. No `Pattern-Departure` trailer covers heddle.
   - The mitigation to weigh is that these are values in a table keyed by name, which the "No hard-coded choice" rule allows. The repeated-literal rule is the one at issue.
   - The carry-over lists `heddle-file-target-engine-literals`.

**Tool calls (4 of 6)**
1. Read the other run-root scan loops and searched for outside readers of the run root; the context held only their `ReadDir` heads.
2. Searched verb lists, numbered passage names, instruction-file name literals and `Target.Engine` readers; the context had no search for these.
3. Read `creelFlagDirForHook` and `CreelFlagDir` and searched the table builders for writes; the context did not show them.
4. Read heddle `overview.go`'s use of `Target.Engine` and `claudeengine`'s `capabilities.go`; both turned up in call 2 and were not in context.
