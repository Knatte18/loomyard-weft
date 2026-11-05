```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The two findings with a set premise are F1 and F7, and the fix covers every member of both sets.

- **F1** (sites that treat `draft` as the only unreleased status): the only guard that tests for it is the start guard. `startguard.go:99` now matches both `draft` and `discussed`. The other `StatusDraft` uses outside tests are `pairteardown/claim.go:108` and `battencli/cancelcmd.go:515`. Both write `draft` to settle a claim or cancel a run, so a `discussed` check does not belong there.
- **F7** (text that names `draft` alone as released by null): `boardcli/cli.go` lines 640, 646 and 879 are fixed, and `boardengine/doc.go:244` is fixed. Two sites still describe release by `draft` alone, and both are accurate as written:
  - `cli.go:882` says "A task the merge creates is draft", which is only about creation.
  - `doc.go:202` is the Draft layer, and `doc.go:204` covers Discussed separately.

  `doc.go:272` is the generic release sentence, and `doc.go:273` right below it already covers `discussed`.

Tool calls:
1. Grepped `StatusDraft`, `IsDraft` and `isDraft` in non-test Go code, to rerun F1's set search. My context held only the files the fixer read.
2. Grepped the release and draft-task wording across `internal`, `docs` and `README.md`, to rerun F7's set search.
3. Read `pairteardown/claim.go` and `battencli/cancelcmd.go` around the `StatusDraft` writes, to confirm they write the status and do not guard on it. Neither file was in my context.
