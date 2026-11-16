# Card 21 — decision-notice-weaver

**Create:**
- `plan:internal/weaverengine#RecordDelivery` -> `func RecordDelivery(anchorRoot, strandName string, headings []string) error`
- `plan:internal/weaverengine#TestRecordDelivery` -> `func TestRecordDelivery(t *testing.T)`
- `plan:internal/loomcli#weaverDeliver` -> `func weaverDeliver(location *lymcwd.Location) func(strandName string, delivered []string) ([]string, error)`

**Edit:**
- `internal/loomcli#decisionDeps`
- `internal/loomcli#decisionState.preRun`
- `internal/loomcli#decisionFake.deps`
- `internal/loomcli#decisionNoticeInline` -> `func decisionNoticeInline(strand string, weaverActive bool) bool`
- `internal/loomcli#heldBackReason` -> `func heldBackReason(strand string, halted shedengine.State, weaverActive bool) string`
- `internal/loomcli#addedNoticeLines` -> `func addedNoticeLines(deps decisionDeps, added []discussionparser.AddedDecision, strand string, weaverActive bool) []string`
- `internal/loomcli#notifyLiveRuns` -> `func notifyLiveRuns(deps decisionDeps, halted shedengine.State, linesFor func(strand string, weaverActive bool) []string) ([]string, []decisionNoticeHeldBack, []decisionNoticeError)`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#decisionReplaceVerb`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#nilLegal`
- `internal/loomcli#TestDecisionVerb_NotifiesLiveAgents`
- `internal/loomcli/doc.go`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/weaverengine#ActiveOrSetting`
- `internal/weaverengine#BatchStrandRole`
- `internal/gatefixer#RoleVerifyFixer`
- `internal/loomcli#decisionNoticeLine`
- `internal/loomcli#liveRunStrands`

**Intent:** Weaver's strands get the parent's decisions while their run runs, and each delivery is recorded, keyed by strand name, the key the notice loop holds (D12, D13).

- `decisionNoticeInline` and `heldBackReason` treat a strand whose role matches `weaverengine.BatchStrandRole`, or `gatefixer.RoleVerifyFixer` while `weaverActive` is set, as an implementer strand: it gets every entry's text inline, and a halted run's such strand is held back, as Webster's Master session is.
  `notifyLiveRuns` computes `weaverActive` once from the seam's `ActiveOrSetting` answering Weaver and passes it to `heldBackReason` and to its `linesFor` callback.
  `decisionVerb`'s callback hands it to `addedNoticeLines`, which passes it to `decisionNoticeInline`; `decisionReplaceVerb`'s callback ignores it, since its one line does not depend on the strand.
  The advisory read falls back to the setting on a seam refusal and logs its reason, so a corrupt Webster state never blocks a notice.
- `decisionDeps` gains `recordDelivery func(strandName string, headings []string) error`; `decisionState.preRun` fills it with `weaverengine.RecordDelivery` over the target's anchor, and the test fake records each call.
  `notifyLiveRuns` calls it for each notice posted to a Weaver strand, with the headings of the entries posted; a failure is logged and reported as the strand's notice error, never stopping the next post.
- `RecordDelivery` adds the headings, under Weaver's state-mutation lease, to the `Delivered` list of the attempt or fixer whose `StrandName` matches; a name state does not record, or an absent state, is a no-op.
- `weaverDeliver` is `RunDeps.Deliver`'s production form: it reads the decision record's added entries, keeps those whose headings `delivered` lacks, posts them to the strand as one inline notice (`decisionNoticeLine`) through creel as the hub's LOOM program, and returns the headings posted.
  `loomCLI.wire` sets `WeaverDeps.Deliver` to it, and `nilLegal` drops `WeaverDeps.Deliver`.
  `Run`'s resume sends a probed strand only the entries its `Delivered` lacks (card 13).
- `loomcli/doc.go`'s notice paragraph names Weaver's strands.

`TestDecisionVerb_NotifiesLiveAgents` gains rows: a running Weaver run's batch strand gets the inline notice and `recordDelivery` receives its name and headings; a running Weaver run's `verify-fixer` strand gets the inline notice too; a halted run's Weaver strand is held back and nothing is recorded; a corrupt Webster state under a Weaver run still notifies.
`TestRecordDelivery` is tier0: the matching attempt records the headings once; a matching fixer records them; an unknown name and an absent state change nothing.

**ImpactSummary:** Two notice predicates gain Weaver's roles, the notice loop records deliveries through one new deps field, and loom's wiring fills Weaver's delivery seam; Webster's notices are unchanged.
