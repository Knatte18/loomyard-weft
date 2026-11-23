```yaml
lens: fix-affected-set
outcome: reported
```

Task: for each fix whose finding covers a set of sites, rerun the review's search and check the fix covers every member.

**Findings left unchecked:** none. F1, F2, F7, F8, F10 and F11 were rerun. The rest have no set premise.

No findings.

- **F1:** these production `PushRun(prime, prime, slug, …)` callers name a slug:
  - `battencli/commitstatus.go:172`
  - `battencli/cancelcmd.go:352`
  - `battencli/catchup.go:266`
  - `pairteardown/handover.go:185`
  - `tendercli/runintents.go:242`

  Card 3 now reads takeovers on every prime records push "whatever `runSlug` is" and gives its callers only as examples ("such as"). Card 4 says the `*TakeoversError` branch "serves every prime push", so the catch-up and tender callers the review did not name are covered too.
- **F2:** other seam lists over `Teardown`:
  - `shedrecipe/entries_batten.go:67-70` requires only `Shutdown` and `Remove`. `Push` and `Level` are nil-legal, so it needs no edit.
  - `battencli/wire_test.go:64-65` is an explicit list, not a reflective one. It cannot fail when the new fields are added, and battencli fills both.

  The two reflective maps the review named, `nilLegal` and `intentionallyNil`, are now under card 10's **Edit:**.
- **F7:** a search for "synchronous" and "owes no lift" found only the two tender texts card 7 now edits. Card 7 already covers the other transition-push texts: `statuscommit.go:2/34`, `loomcli/wiring.go:208/342` and `battencli/commitstatus.go:212`. The remaining hits describe pushes that stay synchronous and fall outside F7's set.
- **F8:** the owed rows `owedRows` (`PrimeRecordsPushOwed_*`), `Deferred_*` and `TeardownLockHeldPastTheWaitOwesTheRecordsPush` are all named in the rewritten card 1. Card 11's reference to the dropped test was retargeted as well.
- **F10:** `RestartPusher` appears only as `runpusher` in cards 5 and 8, with no leftover `battencli#RestartPusher`.
- **F11:** `pushanchored.go:23` is the only doc naming the commit-records or transition push. The other `StatusPushLockWait` docs (`ownerarchive.go:50`, `takeoverwrite.go:46`) do not make that claim.

Tool calls beyond context:
1. Grepped the `PushRun(` callers and the `Teardown.Shutdown|Remove` lists, to rerun F1's and F2's searches.
2. Grepped the synchronous-push docs and the `StatusPushLockWait` comments, to rerun F7's and F11's searches.
3. Grepped `RestartPusher` in the plan and the owed-row names in the test file, to rerun F10's and F8's searches.
4. Read the `shedrecipe` and `battencli` `wire_test` seam lists and `remove.go:200-212`, to check F2's extra list members and a leftover synchronous-push comment.
