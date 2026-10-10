# webster: fabric-reference audit fails a piped lyx help call inside a compound command

`audit-piped-help-call` · Note · bug, webster, fabric

Imported from [issue #518](https://github.com/Knatte18/loomyard/issues/518).

Seen in run quick-recipe (2026-10-10): batch 01 failed record-batch on a fabric-reference finding for the fork command `sed -n 285,300p contracts/specs/refusal-spec.md | cut -c1-200; grep -n "fabric commit" -r contracts/specs plugins 2>/dev/null | head -5; lyx fabric --help 2>&1 | head -30`. recover-batch refused with needs_fresh.

Workaround: `lyx webster reset --to batch-start --batch 01`, `lyx webster accept-audit --batch 1`, `lyx webster recover-batch 1`; the batch is redone.

Expected: a lyx help call piped into a reader (`lyx ... --help 2>&1 | head`), within a compound of read-only segments, counts as read-only. Same class as #514, whose fix (run webster-audit-gaps) admits --help only as the last word of a segment.