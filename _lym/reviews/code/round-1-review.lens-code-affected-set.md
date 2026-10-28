```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I found no findings.

**Per change, the set found and whether it is covered:**

1. **`DarnOutcome.Reason` replaced by `FindingsPath`.**
   - Set: the only users are `internal/loomshed/darnwrite.go`, `darnwrite_test.go`, `internal/loomcli/darnwiring.go` and `wiring_darn_test.go`.
   - Two other constructors build `DarnOutcome{}` with no field: `internal/shedrecipe/entries_darnwrite_test.go:34` and `internal/testkit/envkit/envkit.go:384`. Neither is affected.
   - No other reader of `Reason` exists. Covered.
2. **`latestDarnOutcome` gains a `findingsPath` parameter.**
   - Set: the single production caller at `darnwiring.go:156` and `TestLatestDarnOutcome` (`wiring_darn_test.go:408-480`).
   - The test passes `findingsPath` and has rows for kept and not-kept findings. Covered.
3. **`renderPriorWork` output changed (the `halted with:` line is gone, and the outcome, findings path and decisions line are new).**
   - Set: the `darnwrite_test.go` rows and the `doc.go` text, both updated.
   - The stencil `contracts/stencils/darn/darn-template-write.md:35` says to continue from the commits and "fix what the prior work names". That is consistent with the new note.
   - `internal/darnengine` (`spec.go`, `spec_test.go`) only renders `(none)` for an empty token, so it is unaffected. Covered.
4. **Darn-Write gate entries get a `Reprompt` line.**
   - Set: `darnWriteEntry` and its doc comment, and the `HappyPath` test rows.
   - `shuttleengine/gate.go` already supports `Reprompt` and requires a single-line result, which the new line satisfies.
   - A search for the default text "fix every finding" and "Gate findings recorded at" found no Darn-Write-specific assertion or doc that the new line contradicts. The other hits are other roles' stencils, webster, burler and `crucible`.
   - `env.DecisionRecordPath` is set in `wire` (`wiring.go:571`), so the darn row gets it. Covered.
5. **PATTERN and rule documents.**
   - A search of `PATTERN.md`, `pattern/`, `docs/overview.md`, `CLAUDE.md` and `contracts/recipes` for text on Darn-Write halt reasons, re-prompts or prior work found none.
   - The commit declares no PATTERN changes and no `Pattern-Departure:` trailer, so none is owed. Covered.
6. **Stencil wording.**
   - The diff changes no stencil. Covered.
7. **`doc.go` and the `entries_darnwrite.go` doc comment.**
   - Both were updated, and no other doc describes the removed halt-reason behavior. Covered.

**Tool calls:**
1. Grep for other users of `DarnOutcome` and `latestDarnOutcome`, for the default and new re-prompt text, for PATTERN, docs and recipe mentions of Darn-Write halt reason, re-prompt or prior work, and a read of the end of `wiring_darn_test.go`. My context lacked the full call-site set.
2. Grep of `darn-template-write.md` for "fix every finding" and "Added after Discussion". My context lacked whether the stencil contradicts the new re-prompt line.
