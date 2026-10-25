```yaml
lens: fix-consistency
outcome: reported
```

No findings from the fix-consistency lens on F1, but one on F2 left a stale twin.

- **LOW — `internal/websterengine/doc.go:63`:** the package doc still says `ParseReport` "refuses an entry missing any of the three", meaning `old`, `new` and `reason`. After F2 the code refuses only a missing `old` or `new`, or the same glyph twice. The doc should say `reason` is requested but not enforced. `report.go` is the only place that comment disagrees with the code.
- **F1 (rewritten_refs assertion):** the fixture change is consistent. The only other reader of the old "Stands apart" intent text was the prose-mentions assertion, which the fix updated. The other callers of `newCardVerbFixture` don't depend on the intent text.
- **F2 stencils:** `webster-body-implementer.md:47` and `webster-template-master.md:112` still ask for a reason, as the review wanted.
- **F2 tests:** the test rows at `report_test.go:114-117` don't depend on a missing reason.

Calls made:
1. `grep -rn` across `*.go`, `*.md` and `*.yaml` for "reason" and "renames" wording. I needed it because my context didn't show the other places that describe the reason requirement.
