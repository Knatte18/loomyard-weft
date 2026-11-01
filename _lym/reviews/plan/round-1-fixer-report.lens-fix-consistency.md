```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency report: whether the changed text agrees with the sibling cards, the decision record and the code it depends on.

**LOW** — card 11, Choice (D2), the new Bound sentence ("each holder's run being bounded by its tier's or build step's `timeout_min`").
Summary: the sentence leaves out one kind of slot holder that card 14's new text names in the same change set.
That holder is a card-verify slot-held line, and the `timeout_min` values do not bound it.
Evidence:
- Card 14's new sentence says a gate call without `--wait` "is a slot-held line, run inside one slot under the card-verify runner's timeout".
- `internal/cardverify/doc.go:10` says such a line "takes one slot of the pool … and runs under the timeout counted from the moment the slot is held".
- `internal/cardverify/runner.go:20-21` sets that timeout to `DefaultTimeout = 10 * time.Minute`.
- So an in-process call can wait on a holder that only the card-verify line timeout bounds, which card 11's bound does not name.
- Suggested wording: "…its tier's or build step's `timeout_min`, or a card-verify slot-held line's timeout, …".

Checks that found nothing wrong:
- **F1:** The decision record's D3 says the front writes the lease "under the key directory's lock". Card 10's `state.UpdateJSON` takes its own `lease.json.lock` while the front still holds the key directory's lock, so both statements hold. Card 12's "chosen cap is … its lease" still agrees with keep-the-later.
- **F1:** `current.json` has a lock path of its own, separate from the `lock` the front holds, so it cannot deadlock against it.
- **F3:** Card 11's `TestJoinable` row agrees with the `joinable` rule and with `startReading`'s signature in card 10. Nothing in the plan still references `TestDetachStore` or `TestStartReading`.
- **F2:** Card 11's Texts paragraph (help loses the cap) and card 4's `DefaultAwaitCap` doc change both agree with removing the subtest.
- **F4:** Spec check 44's way forward (`--wait`) still holds for a Go-run line. The `plancontext.go` comments carry no rationale, so nothing there goes stale.
- **F6:** The new no-Verify branch agrees with card 13's rule that the front starts only for a card that has a Verify line at the front's read.
- **F7:** The `none` helper line agrees with the row that asserts it.
- **F8:** The new `awaitVerb` comment agrees with card 4's "the verb still needs no hub or repository".
- **F9:** Cards 8 and 9 now list their docs under Edit, and nothing else refers to them as Extends.

Tool calls:
1. Grepped the plan cards for references to the changed text, because my context had no sibling-wide search.
2. Read the cardverify timeout lines and the list of decision-record files, because my context did not show cardverify's runner bound.
3. Grepped the decision record for lease, cap and tolerant-records text, because my context did not include the decision record.
4. Grepped card 12 for lease and run-id references, because my context did not include card 12.
