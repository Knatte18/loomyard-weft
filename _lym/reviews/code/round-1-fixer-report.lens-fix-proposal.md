```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F4, F5, F6, F7, F8, F9, F10).

**F6, LOW. `internal/fabricengine/clone.go`, `RefuseHubWithoutBolt`: `oneRepository = nsErr != nil || ns != (gitrepo.RefNamespace{})`.**
- Any `originNamespaceAt` error now reads as a one-repository hub, so the refusal names the join and seed commands, not the two-URL clone.
- `originNamespaceAt` returns errors that are not records-layout refusals: the `.git` Lstat failure at `recordslayout.go:70`, the config read failure at `:76`, and the claim read failure at `:91`. A two-repository hub whose config cannot be read would be told to run `lym fabric clone <code-url> --reset` or a `--single-repo` seed. Neither suits it.
- The reviewer's proposal said "spell the clone in the one-repository form", which assumed every layout error comes from a records store. Only an error matching `ErrRecordsLayout` does.
- Narrower alternative: `errors.Is(nsErr, ErrRecordsLayout)` decides one-repository. Any other error is carried on the two-repository form.

**F4, LOW. `internal/fabriccli/fabric.go`: `plain_archive_tags` is now absent when the listing fails.**
- The key used to be present on every `--remote` run. A failed listing now sets `plain_archive_tags_error` and drops `plain_archive_tags`.
- The cleanup help and `doc.go` say "lists in plain_archive_tags". A consumer that reads the key as always present gets a missing key where it used to get a list.
- The reviewer proposed this shape, so it is a trade-off: an empty list would read as "none found" when the listing never ran, which is worse. Nothing documents that the key is absent on error.

**F2, no finding.**
- `sentinelError.Is` matches the sentinel, so `errors.Is(err, fabricengine.ErrRecordsLayout)` holds for the unwrapped `FetchPairTips` error. This holds for both the missing-origin and missing-refspec messages.
- Returning the layout error as is lands in `catchUpWaitReason` (`internal/dobbyengine/catchupwait.go:122-131`) as `CatchUpWaitFailed`, not `CatchUpWaitOrigin`.
- That is the intended reclassification. A layout error no longer sets the "origin is unreachable" wait reason and is not retried as one.

**F5, F10, F7, F1, F8, F9, no finding.**
- F10: `--force-bootstrap` in `cloneResetCommand` is harmless on a one-URL form, where the flag is ignored. The CLI accepts it.
- F5: the seed command with `--single-repo --shortname <shortname>` is accepted by `refuseOneURLClone` for an origin with records branches and no board.
- F7: the two-repository row's expectations match `cloneResetCommand` output.

Further calls:
1. `grep` for `FetchRun` and `ErrOriginUnreachable` in `internal/dobbyengine/catchup*.go`: to see where a `FetchRun` error is classified. My context did not hold dobbyengine.
2. `Read` of `internal/dobbyengine/catchupwait.go:105-150`: to confirm `catchUpWaitReason` sends a non-unreachable error to `CatchUpWaitFailed`. My context did not hold that file.
