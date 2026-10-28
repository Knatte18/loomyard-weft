```yaml
lens: forced-edits
outcome: reported
```

The plan is mostly sound under this lens, but five edits its changes force are listed on no card. One makes card 5 under-scoped as written.

1. **MEDIUM — Card 5 (D5): `pattern/PATTERN-test-speed.md` is listed under Extends but needs Edit.**
   `pattern/PATTERN-test-speed.md:83` says "`//lym:guard` counts as directly above when only `//testtiming:keep` lines separate it from the func line".
   Card 5 widens `internal/impactset#onlyKeepLinesAfter` to admit `//lym:seam` lines too, so that sentence becomes false.
   Adding a new section does not fix it; the line itself has to be rewritten.
   Fix: move `pattern/PATTERN-test-speed.md` from Extends to Edit, and have the Intent rewrite line 83.

2. **MEDIUM — Card 5 (D5): `internal/impactset/doc.go` and `internal/impactset#GuardScanError` are left describing the old rule.**
   - `internal/impactset/doc.go:41` says only a `//testtiming:keep` line between the marker and the func line leaves the marker in place. Seam lines now do too.
   - The same doc and the field doc on `GuardScanError.Kind` ("which of the three failures this is") enumerate three kinds. Card 5 adds a fourth, `SeamMalformed`.
   - Card 5 lists the doc only under Extends and does not list `GuardScanError` at all.
   - Fix: list `internal/impactset/doc.go` under Edit, and add `internal/impactset#GuardScanError` under Edit so its type and field docs name the seam failures.

3. **LOW — Card 4 (D4): two `shedrecipe.Env` field docs become wrong and no card lists them.**
   - Card 4 makes the loom "discussion" gate require `env.WorktreeRoot` and `env.PlanIndex`.
   - `internal/shedrecipe/recipe.go:87` still says `WorktreeRoot` is "read by the gate resolver's "plan" and "verify" gates".
   - `internal/shedrecipe/recipe.go:277` still says `PlanIndex` is "the code index the "plan" gate resolves plan refs against".
   - Card 7 edits `internal/shedrecipe#Env` later, but only for `TierTags`.
   - Fix: add `internal/shedrecipe#Env` to card 4's Edit, and have the Intent update both field docs to include the loom "discussion" gate.

4. **LOW — Card 1 (D1, D2): the file header of `internal/discussionparser/validate.go` becomes false.**
   - Lines 1–2 say "The loom gate's other check is CheckRelevant, and a darn run runs Validate alone".
   - Card 1 makes both callers also run `CheckDecisionIDs`.
   - The header sits outside every member span, so only the file target covers it.
   - Fix: give card 1 a target or Intent line that rewrites that header. If a file target beside a member is refused, the cheapest option is to name the header change in the Intent of a card that already edits that file.

5. **LOW — Card 4 (D4): no card states that `internal/loomshed/doc.go`'s no-tree-sitter claim still holds.**
   - `internal/loomshed/doc.go:13` says "Its plan gates resolve plan refs through a told planindex.Index, so the package links no tree-sitter grammar."
   - Card 4 adds a direct import of `github.com/Knatte18/quarry/glyph` and resolves discussion citations through the index as well.
   - The claim stays true only because `internal/planparser` already imports that package, and no card's Intent says so.
   - Fix: have card 4's Extends of `internal/loomshed/doc.go` widen that sentence to cover the discussion gate's citation check.

Checked and clean:
- Callers of the re-signed `ValidateLoomDiscussion`, `NewDiscussionGate` and `planContextFindings`.
- Test fakes that embed `planindex.Index`.
- The `allowedStatusConsumers` registration.
- The `loomshedAllowedImports` additions.
- Package clauses of the Create handles, including `internal/fabricengine_test` and `internal/websterengine_test`.
- Help-text pins: none exist.
- Whether `wireLightweight` fills `PlanIndex`: it does, at `internal/loomcli/wiring.go:309`.

Tool calls I made:
- `internal/loomcli/wiring.go` `wireLightweight`, plus the gate tests and docs that mention the discussion gate: to confirm the verb's `WorktreeRoot` and `PlanIndex` source and find tests that resolve the "discussion" gate.
- `wireLightweight` body and the `internal/shedrecipe/entries_gate_test.go` / `internal/loomrecipe/gatequiescence_test.go` Envs: to check whether the new loom "discussion" requirements break existing resolve tests.
- Help-text pins, `internal/testkit/enforcement_test.go`, and commentlint's directive pattern: to check for forced edits from the help changes, the envkit test import and the new marker lines.
- `internal/loomcli/renumber_test.go` and the `GuardScanKind` consumers: to rule out help pins and switches over `Kind` outside impactset.
- `internal/loomshed/doc.go` head, `internal/loomcli/doc.go`, `internal/impactset/doc.go:41` and `pattern/PATTERN-test-speed.md:83`: to find doc text the changes make false.
