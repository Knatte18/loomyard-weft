```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward — checked each refusal, stop, reset or recovery step the cards add for a named next step that can succeed from the state it leaves.

**Findings**

1. **BLOCKING** — Card 7 (push-side-core), the verdict's way forwards and typed errors.
   - **Summary:** the verdict is built only from the snapshot's `Outcome`s, which keep a class and a detail string, so the error objects that carry each refusal's way forward are gone.
   - **Evidence:** `internal/fabriccore/action.go:81-86` gives `Outcome` only `Action`, `Class` and `Detail`. Card 7 says `pushClassified` "writes the record `JudgePush` names" from that snapshot. Today's record and error take data from the typed errors:
     - the `anchor_change` record copies `anchorStop.WayForward` (`internal/fabricengine/pushclassified.go:558-561`), and `FailedPushWayForward` returns `f.WayForward` for that class (`failedpush.go:286-287`). Rebuilt from `Detail`, the record's way forward is empty, and `PushFailedError.Error()` ends in "way forward: " followed by nothing.
     - the replay conflict's paths (`pushclassified.go:641-646`) feed `recordsConflictWayForward`'s "settle the named paths" line.
     - the takeover error has to reach callers as the same object. `internal/pairteardown/handover.go:127` and `:134` use `errors.As` for `*TakeoverError` and `*TakeoversError`, and `TakeoverError.Error()` holds the "`lym adopt` takes it back" way forward (`takeover.go:36-38`). `IsPushOwed` reads `ErrTakeoverPassOwed`.
   - **Needed:** the card should say that the engine keeps each action's error by outcome and returns or classifies that error, not the rebuilt text. The card does not say this.

2. **MEDIUM** — Card 7, the ended-entry tracking-ref delete.
   - **Summary:** the card turns `dropTrackingRefOfEndedBranch` into a local `ActionRefDelete` but gives no verdict for that delete failing or being refused.
   - **Evidence:** today a failure returns "fabricengine: delete the tracking ref of the branch %s, whose entry origin ended" (`pushclassified.go:785-787`). Under D4 a destructive action runs through `destroy.go`, whose guard can refuse it (`OutcomeDestructiveRefusal`). Card 7's verdict list covers only "no push and no record", so a refused or failed delete has no next answer, error or way forward. `TestPushSide` has no row for this outcome.

3. **MEDIUM** — Card 1 and card 3, the unset-seam refusal's way forward.
   - **Summary:** the `WayForward` says "install fabric's required seams through `fabriccli.InstallRequiredSeams`". That is a Go call no operator or agent can make, so the refusal names no runnable step.
   - **Evidence:**
     - `cmd/lym/main.go` installs inside `if !testing.Testing()`, so the lym binary never meets the refusal. Only a test binary that builds no hub, or one that unsets a seam, meets it (D5). So the reader is a developer.
     - The text says "as the lym binary and `internal/hubforge` do", but neither card names the fix for a test that builds no hub: build the hub through `internal/hubforge`, or call the installer in `TestMain`.
     - `PATTERN-refusal-way-forward` asks that a refusal name a way forward that progresses. The card should name the `TestMain` or hubforge step for a test binary.

4. **LOW** — Card 7, `UnseededBoardRefusal`.
   - **Summary:** the card says the refusal is "returned before any lock", which the code it moves does not do.
   - **Evidence:** `pushClassified`'s callers already hold the push lock when it runs. `coalesceFlagged` runs the step under `fabric.push.lock` (`pendingflags.go:142-148`), and the board refusal runs inside `pushClassified` (`pushclassified.go:140-143`). The way forwards themselves are kept word for word and still progress, so only the claim is wrong.

5. **LOW** — Card 7, the new row `AnOriginTipMovedAfterTheReadLosesTheLease`.
   - **Summary:** the recovery end is unnamed and the setup is doubtful.
   - **Evidence:**
     - A lost lease makes `resetUnnamedTip` return `(false, nil)` (`pushclassified.go:465-466`). That leads to `errOriginAhead` and then `failRejection`'s `not_fast_forward` record, whose way forward is `codeDivergedWayForward`'s merge. The card says only that "the push records the rejection". It does not assert that class or way forward, so the row does not pin the next step.
     - The row moves origin's tip from a pre-receive hook of the leased push itself. That needs the hook to push to the same repository again without recursing. The card names no guard for this, so the setup may not reach the lease check as claimed.

6. **LOW** — Card 9, the run-scope refusal.
   - **Summary:** the refusal "no pair worktree %s" is carried "as today", with no way forward.
   - **Evidence:** `originreconcile.go:455-456`. The card moves it into the new pre-lock phase as a read error and could have added the way forward (`PairMayBeAbsent`, or re-adding the pair). It names none. This is an old gap carried forward, not one the card adds.

**Per refusal: the next step, and whether it progresses**

| Card | Refusal or stop | Next step | Progresses? |
|---|---|---|---|
| 1 | `RefuseUnsupplied` | install the seams | Only for a developer (finding 3) |
| 4 | `OriginRunStatus` / `OriginCodeTip` refusal | same | Same as card 1 |
| 7 | Unseeded board | today's four steps | Yes |
| 7 | gone / rewound / transport / auth / not_fast_forward / other records | today's texts | Yes, except anchor_change and the conflict paths (finding 1) |
| 7 | Takeover | `lym adopt` | No, the text is lost (finding 1) |
| 7 | Ended-entry delete failure | none | No (finding 2) |
| 8 | Owed deferred, transport and teardown_lock | retry | Yes |
| 8 | `ErrEntryEnded` | teardown | Yes |
| 9 | Unreachable | today's callers | Yes |
| 9 | Run scope with no pair | none | No, carried from today (finding 6) |
| 10 | Busy teardown | `busyWayForward`, word for word | Yes |
| 11 | busy / live / anchor_stopped / dirty | `branchStopWayForward` | Yes |
| 11 | Live-at-execution and dirty-at-execution refusals | same lines | Yes |

**Tool calls**
- Listed the main packages and cliwire pre-run hooks, to see whether any process besides `cmd/lym` runs fabric verbs without the installer. My context had no list of main packages.
- Grepped `tools/` and `cmd/testtiming` for calls to seam-using verbs (none found), and looked for `TakeoverError`'s definition. I had neither in context.
- Grepped for callers outside fabric that use `errors.As` or `errors.Is` on the takeover, hand-over and anchor errors, and read `TakeoverError`. My context lacked `takeover.go` and those callers.
