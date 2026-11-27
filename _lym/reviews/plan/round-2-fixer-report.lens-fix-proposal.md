```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the reviewer's proposals the fix carried out for F15 and F16.

**Proposals checked, F15:** cutting the card-15 halt and settling the count now; naming further engine deletions together with their holders.
**Proposals checked, F16:** a fixed store order; a `Store` for each field's error; `Phases` holding a phase only after its read succeeds; a failed re-read recording the phase it ran under.

1. **MEDIUM. F15: cutting the halt leaves nothing that acts if the part's test code grows.** This concerns the changed text at `00-overview.md:70-71` ("come to about 1,050; … about 700, so the part ends about 180 lines down").
   - **Case:** the margin of about 180 lines is within the error of the estimates it rests on. The review puts the additions at "800 lines or more", an open bound. The 1,050 figure comes from the plan's writing with no derivation shown. The 700 figure is a rough measure of spans: the whole-test spans are measured, but the tails and subtests were counted by eye.
   - **Effect:** if the additions come in about 200 lines over the estimate, the part grows. With the halt cut, card 15's commit body only reports the totals and no step acts on them. The added entry "This part's fabric test code shrinks in net" would then ship broken.
   - **Evidence:** `00-overview.md:70-73`, and the review's line 67 ("I estimate them at 800 lines or more").
   - **Possible fix:** state in the overview the limit the additions must stay under, or name further reserve deletions on card 15. Either way the plan says up front what pays for an overrun, rather than finding out at run time.

2. **MEDIUM. F16: reporting only the first failing store's error hides a second store that also failed.** This concerns the changed text at `15-snapshot-reader.md:83-86` ("a failing store does not stop the walk … the error returned is the first failing store's, whose `Store` it carries").
   - **Case:** the code and records origins are both unreachable, and the rule asks for `FieldOriginRefs`. The walk tries both stores, but `read` returns one `*fieldReadError`, the one for `StoreCode`. `runRule` turns one read error into one `ReadFailure` (`14-executor-loop.md:47`), so `ReadFailures` names only `StoreCode`.
   - **Effect:** the records store's `OriginRead` stays false and no `ReadFailure` names it. A rule cannot tell "records origin unreachable" from "records origin not read". Asking again repeats the same single failure, because the code store always fails first.
   - **Evidence:** `15-snapshot-reader.md:83-86`, `14-executor-loop.md:46-47`.
   - **Possible fix:** return one error per failing store, for example joined with `errors.Join`, and have `runRule` record a `ReadFailure` for each `*fieldReadError` it finds.

These held up when I checked them in the code:
- `TestGate_ZeroValueDeclarationsAreRefusals`' `PathZeroOwnership` and `PathZeroDirtiness` rows are held by the `WithAbsentTarget` rows at `destroy_test.go:336-358`.
- `TestDropHandOverMark` is held by `TestTakeOverRun`'s subtests, and `TakeOverRun` (`takeoverwrite.go:162`) is the only production caller.
- `TestCommit_TwoSided_WarpFirstOrdering`'s check that the Warp-SHA trailer equals `WarpSHA` is held by `TrailersEndTheWeftMessageOnly`'s `wantTail` (`commit_integration_test.go:331`).

Tool calls:
- `destroy_test.go:360-420` and a grep for its `WithAbsentTarget` rows: to check the holders card 15 names for the cut gate rows, which my context did not show.
- `TestTakeOverRun`, `TestCommitWithTrailers` and `TestCommit_TwoSided_WarpFirstOrdering` located by grep: to compare deleted tests with their named holders.
- The bodies of `TrailersEndTheWeftMessageOnly`, `TestTakeOverRun`'s subtests and `TestDropHandOverMark`: to compare their assertions.
- A grep for production callers of `DropHandOverMark`: to check card 15's claim that `TakeOverRun` is its only caller.
