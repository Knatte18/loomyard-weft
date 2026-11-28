```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out as proposed (the fix-proposal lens).

1. **MEDIUM.** Changed text: `_lym/landing/summary.md`, "Design decisions" bullet (F12).
   - **Problem:** the proposed sentence, "This change covers Record-Seed's engine and contract steps only; …", is false about the change as a whole.
   - **Evidence:** the same document's "What changes" lists the new jacquard recipe, tapestry rewritten as two blocks, the `shedbuild.Expand` rule, the vocabulary and the wiring. A PR reader would take the sentence as the scope of the diff. It needs to be limited to Record-Seed, along the lines of "For Record-Seed, this change covers the engine and contract steps; …".

2. **MEDIUM.** Changed text: `loomshed.SeedParentParam` and its users (F1).
   - **Problem:** the proposal named only the `wiring.go` and `sharedbootstrap.go` sites, so a second writer of the same seed param still uses the bare literal.
   - **Evidence:** `internal/battencli/wire.go:639` has `return map[string]string{"parent": origin.ParentBranch}, nil` in `childSeedParams`. That function's comment says the child's seed must agree with the params the child's bootstrap writes, which are now keyed by `loomshed.SeedParentParam`. `wire.go:45` already imports `internal/loomshed`, so using the constant adds no import edge.

3. **LOW.** Changed text: `internal/loomshed/doc.go` package comment (F7).
   - **Problem:** the replacement sentences leave the next line's "it" without a clear referent.
   - **Evidence:** the comment now reads "Loom's producer lists live in the recipe files under contracts/recipes. / Tapestry's is composed from the motif and jacquard recipes through block rows. / internal/loomrecipe assembles a *shedengine.Shed from it." "It" used to mean the single list in `tapestry-recipe.yaml`. It now follows a plural "lists" and a sentence about tapestry's composition, while `loomrecipe` builds more than tapestry.

4. **LOW.** Changed text: `keptRecordSupportLog` in `internal/loomshed/recordwrite.go`, and its test literal (F9).
   - **Problem:** the proposed interview line says "when the record was seeded", but this path runs only when the record already existed and the row seeded nothing.
   - **Evidence:** the log is chosen only when `logMissing && !storedFound && p.noStoredReason != nil`. A missing record in that state returns Stuck earlier, so here the record was never seeded by this call. The same line goes on to say "the record was kept as it stood".

I found no breaking case for the F2, F3, F4, F6, F8 and F10 proposals.

Tool calls:
- Read `internal/battencli/wire.go:620-642` and grepped `battencli` for the `loomshed` import; my context had only a one-line grep hit for line 639 and nothing about that package's imports.
