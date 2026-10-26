# Webster audit: a card slug ending in -weft makes every gate test --card command a fabric reference

`weft-suffix-card-slug-audit` · Task · bug, webster, fabric · running · PR-Review

Imported from [issue #874](https://github.com/Knatte18/loomyard/issues/874).

Seen (umbrella-anchor, batch 06, 2026-10-25): the implementer fork for card `6-clone-records-marker-on-prime-weft` ran `lyx gate test --wait --gate card --card 6-clone-records-marker-on-prime-weft`, and record-batch marked the batch failed with a fabric-reference finding. The fork touched no fabric repo.

Cause: ReferenceRule's barePattern in internal/fabricengine/refscanner.go flags any unquoted word ending in the weft suffix, so a card slug ending in `-weft` turns every command naming the card into a fabric reference. recover-batch then refused with needs_fresh.

Workaround: the parent judged the finding false and had Merriam run `lyx webster accept-audit --batch 6` and `recover-batch 6`; later forks quote the slug.

Expected: a card slug is never read as a weft sibling worktree; either the plan validator refuses a card slug ending in the weft suffix, or the audit does not read a `--card` value as a path.