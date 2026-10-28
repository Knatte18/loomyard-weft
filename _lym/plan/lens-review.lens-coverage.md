```yaml
lens: coverage
outcome: reported
```

coverage lens: every decision and acceptance criterion checked against the cards, with the planned rows that reach them.

**Findings**

1. **BLOCKING** — D10 sender-notice rule, card 7 (`07-chase-clock.md`), `settleUnansweredLocked`. Card 7 sends the notice to "a program whose mailbox is known", but a program sender's mailbox is always known by the time its message is chased, because `pool.applyPosted` sets `p.programs[rec.From] = true` for every program that posts (`internal/creelengine/fold.go`, `pool.applyPosted`). D10 and its criterion say the notice goes only to "a program that has read once", and that "a program sender that never read gets no notice". As written, the card's own row ("a program sender that never read gets no notice") fails. If the row were bent to pass, the code would ship the wrong behavior. The rule needs a read-once record of its own, separate from the post-or-read `programs` map.

2. **MEDIUM** — criterion "D1, D3: in a pair and in a beam, the `--parent` receiver and the hold notice's orch are the same strand", card 4. Card 4 argues this holds "by construction" and plans no row: "no second test of the delegation is planned". Nothing tests that `NotifyAsker` calls `orchroute.Asker`. Nothing tests that the three `SetNotifier` closures (`internal/burlercli#burlerCLI.wireHub`, `internal/loomcli#loomCLI.wire`, `internal/webstercli#websterCLI.wireHub`) call `NotifyAsker` and not `NotifyParent`. Reverting any one of them passes every planned row.

3. **MEDIUM** — D10 criterion "a reply to a chase … by a strand that forwarded the chased message after the chase is refused naming the way forward", card 8 and the Shared Decision `former-holder-reply-guard`. The plan waives this criterion's test by declaring the branch unreachable. The premise holds: `Store.Forward` refuses a read message (`internal/creelengine/verbs.go`, `Store.Forward`, "is already read and cannot be forwarded"), and a chase only follows a read. But dropping a decided acceptance row is a design change. It needs an `Added after Discussion` entry recorded by the parent, and the decision record carries none. The plan records it only as its own Shared Decision.

4. **MEDIUM** — D12 verb wiring, card 11 `plan:internal/reportcli#repliesCommand`. The only planned test is the pure `TestComputeReplies`. No row reaches:
   - the verb's refusal outside the prime ("runs from the prime");
   - the `GoClient.Log` read;
   - the `roles` envelope.

   `TestReportVerbs_RefuseOutsideThePrimeThenReportTheSeededRun` and `TestReportVerbs_RefuseBadFlags` cover only `convergence` and `timing`.

5. **MEDIUM** — D13 scan surfaces, card 16 `plan:cmd/lym#TestAgentMessaging_NoEngineTool`. `FixtureRows` reaches only the stencil allowlist branch. Three branches are reached only by `Tree` over a clean tree, so a broken extractor still passes because `RequireFloor` counts files, not literals:
   - the Go string-literal branch (`*ast.BasicLit`);
   - the comment-is-no-hit branch;
   - the `testdata` `.md` surface.

   The criterion "the scan fails on a … testdata prompt or Go literal naming `SendMessage`" has no row that fails if that half breaks.

6. **MEDIUM** — D8 criterion "`lym shuttle send` and `lym shuttle interrupt` do not exist", card 15. No row asserts the verbs are gone:
   - `TestRunCLI_PositionalArgValidation` only drops its send and interrupt rows.
   - `cmd/lym#removedVerbs` matches the text "lym shuttle send", which a cobra `Use: "send &lt;guid&gt; &lt;text&gt;"` never contains (`internal/shuttlecli/send.go`).

   Re-registering `sendCmd` passes every planned test.

7. **MEDIUM** — D10 branch: a reply wait registered after the `unanswered` record, card 7 `internal/creelengine#Store.settledOutcomeLocked`. No row reaches the new `OutcomeUnanswered` case of `settledOutcomeLocked`. That is the path a reconnecting `--wait-reply` takes (`internal/creelengine/client.go`, `Client.resume` → `VerbWaitReply`).

8. **MEDIUM** — D10 criterion "a pause or another sender's hold stops the clock … a restart after a pause counts no paused time", plus `chase-aged` persistence, card 7. The restart row covers only `chase-aged` written when the clock stops. No row covers `chase-aged` written every `agedPersistInterval` while the clock runs, though D10 requires it and a crash restart depends on it.

9. **MEDIUM** — D10 read-wake paths, card 7 (`internal/creelengine#Reading.Ack`, `internal/creelengine#Store.markNoticeRead`). Card 7 changes both to wake the chaser. Its rows read the marked message through one unstated path. The other path is unreached:
   - The batten row's message is a program notice, which `markNoticeRead` journals read on a pane-mail strand.
   - A strand's `Read` / `Ack` path wakes the chaser in a separate branch.

10. **LOW** — D10 "A reply that comes later is still delivered", card 7. No row posts a reply after the `unanswered` outcome and asserts it reaches the sender.

11. **LOW** — card 7's `isChaseable` "holder is a strand" branch. No row shows that a marked message held by a program is never chased.

12. **LOW** — card 7's EXTERNAL-sender row asserts only the chase's text. It does not assert that the unanswered outcome posts no notice to EXTERNAL and leaves the journal record alone.

13. **LOW** — D2, D4, D5 and D6 criteria, cards 12 and 14:
    - the rendered directive names `--parent --expect-reply` and `--reply`, and trusts orch strands and lym programs;
    - every question site carries `--expect-reply`;
    - the appended sentence names parent, orch and program mail as followable.

    The plan argues these hold without a test; only the `SendMessage` absence is scanned (card 16). This is a criterion carried without a row, though the rubric discourages asserting a stencil's static prose. Reported for the record.

14. **LOW** — D13 scope, card 16. D13 names "any file under `contracts/stencils/`". The scan reads only `.md` there, and Go literals only outside `_test.go`, so `contracts/stencils/*_test.go` literals are unscanned. Today that directory holds only `.md` and `.go` files, so the gap is narrow. The allowlist's "role's spec sets `ForkSubagents`" half is left to review.

**Tool calls**
- `find contracts/stencils`, `grep SetNotifier(`, the functions enclosing `run_test.go:1023` and `run_inject_test.go:91`: my context lacked the full set of stencil file types, the full list of notifier wirings, and which tests hold the refusal assertions.
- `sed internal/battencli/drive.go:440-456`: to check whether a fourth notifier wiring, which card 4 does not list, uses a different rule. It does not; it already calls `notifyRunParent`, the run-notice rule.
