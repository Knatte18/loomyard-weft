```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the searches behind the set-premise findings to see whether the fix covers every member.

Findings I left unchecked: F6 and F9. I did not rerun their searches. F6's new raw writes all sit in new code the plan creates, and F9's deletions are a set the review defined inside card 15's own text, so no search of the code tree would add members. F5's set (the callers of `taskAnchor` and `driverLockBusyError`) was already searched in the parent context: `archive.go:173`, `archive_test.go:83`, `busyError`, `teardown.go:312` and `cancel.go:74`. Card 4's new Edit list covers all of them.

F1 is fully covered. Every weft pull or replay in production code is now behind a stop: `pullWeftArm` at `pull.go:312`, `settleUnmovedWeft` at `pull.go:379` (reached only through `pullWeftArm`), `retryWeftPush` at `pushclassified.go:510`, `originMover.move` at `originreconcilemoves.go:219`, and `fastForwardWeft` at `parenttips.go:280`.

**LOW — F12's fix misses some stale doc sites.** The changed text is card 6's Edit list and its new sentence placing the marker at each worktree's weft root. It names only the `unwire.go` and `codebinding.go` file headers, the unwire help in `Command`, and `staleFabricAnchorName`. These sites still describe the anchor as recorded on `weft:main` (Bolt):
- `internal/fabriccli/clone.go:27`: `CloneAndWire`'s doc says "the weft:main anchor+config commit and push". Card 6 lists `CloneAndWire` in its Edit list, but only for the commit subject.
- `internal/fabriccli/doc.go:12`: "committed onto weft:main beside the recorded anchor subpath".
- `internal/fabriccli/doc.go:19`: "a value that disagrees with the subpath recorded on weft:main is refused on a re-clone". Card 6 makes the main-weft marker the first source.
- `internal/fabricengine/unwire.go:60`: `Unwire`'s own doc says "Unwire never touches the repo-wide weft:main records". This is a function doc, not the file header the card names.

Neither `internal/fabriccli/doc.go` nor `Unwire` is on card 6's Edit list.

Tool calls:
1. Grepped production `.go` files for `.PullWeft(` and `replayWeft(` callers, to rerun F1's search. My context held only `parenttips.go` and pieces of the pull path.
2. Grepped `internal`, `cmd` and `docs` for `weft:main` lines that mention the anchor, to rerun F12's search. My context held only the four sites the review named.
3. Read `unwire.go:56-62`, `fabriccli/clone.go:1-6` and `:24-29`, and the line of `CloneAndWire`, to confirm the extra sites from call 2 talk about the anchor.
