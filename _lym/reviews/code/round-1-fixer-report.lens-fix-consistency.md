```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F7 change set.

1. **LOW: `docs/shared-libs/lymcwd.md`, `Resolve` "On failure, one of two errors", with its twin in the `Resolve` godoc at `internal/lymcwd/lymcwd.go:84`.**
   The fix added an exported type, `AnchorGateError` (`internal/lymcwd/anchor.go`), as the concrete error behind the strict gate's `ErrCwdOutsideAnchor`.
   `driftcli.runCaptureChild` now depends on it, through `errors.As`, to take `Refusal` alone.
   Neither the module doc nor the `Resolve` godoc mentions the type, its `Refusal` and `AnchorDir` fields, or that a process whose cwd no shell chose reports `Refusal` alone.
   Both texts stay literally true, because the error still wraps `ErrCwdOutsideAnchor`, but they no longer describe the whole error contract of this surface.
   The fixer's write surface covers the module doc when the change calls for it.

I found nothing else that disagrees:
- `driftcli/doc.go`'s child-refusal paragraph still holds.
- `summary.md`'s Tests and "What changes" lines match the code.
- The `levelOnly` row agrees with the `store_test.go` comment.
- No markdown outside the review quotes the refusal text.

Tool calls:
1. `sed`/`git grep` over `internal/driftcli/doc.go`, the markdown files and `lymcwd.go`: docs describing the gate error were not in context.
2. `sed` of `docs/shared-libs/lymcwd.md` lines 36–95: the module doc's `Resolve` error list was not in context.
