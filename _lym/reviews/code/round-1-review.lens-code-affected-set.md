```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens for this round.

**Per change, the set found:**

1. **`Board.PromoteTo` signature (now returns siblings, takes `dependsOn`).**
   - The set is the `Promote` wrapper, the CLI call at `cli.go:209`, and the tests in `sketch_test.go`, `slugrule_test.go` and `raddlefile_test.go`.
   - All callers are updated. Covered.
   - `Store.promoteTo` has one more caller, `Store.Promote`, and it is updated. Covered.

2. **`promote` payload gains `depends_on`, and the `Use` token changes.**
   - The set is `help_test.go` and `payload_test.go`; both are updated. Covered.
   - It also includes `plugins/ly/skills/board/SKILL.md:113-121`, the agent-facing guide to promote. It still shows only `lym board promote '{"slug":"my-note"}'` and says nothing of `depends_on` or the race with a later `set-deps`. Not covered (finding F1).
   - `docs/overview.md:463` only mentions a plain promote, so it needs no change.

3. **Sibling `notice` added to the promote envelope.**
   - `boardcli/doc.go` and the `Long` text are updated. Covered.
   - Siblings are computed inside `writeReportingSiblings`. The no-write path (an entry already a task) returns empty siblings, which is consistent.

4. **`dependsOnList` extracted from `set-deps`.** Behaviour is unchanged: a nil value still gives an empty list, and the error strings are identical. Covered.

5. **Rules documents.** I found no `Pattern-Departure` trailer in either commit and no declared PATTERN changes in the directive. `PATTERN-sole-parsers` and the other entries I checked do not cover board payloads. Nothing is owed.

6. **Stencils.** The commit touches none, and I found no stencil that mentions `board promote` in the grep over `internal/`. Nothing is owed.

**Findings:**

- **F1, MEDIUM, scope.**
  - Location: `plugins/ly/skills/board/SKILL.md:113-121`.
  - Summary: the board skill's promote section does not mention the new optional `depends_on`, nor that a Sketch's dependencies belong in the promote call rather than a later `set-deps`. This is the exact ordering the issue's orch tripped on.
  - Evidence: lines 113-121 show only the slug-only example.
  - Fix: add a `depends_on` example and a one-line rule to that section. Unlike the `set-deps` ordering problem, this is the only agent-visible guidance on the fix.

- **F2, LOW, consistency.**
  - Location: `internal/boardcli/cli.go:167`. The `Use` token is `promote {slug|id|handle, depends_on?}`.
  - Summary: `payloadLegend` in `payload.go:12` says "a|b exactly one of the keys" and "? marks an optional key". The token mixes a `|` group and a `,` in one brace, so I could not confirm it follows the declared grammar. `TestPayloadUse_MatchesDeclaredKeys` in `payload_test.go:33` is the check, and it was updated. Treat as LOW and unverified unless that test passes.
  - Fix: compare the token with how `set-deps` spells its `Use` and match it.

**Tool calls:** none. My context already held every file I used, so I made no further reads.
