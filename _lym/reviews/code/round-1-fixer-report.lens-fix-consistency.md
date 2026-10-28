```yaml
lens: fix-consistency
outcome: reported
```

No findings. Every twin of the narrowed card-id sentence now says the same thing. These are `fabricref.go:32`, `fabricref.go:75`, `doc.go:146` and `loom-plan-spec.md:535`. The grep for the old wording found no stale copy elsewhere. The `_lym/landing/summary.md` edit matches too. `ReferenceRule`'s doc comment at `refscanner.go:45` and `MatchPath`'s at `refscanner.go:72` already describe gate blanking, so nothing was left behind there. I did not re-read the test rows.

Tool calls:
1. One `grep -rniE` over `*.go` and `*.md` for the old and new gate-exemption wording, to find unchanged twins outside the files in my context.
