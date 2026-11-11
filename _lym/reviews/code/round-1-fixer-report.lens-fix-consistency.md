```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1 and F2 test additions. I found two stale file headers, both LOW.

**LOW: `internal/lymcwd/lymcwd_test.go` header (lines 1–5).**
- The header lists what ResolveWithin coverage includes: "ResolveWithin's anchor-subtree and records-sibling acceptance and the stale-marker refusal."
- The fix added `TestResolveWithin_NoOwnerOrNoMarker`, which covers a suffixed checkout with no owner beside it and a checkout with no recorded marker. The header does not mention either.
- The file-header rule says header plus filename should let a reader judge what the file covers. The header should name the new fallbacks.

**LOW: `internal/shuttlecli/cli_test.go` header (lines 1–5).**
- The header says the file "covers the shuttlecli cobra seam through RunCLI: run's flag-shape validation and the positional-argument validation of the remaining verbs."
- The fix added `TestRunCLI_AwaitForksResolvesFromTheRecordsSibling`, which covers per-verb resolver dispatch. That is neither flag shape nor positional arguments, so the header no longer matches the file.

I checked these and found nothing:
- **Error text:** `lymcwd.ErrCwdOutsideAnchor.Error()` is a prefix of the wrapped strict-gate error, so the `state` row's `Contains` holds. The `no shuttle strand` string matches `awaitforks.go`.
- **Imports and helpers:** the new imports (`gitkit`, `weftname`) are used, and the helper pattern matches the sibling `TestResolveWithin`.
- **Naming and process-global state:** the new test names follow the neighbouring convention. The non-parallel note matches the `t.Setenv` use.
- **Source comments:** no source comment changed, and the `cli.go` header and the `await-forks` Long text already agree with the tested behaviour.

Tool calls made: none. My context already held every changed file.
