```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. I found three leftovers, listed per change below.

**Per change: the set found, covered or not**
- **`PushNotifier` signature (`hubPath, line` → `hubPath, PushNotice`).**
  - Set: `orchcli/pushnotifier.go`, `fabricengine/remoteteardown.go`, `fabricengine/failedpush.go` and `fabricengine/pushnotifier_helper_test.go`.
  - Set (continued): `fabricengine/remoteteardown_test.go:25`, `orchcli/pushnotifier_integration_test.go` and `orchcli/notifierwiring_test.go`.
  - Covered. No other implementer or caller turned up in the grep.
- **`notifyPoster.Post` → `PostTagged`.**
  - Set: `notifyPrime` and `postNotice` in `orchcli/notify.go`, and `notify_test.go:92`.
  - Covered. The production client is `creelcli.GoClient`, which has `PostTagged`.
- **New `postNotice`/`notifyPrime` tags parameter.**
  - Set: every `NotifyPrime`, `NotifyParent`, `NotifyFrom`, `NotifyFromScoped`, `NotifyAsker` and `NotifyOrchAt` wrapper in `orchcli/notify.go`.
  - Set (continued): the `pushnotifier.go` call site and the `notify_test.go` call.
  - Covered. The external callers (`loomcli`, `adoptcli`, `beamcli`, `battencli`) keep their signatures.
- **Run notices now go through `RunNoticeDeps.Post`, no longer `Deps.Notice`.**
  - Set: `dobbyengine/runnotice.go`, `dobbyengine/doc.go`, `beamcli/dobbydeps.go:138-141`, `dobbyengine/runnotice_test.go:51`, `beamcli/dobbydeps_integration_test.go:114-121` and `battencli/dobbynotice.go`.
  - Production wiring and the unit test are covered.
  - **Finding 3:** the integration test's "every seam filled" check is not covered.
- **`pairteardown` `dropRunMail` / `InstallRunMailDropper`.**
  - Set: `cmd/lym/main.go`, `orchcli/runmail.go`, `pairteardown/{runmail,teardown,doc}.go`, `pattern/PATTERN-pair-remote.md` and `creelcli` doc.
  - Code and docs are covered. **Findings 1 and 2** are about the pattern and doc text.
- **Creel wire and verb additions (`drop-run`, `Topic`/`Run` fields).**
  - Set: the verbs header in `verbs.go`, `creelengine/doc.go` and `creelcli/doc.go`. `crosshub.go` `postOtherHub` was not read, so I have not verified the doc claim that a cross-hub post drops the tags.
  - Set (continued): `docs/overview.md:873` lists the CLI verbs, and `drop-run` has no CLI verb, so it needs no entry.
  - Covered.
- **Commit trailers.**
  - Neither commit carries a `Pattern-Departure:` trailer, and I found no departure that needs one.

**Findings**
1. **MEDIUM, consistency.** `pattern/PATTERN-pair-remote.md:70-74`.
   - The new bullet "After a successful removal the composite drops the hub's unread creel mail…" was inserted in the middle of the preceding board-settle bullet.
   - The lines `A \`running\` record with no live registered batten process means the run is gone.` and `The composite keeps the claim and warns with \`lym batten cancel <slug>\`.` (lines 73-74) now read as part of the drop-mail bullet.
   - The settle bullet loses its continuation, and the drop bullet gains text that has nothing to do with it.
   - Fix: place the new bullet after line 74.
2. **LOW, consistency.** `internal/pairteardown/teardown.go:432-434`.
   - The new paragraph "A successful removal drops the hub's unread creel mail…" sits before the doc comment's `// RemovePair removes the task's pair.` sentence, after the `ClearStray` paragraph.
   - The symbol's own sentence now opens mid-comment under the new paragraph.
   - Fix: move the paragraph below the `RemovePair removes…` sentences, or after the `ClearStray` paragraph with the self-reference first.
3. **MEDIUM, design.** `internal/beamcli/dobbydeps_integration_test.go:115`.
   - The "every seam filled" check lists `ReadChild`, `DriverAlive`, `Agents`, `NewestAgentEnd`, `Pause`, `ReviveAPIError` and `MarkerPath`, but not the new `Post`.
   - `deliverRunNotice` returns false on a nil `Post` (`runnotice.go`), so a wiring regression silently sends no run notices from any dobby.
   - Fix: add `notices.Post == nil` to that condition.

**Tool calls**
1. Grep for `PushNotifier` and `SyncOptions` users, the `RunNotices` and dobby `Deps` constructions, and `withdraw` in non-Go files: the manifest lacked implementers and fixtures outside the diff.
2. Grep for creel verb and protocol-field docs, the posted and carried event sites, and callers of the `Notify*` wrappers: the manifest lacked the callers of the changed signatures.
3. Grep for the undelivered-reason consumers and post paths, plus a read of `dobbydeps_integration_test.go:100-135`: the manifest lacked the seam-filled check and the cross-hub post path.
