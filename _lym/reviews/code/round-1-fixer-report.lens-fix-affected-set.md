```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F3/F4 wording set:** a grep for "board write refuses" and "rules a board write" finds no remaining site. The only slug-rule statements outside the edited ones are `internal/raddleparser/doc.go:15` ("a slug the board would refuse fails before approval"), `internal/boardcli/doc.go:48` and `internal/boardengine/doc.go:264`. None claims that every board write applies the pair rule, and the `boardcli` and `boardengine` ones agree with the fix.
- **F4 spec sites:** both of the spec's slug statements (`raddle-proposal-spec.md:65` and `:110`) now agree.
- **F1 marker set:** the only other test in the `raddleparser` test files that sits right under a `testtiming:keep` marker is the fixed one, so no sibling of it lacks a marker for the same reason. I checked the marker sibling by grep count, not by reading each test's contained-or-not status. The redundancy report that lists the contained tests was out of reach.
- **F2 junction-name set:** the single closure at `raddlecontext.go:85` is the only `ValidateSlug` wiring in the context builder. The one added row, `docs`, pins its junction argument.

Unchecked: whether the redundancy report lists any other contained test lacking a marker. The report was outside what I could read.

Calls made (3 of 6):
1. Grepped Go and markdown files for the "board write refuses" wording, because my context held no repo-wide search of it.
2. Grepped non-test files for `slug-invalid` and slug-rule statements, to rerun the F3/F4 sibling search.
3. Read `internal/raddleparser/doc.go:10-20`, counted `testtiming:keep` markers in the `raddleparser` tests, and ran `git diff --stat`, because my context lacked all three.
