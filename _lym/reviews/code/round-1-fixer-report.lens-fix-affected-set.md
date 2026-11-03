```yaml
lens: fix-affected-set
outcome: reported
```

No findings on the fix itself, but the stale-wording searches turned up three sites the fix left untouched. The first two go with F5–F7 (the same "judged as a plain turn end" premise). The third goes with F4.

**Unchecked by me:** F2 and F3 (test-gap findings, no set premise). F1's set, every reader and writer of `waitingTasks` and `waitingTurnEnd`, was checked and is covered. The searches found writers only in `pollEventsTick`, `expiredTurnEnd` and `recordWaiting`. The `countedTasks` fallback in `endedShells` is intact.

**Findings**

1. **MEDIUM** — F6/F7 premise, `docs/overview.md:423` (not touched by the fix).
   - It still says a turn end waiting only on non-awaited shells is one "which a gated run judges as a plain turn end and an ungated run holds once they pass the shell wait bound".
   - That is false for a gated run with every output file present: it now nudges, waits up to the bound, then counts Done.
   - It is the same stale claim F6 and F7 fixed in `spec.go` and `runlevel.go`.

2. **LOW** — F6 premise, `internal/shuttleengine/doc.go:440`.
   - It says `Spec.AwaitedShellPrefixes` keeps a shell "out of the plain-turn-end judgment".
   - The sibling wording at `spec.go:124` was reworded to "waiting with no nudge and no expiry", so the two now disagree.

3. **LOW** — F4 premise, `internal/shuttleengine/doc.go:430-431`.
   - It says the nudge names "each such shell no earlier line of the wait named" and that "a send that fails is logged and not retried".
   - After F4, a dropped or failed line leaves its shells unnamed, and the next waiting turn end queues the line again.
   - "Not retried" is only true in place.

4. **LOW** — F5 premise, `internal/shuttleengine/completionsignal_enforcement_test.go:189`.
   - The comment says the third site "judges a gated waiting turn end on non-awaited shells as a plain turn end, Done only after consulting the files".
   - With every file present, the gated branch now returns through `expiredTurnEnd`, so "plain turn end" no longer describes it.
   - The files check still holds, since `expiredTurnEnd` requires `filesExist` for a gated run.

**Checked and fine**
- `masterAwaitedShellPrefixes` is only consumed at `runlevel.go:779`.
- `websterengine/doc.go:640` already reads correctly.
- `config.go:38` and `waitmark.go` match the new behaviour.
- The remaining "plain turn end" hits (`doc.go:429`, `claudeengine/doc.go:312`, corpus `case.yaml` lines 12 and 43) all concern the output-file-missing case. The review rejected the `case.yaml` wording.

**Tool calls**
1. `grep` for "judged at once" and "plain turn end" wording across `.go`, `.md` and `.yaml`: F5–F7 are stale-wording set findings, and the fix edited only the three named sites.
2. `grep` for `waitingTurnEnd`, `shellNudgeIDs`, `nudgedShells` and `.shellNudge` in non-test code: F1 and F4 are set findings over their readers and writers, which my context lacked.
3. `grep` for `AwaitedShellPrefixes` and "non-awaited" outside `wait.go`: F6 and F7 are set findings over the docs that describe awaited-shell behaviour.
4. `sed` over `internal/shuttleengine/doc.go:426-443`, `completionsignal_enforcement_test.go:186-191` and `git diff --stat` for `doc.go` and `overview.md`: to read the surrounding wording of the sites calls 1 and 3 surfaced, and to confirm the fix left them unedited.
