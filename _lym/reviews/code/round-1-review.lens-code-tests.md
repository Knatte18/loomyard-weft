```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that would fail if it broke, or none.

**Per behaviour: pinning test**

1. **`resume_on_wake` defaults off, and `--resume-on-wake` or `=false` overrides it, when the flag is given or absent.** Pinned by `TestChooseResumeOnWake` (`internal/hibernatecli/cli_test.go:14`). Its four rows are no flag with no file, no flag with the file on, flag on over a file that is off, and flag off over a file that is on.
2. **Request records `ResumeOnWake`.** Pinned by the `TestSubmit` row "the resume-on-wake choice is written into the request" (`verbs_test.go:123`), for both values.
3. **On path (lift on every path after the wake).** Pinned by the existing wake rows. `ownRequest` now sets `ResumeOnWake: true` (`world_test.go:475`), so those rows still pin the lift.
4. **Off path, success: reconcile and pair holds run, request ends done with the hub entry owned and live, and the notice names the kept pause.** Pinned by `watch_test.go:570`, which asserts one reconcile, `PhaseDone`, `Owned` set, live scopes hub + p1, and the notice text.
5. **Off path, failure at the wake leaves the entry live.** Pinned only for the `reconcileAtWake`-independent TenderStatus read failure. The row at `watch_test.go:589` forces the wake's status read to fail, and that read happens inside `reconcileAtWake`. It asserts `PhaseFailed`, the owned entry live and the notice.
6. **`failAtWake` after `advance(PhaseResuming)` fails.** None. No test makes the resuming-phase write fail with the switch off. The same failure with the switch on is covered by the existing rows that already pin a failed write at the wake. Both paths call `failAtWake`, so the off branch of that call site is unpinned.
7. **`failAtWake` with `lift` true, when the switch is on.** Pinned by the existing on-path wake-failure rows, because the fake world's `ownRequest` is on.
8. **`keptOutcome` with `Owned == nil`, off path, no recorded entry.** None. Row `watch_test.go:721` covers the on-path case only. Nothing checks that the off-path notice with no owned entry reads "it owned no pause…" rather than the kept sentence. The off-path failure with `target == nil` ("it owned no pause to lift.") is also unpinned.
9. **Off-path failure where `finish` fails to write.** None. `failRequest` with `lift` false and a failed write is unpinned. The on-path equivalent is covered.
10. **Pre-wake failures still lift whatever the switch says (`fail` passes `true`).** None. Every `fail` row runs with the on request. Nothing runs a pre-wake failure (pause error, confirm timeout, no wake) with `ResumeOnWake = false` to check the entry is still lifted. A change to `fail` that stopped lifting for an off request would pass the suite.
11. **`lym hibernate cancel` lifts a kept pause for a terminal request.** The row at `verbs_test.go:299` pins a terminal request whose owned entry is still live. It does not use a request that actually came through the kept path, but the engine does not branch on the switch, so the row pins the behaviour for both done and failed phases only if it covers both. I read it as `done` or one phase, with no row for `failed`. Low risk, since `Cancel` ignores the phase.
12. **Cancel refusal message.** The diff changes the string. The existing refusal row presumably asserts "way forward", and nothing pins the new wording. Static prose, exempt.
13. **Hibernate config module: key registered hub-wide and local layer refused.** Pinned by the edited `TestModules_SeedOnlyAndHubWideFlags` (`configreg_test.go:58`). `TestModules_EveryTemplateKeyClassified` (`configreg_test.go:75`) derives `LocalKeys` and compares it to `wantLocalKeys`. The diff does not show `wantLocalKeys` gaining a `hibernate` entry, so whether that row would fail for the new key is not shown in this diff.
14. **`LoadConfig` degrades to the template with no file, and rejects a bad value.** The "no flag and no hub file" row of `TestChooseResumeOnWake` pins the degrade. No test pins the `MarkInvalid` unmarshal failure, and none pins a config load failure refusing the request. The unmarshal failure path is unpinned (`config.go` unmarshal branch, `cli.go:116-123` error return).
15. **`internal/hibernateconfig` pinned in the degrading `LoadOrTemplate` set.** Pinned by the `configstrictness_test.go:60` row.
16. **`lym hibernate status` shows `resume_on_wake`.** The summary claims it. It follows from the JSON tag on `Request` and no test asserts it. The key's presence in the written request file is covered indirectly by the `TestSubmit` row, which compares struct values, not the JSON key.
17. **A request file with no `resume_on_wake` key reads as off.** None. No test decodes a legacy request file.

**Findings**

- **MEDIUM, design:** Behaviour 10 has no test (`internal/hibernateengine/watch.go:296-298`, `fail` passing `true`). The decision record says a failure before the hibernate lifts the entry whatever the switch says. No row exercises a pre-wake failure with `ResumeOnWake = false`. Evidence: `watch_test.go` has `watchKeeping` (line 40) and uses it only in the two off-path rows (lines 570 and 589), and `ownRequest` is on in every other row (`world_test.go:475`).
- **MEDIUM, design:** Behaviour 6 has no test (`watch.go:139-142`, `advance` failure routed to `failAtWake`). The off-path row at `watch_test.go:589` reaches `failAtWake` only through `reconcileAtWake` (`watch.go:152-155`). Reverting the `advance` call site to `fail` would not fail any off-path row.
- **LOW, design:** Behaviours 8 and 9 have no test. The off-path notice with no owned entry (`watch.go:219-224`, `keptOutcome` with `Owned == nil`) and an off-path failure that records no entry or hits a failed write (`watch.go:314-322`) are both untested.
- **LOW, design:** Behaviour 14 is untested. No test covers a config load or unmarshal failure refusing `hibernate request` (`cli.go:119-122`, `config.go` `MarkInvalid` branch).
- **LOW, consistency:** Behaviour 17 is untested. A pre-upgrade request file with no `resume_on_wake` key decodes as off, and the diff names this as the intended compatibility. No row decodes such a file (`request.go:62-63`). With the switch off, a daemon that survives an upgrade would keep its pause, a behaviour change no test pins.

No redundant or duplicate new test found. The three new rows each pin a distinct behaviour.

**Tool calls (1 of 6)**
1. A Bash `ls` and `grep` over `internal/hibernateconfig` and the hibernate tests, because my context lacked the test files' existing rows and the registry's `wantLocalKeys`. The grep output shows `wantLocalKeys` only at `configreg_test.go:78` without its body, so behaviour 13 stays unconfirmed. `internal/hibernateconfig` holds no `_test.go` file.
