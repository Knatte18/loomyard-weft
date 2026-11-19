```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: checked whether the F1 and F2 fixes cover every site the findings' premises name.

**No findings.**

**F2 (routed-field comments).** I searched `internal/` and `contracts` for "routed to", `RoutedTo`, "routine Stuck" and "fourth field".
- Only the two haltnote comments (`internal/haltnote/haltnote.go:21` and `:50-51`) described the field as a Stuck hand-off, and both now name a routed Stuck or a reroute.
- `internal/haltnote/haltnote.go:56` ("a history row whose step handed the run on") is neutral, so it is not a stale member.
- The other sites already describe the field correctly:
  - `internal/shedengine/status.go:106` documents both the reroute and the Stuck-bounce case.
  - `internal/shedengine/activity.go` handles the reroute and bounce wording separately.
  - `internal/samplerengine/record.go:45` and `internal/loomcli/raddlewiring.go:58` are Stuck-specific by design.

**F1 (quoted-summary hint).** I searched for every use of `malformedQuotedSummary`, `quotingHint`, `unquotedColonError` and the fragment rule's wording.
- The regex is used only in `quotingHint`, which is called only at `internal/burlerengine/verdict.go:119`, so no other caller depends on the old multi-line match.
- `checkReviewKeys` (`verdict.go:192`) reports a YAML error without the hint, but it is only reached after `ParseReview` has already parsed the same header. It cannot fail on input the first parse accepted, so it is not an uncovered member.
- `contracts/stencils/burler/burler-review-format.md:50` states the fragment rule and does not describe the regex.
- Test rows now cover the three cases the finding names:
  - A fully quoted summary with a colon error in `fix` gets the colon hint.
  - A quoted fragment followed by prose containing a colon gets the fragment hint.
  - The existing bad yaml row asserts the colon hint text is absent.

Calls made (3 of 6):
1. `grep` for routed-to terms across `internal`, `docs` and `contracts`: my context lacked any survey of other sites describing the routed field.
2. `grep` for `malformedQuotedSummary`, `quotingHint`, `unquotedColonError` and the fragment wording: my context lacked the full set of regex consumers and docs.
3. `grep` of `internal/haltnote`, `internal/loomcli/halt.go`, `internal/reportengine` and `contracts` for the routed-field wording: my context lacked the haltnote consumers and the format doc.
