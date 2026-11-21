```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F2, F5 and F9 change set for twins and siblings left stale (lens fix-consistency).

**Findings**

LOW — `internal/burlerengine/doc.go:602` and `internal/burlerengine/roundcheck.go:67` still say "A file that an entry of RoundContext.AmendingTexts names is exempt from the changed-file rule."
- The F5 change (`roundcheck.go`, `namedByAmendment`) now excludes the plan overview even when an entry names it.
- Neither sentence says the exemption covers plan card files only.
- `namedByAmendment`'s own doc comment does say "plan card file", so the code comment is right; the two summary sentences are the broader, stale wording.
- Evidence: `roundcheck.go:330` reads "path is a plan card file named by…", while `doc.go:602` and `roundcheck.go:67` say "a file".

No stale twin found for the other two findings.
- F2: `grep` finds no other "re-run the same webster verb" text and no other reference to the removed "is not an added-decision heading" error from `DroppedCreates`.
- The identical error at `append.go:196` sits in a different function that still reaches it, so it is correct as it stands.
- F2: the `DroppedCreates` entry in `discussionparser/doc.go` never mentioned the heading error.
- F5: `IsPlanOverviewFile` is the only new predicate, and `planparser` still uses `overviewFileName` in its other sites, so no second literal was added.

**Calls made**
1. `grep` over `internal`, `_lym/specs` and `docs` for the exemption wording and the old way-forward phrase, to find twin sentences my context lacked.
