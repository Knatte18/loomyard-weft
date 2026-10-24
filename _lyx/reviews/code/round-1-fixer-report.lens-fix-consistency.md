```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix's change set for sibling and cross-reference disagreements (lens fix-consistency).

**Findings**

1. **MEDIUM** — `internal/fabricengine/doc.go:596-597`. The package doc says an adopted weft worktree that already carries its origin record "keeps it, neither rewritten nor committed". The F2 change in `add.go` now rewrites and commits that record when `opts.Beam` is set and the record is unmarked. The new test `TestAdd_AdoptedWeftTakesTheBeamMarkOnly` asserts exactly that, so the doc contradicts both code and test. The comment in `add.go:505` was updated; this twin was not.

2. **MEDIUM** — the guard's description is stale in four places after F1 moved it to the nearest live orch. Each still says the caller is compared to the "recorded parent":
   - `internal/loomcli/decision.go:8` and `:10`, the file header
   - `internal/loomcli/decision.go:882`, `:905`, the help text
   - `docs/overview.md:529`
   
   The `decisionDeps.parent` field comment was updated, but these were not. With the beam orch stopped, the prime's orch now passes, which these texts say is refused.

3. **LOW** — the `Add` doc comment (`internal/fabricengine/add.go:186-190`) covers origin-record behavior but omits the new rule that an adopted record gains the beam mark. `internal/beamcli/doc.go:19` also says "an existing pair gets the mark through `fabricengine.MarkBeam`", which is still true. Only the `Add` and `doc.go` text needs the addition.

4. **LOW** — `internal/loomcli/decision.go:564`. The refusal text "the run's parent could not be resolved" and the surrounding "parent's to record; ask %s to run the verb" (`:571`) are now accurate only if `parent.Name` is the nearest live orch. The text is still readable, but `ask %s` now names a different orch than the recorded parent the prompts show.

**Checked, no finding**
- F3: no `umbrella` remains in `internal/boardengine/*_test.go`.
- F4: no other "orch's reed session" text remains in `docs` or `internal/battencli`.
- The `hubgeom/doc.go` addition agrees with the `NearestLiveParent` doc comment.

**Tool calls**
1. A grep for "recorded parent", "run's parent" and `ResolveParent` across `decision.go`, `loomcli/doc.go`, `docs/*.md` and `PATTERN.md`; the context held none of the stale-text sites.
2. A read of `add.go:186-200`, to check the `Add` doc comment; the context only showed lines from 440 on.
3. A grep for "orch's reed session" in `docs` and `internal/battencli`, to look for twins of the F4 sentence.
4. A grep for `umbrella` in the boardengine tests and for "neither rewritten" wording; this found the `fabricengine/doc.go:596` twin.
5. A read of `fabricengine/doc.go:594-599`, `decision.go:6-12`, `decision.go:880-907` and `docs/overview.md:527-530`, to confirm the stale guard and record text.
