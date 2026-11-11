```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests, one line per behaviour with the test that pins it, or none.

**Per behaviour**

1. Supersede drops only the same sender's unread, unreserved mail to the same receiver and topic. Pinned by `TestStore_SupersedeReturnsTheSendersUnreadMailUnderTheSameTopic` (`verbs_test.go`). Rows cover other topic, other sender, other receiver, a message already read, and an empty topic. A message reserved by an in-progress read is not covered.
2. A supersede answers a wait on the older mail with `OutcomeReturned`. Pinned by the same test, whose waiter assertion runs on every row.
3. A supersede does not hide the newest notice, which stays readable. Pinned by the same test (`wantBodies ... "new"`).
4. `DropRun` drops every unread, unreserved message tagged with the run, whatever its sender or holder, in id order. Pinned by `TestStore_DropRun` row 1. It leaves read mail, reserved mail, untagged mail and other runs' mail alone, and answers the dropped waiter.
5. `DropRun` with no match answers none and journals nothing. Pinned by `TestStore_DropRun`, row "AnswersNone…".
6. `DropRun` refuses a non-program sender and an empty run. Pinned by `TestStore_DropRun`, row "IsRefused…".
7. Retention carries Topic and Run through a journal replay. Pinned by `TestJournal_Retention`, whose new assertions cover tagged message 1 and untagged message 5.
8. The `drop-run` and `post` protocol fields travel through `server.go` dispatch and the client's `PostTagged` and `DropRun`. **None.** `TestStore_*` calls the store directly, and the creelcli test calls neither over a connection.
9. `DropRun` on a hub with no journal directory answers nothing and starts no daemon. Pinned by `TestGoClient_DropRunOnAHubWithNoJournalAnswersNothingAndConnectsToNoDaemon`. The journal-exists path that sends the verb is not tested.
10. `notifyPrime` carries the caller's tags on both the strand post and the program fallback. Pinned by `TestNotifyPrime_SignsAndPosts`, the tag assertion across rows.
11. `NotifyParent` tags `Run` with `pair.WorktreeName`. **None.** Tests call `notifyPrime` with fixed tags, and `NotifyParent` is not exercised.
12. `NotifyRunFrom` is used by `battencli.notifyRunParent` with `Run: slug`. **None.** There is no battencli test of the tag.
13. Dobby run notices post through `RunNotices.Post` with the run's slug. Pinned by `TestRunNotice_Conditions`, the `postedSlugs` assertion. Retry and drop through `Post` is pinned by `TestRunNotice_DeliveryIsRetriedOnProbeTicksThenDropped`.
14. The `beamcli` closure builds the per-run topic `dobby-run-notice/<slug>` and Run tags. **None.** The dobbyengine fakes bypass `hubDobbyDeps`.
15. `requirePosted` turns an unposted notice into an error so dobby retries. **None directly.** It is a small helper and the closure above is untested too.
16. A push-failure notice carries topic `push/<repo>/<branch>` and the pair's slug as run. Pinned by `TestFailedPushRecord`, the `NoticeTags_*` rows. They cover pair code, records, prime and bolt. A records branch with an empty `CodeWorktree` is the only `pairSlug` case with no row.
17. The remote-teardown notice carries topic `remote-teardown/<slug>` and no run. Pinned by `TestNoticeRemoteTeardown`. `TestNoticeRemoteTeardownEnded` and `TestRunTeardownChild_NoticesAnUnreadableJob` check the line only, not the topic.
18. `InstallPushNotifier` in orchcli passes the notice's tags to the post. **None.** `TestInstallPushNotifier` was only retyped, and no assertion reads the tags a posted notice carries.
19. `fabricengine.SyncOptions.notifier` passes the full `PushNotice`. Only indirectly, through `TestSyncOptionsNotifier`, which sends a line-only notice.
20. `RemovePair` drops run mail once, after a successful removal only, and before the claim settles. Pinned by `TestTeardown_CallSequence` and `TestRemovePair_ClearStray` (call order plus `droppedRunMail` slug). Not run on a removal error, or on `NothingOfThePairLeft`, which stays `NotFound`.
21. `dropPairRunMail` does nothing with no dropper installed and logs a dropper error without propagating it. Pinned by `TestDropPairRunMail_InstalledDropper`. That test also pins that the installed dropper receives the prime and the slug.
22. `InstallRunMailDropper` in orchcli (a `TEARDOWN` program client calling `DropRun`) behaves as documented. **None** for the closure itself.
23. The root pre-run installs the dropper. Pinned by `TestRootPreRun_InstallsTheHubNotifiers`, a source-text check that the call is present.
24. The teardown drops mail on every route, including hubs with no journal. Covered only by items 9 and 20.

**Findings**

- **MEDIUM, design.** The wire path for `drop-run` and for post tags has no test. `server.go:305` and `server.go:410` and `goclient.go` `PostTagged` and `DropRun` carry the fields. A dropped field there would make supersede and teardown drop silently do nothing, and every store-level test would still pass. Both verbs are exercised only against `Store`. A creelengine server-level or `creelcli` journal-exists test is needed.
- **MEDIUM, design.** The run tags set by the callers have no test: `orchcli/notify.go` `NotifyParent` (`Run: pair.WorktreeName`), `battencli/notices.go` (`Run: slug`), and the `beamcli/dobbydeps.go` run-notice topic. These tags are what make the dropped and superseded mail match. A wrong or empty value would make notice supersede and teardown drop do nothing, and no row would fail.
- **MEDIUM, design.** `orchcli/pushnotifier.go` forwards `notice.Topic` and `notice.Run` to the post. No assertion reads the tags a posted failed-push notice carried. The retyped `TestInstallPushNotifier` only changes signatures.
- **LOW, design.** `orchcli/runmail.go`, the `InstallRunMailDropper` closure, has no test. It is signed `TEARDOWN`, calls `DropRun`, and returns the count.
- **LOW, design.** `pairSlug` has no row for a records branch with an empty `CodeWorktree`, which should answer no run.
- **NIT, consistency.** `TestNoticeRemoteTeardownEnded` and `TestRunTeardownChild_NoticesAnUnreadableJob` check only the line, while the topic is set in the same function (`postRemoteTeardownNotice`) that `TestNoticeRemoteTeardown` pins.

**Test economy**
- **LOW, design.** `TestRootPreRun_InstallsTheHubNotifiers` now asserts the wiring through source-text matching. That is a static-prose assertion, but it extends an existing test and does not add one, so I raise no finding.
- Redundancy candidates all report "not judged". The wire-level tests I list above add behaviour no existing test covers, so none of them would duplicate coverage.

**Tool calls:** one, a read of the saved test diff `b6kanutz2.txt` lines 40 on, since the inline preview stopped at about 2KB.
