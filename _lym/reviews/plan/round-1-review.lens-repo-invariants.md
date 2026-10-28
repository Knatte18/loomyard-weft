```yaml
lens: repo-invariants
outcome: reported
```

Restating my task: repo-invariants lens. For each card I checked test seams and tier tags, doc text the card makes false, and whether the card can pass its own gate from the state the earlier cards leave.

**Per-card list**

- **Card 1 (orchroute-walk)**
  - Reflective seam: held. No `//lym:seam` or reflective test covers the moved members.
  - Tier tags: held. `TestLiveFrom` is tier0, `TestAsker` is tier1, and the TestMain pair `!tier1`/`tier1` copies orchcli's pair.
  - Import scans: held. No importer allowlist covers `shuttleengines`, `orchengine` or `hubgeom`. The `shuttleengines` and `orchengine` seam tests restrict only those packages' own imports.
  - Docs: held. `docs/overview.md` is extended, the new package has a `doc.go`, and orchcli's doc stays true through the delegations.
- **Card 2 (creel-post-parent)**
  - `clientSeam.live`: held. No reflective test covers `clientSeam`.
  - `creelcli/doc.go`: held, extended by the card.
- **Card 3 (hold-notice-creel)**: held. `shuttleengine/doc.go:99` is listed, and so are both tests that assert the notice.
- **Card 4 (notifier-asker-walk)**: held. `orchcli/doc.go:147-149` is extended. The burler, loom and webster `doc.go` files do not describe the notifier.
- **Card 5 (expect-reply-engine)**
  - Seams: held. No reflective test covers `Request`, `record`, `Message` or `LogEntry`.
  - Doc text: held. The creel doc's Mail section and its line form are listed.
  - Out of my lens: `LineFits` has callers in `internal/loomcli/decision.go:304,312`. That belongs to the affected-set lens.
- **Card 6 (expect-reply-cli)**: held.
- **Card 7 (chase-clock)**
  - Test helpers: held. `isSweepWait` is narrowed in creelengine, so the 60 s chaser wait is never stepped in place of a sweep wait. `unboundedWaitFiles` is extended.
  - `PATTERN-daemon-wakeups.md` is extended for the chase timer. Its sweep bullet ("runs while mail or injects exist") is touched in finding 2.
- **Card 8 (chase-reply-forward)**: held.
- **Card 9 (sweep-holder-gone)**
  - creelengine `doc.go` `# Removed holders`: held, listed.
  - `pattern/PATTERN-daemon-wakeups.md` sweep bullet: broken (finding 2).
- **Card 10 (unanswered-cli-log)**
  - creelcli `isSweepWait` (`cli_test.go:49`, `d > 8s`): held. The card's claim "no existing row posts a marked message" is false, since `--wait-reply` implies the mark from card 5 on.
  - It does no harm, though. The existing `--wait-reply` rows that step the sweep (`cli_test.go:530-535`, `996-1002`, `1145-1148`) remove the receiver before it reads, so the message stays unread and no chaser starts.
- **Card 11 (report-replies)**
  - `internal/reportengine` import allowlist: broken (finding 1).
  - Tier tags: held. `TestComputeReplies` is tier0 and `TestRepliesVerb` is tier1.
  - reportengine and reportcli docs: held, listed.
- **Card 12 (stencils-through-creel)**: held. No test outside the listed ones asserts the removed `SendMessage` prose. The pinned tail and the LOOM sentence are kept.
- **Card 13 (drop-peer-messaging)**: held. Every `peer-messaging` site is listed, including the attach row inside `TestRunner_Requires` (`requires_test.go:196`), `Seat.Requires` (`table.go:82`) and `seatengine/doc.go:13`. No md or yaml spec names it.
- **Card 14 (creel-trust-sentence)**: held. No doc or PATTERN text quotes the old wording.
- **Card 15 (remove-shuttle-send-interrupt)**: held.
  - Every `lym shuttle send|interrupt` site that `TestRemovedVerbs_NamedNowhere` would hit is listed: `run.go:1035` through `sweepOrphansOpportunistic`, `messagetail.go`, `interruptguard.go`, `claudeengine/command.go:294`, `docs/overview.md:413`, the PATTERN background and the sandbox suite.
  - `cmd/lym/sandbox_coverage_test.go` is module-level, so no entry goes stale.
- **Card 16 (agent-messaging-scan)**: held. The new test file is tier0. After cards 3 and 12, `holdNotice` is the only Go literal and the seat fixtures are the only testdata prompts.

**Findings**

1. **BLOCKING** · class `consistency` · card 11 (`plan:internal/reportengine#ComputeReplies`)
   - Summary: `ComputeReplies` takes `[]creelengine.LogEntry` and calls `agentname.Parse`/`ParseProgram`. Neither `internal/creelengine` nor `internal/agentname` is in reportengine's import allowlist, and the card does not list that allowlist. `TestToldGeometryInvariant_AllowlistOnly` fails on card 11's own tier0 gate.
   - Evidence: `internal/reportengine/seam_enforcement_test.go:13-29`. The `allowedImports` list has no creelengine or agentname entry, and the test calls `scankit.AssertImportAllowlistNoStale`. The list's own comment reads: "Each later engine card extends it with what that card imports."
   - Fix: add `internal/reportengine#allowedImports` under card 11's `**Extends:**`, gaining `internal/creelengine` and `internal/agentname`.
2. **LOW** · class `consistency` · card 9 (and card 7, which extends the same file)
   - Summary: card 9 widens the sweep's subjects to strands holding a read, marked, unanswered message. The `pattern/PATTERN-daemon-wakeups.md` bullet for creel's sweep still says it "runs while mail or injects exist". Card 9 does not list the file, and card 7 only adds the chase timer to it.
   - Evidence: `pattern/PATTERN-daemon-wakeups.md` bullet "creel's removed-holder sweep in `internal/creelengine/sweep.go`, which runs while mail or injects exist…"; card 9's Intent; card 7's Extends line `pattern/PATTERN-daemon-wakeups.md`.
   - Fix: card 9 extends `pattern/PATTERN-daemon-wakeups.md` so the sweep bullet names the widened subjects.

Outside my lens, noted only: card 7's chaser probes the holder through the delivery seam, which reads reed status. It does that at a fixed one-minute wait, with no backoff while the holder is dormant. That bears on `PATTERN-daemon-wakeups`' Backoff rule, which is the handler's PATTERN-fit check.

Tool calls:
1. Grepped reflective seam markers and read `internal/reportengine/seam_enforcement_test.go`: the context had no seam or import-scan tests for the packages these cards touch.
2. Grepped `LineFits` callers, the trust-sentence wording, the shuttlecli doc, the notifier docs and the shuttleengine doc lines: I needed the doc and caller sites the cards might falsify or miss.
3. Grepped the `--wait-reply` and sweep rows and the build tag in `internal/creelcli/cli_test.go`: I needed to check card 10's `isSweepWait` claim against existing rows, which my context had only partly.
4. Grepped for PATTERN.md `(test)` enforcement and cmd/lym package lists: I needed to know whether a new PATTERN entry or package trips a repo scan.
5. Grepped cmd/lym tests, plugins, tools and contracts for the shuttle verb names, and read the head of `sandbox_coverage_test.go`: I needed coverage tests that could keep the removed verbs.
6. Grepped import scans covering orchengine, reedengine and hubgeom, and read the `shuttleengines` seam test: I needed to know whether the new orchroute package breaks an importer allowlist.
