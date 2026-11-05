```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set for the diff `4c71d798..HEAD`. For each change, the set found outside the touched lines, and whether it is covered.

**Per change**

1. **`wakeHookStrand` bell-hold rule (`deliver.go:287-293`).**
   - Set found:
     - Comment text: `doc.go:174-185`, the `bellGrace` and `ringAnswerWindow` constants (`deliver.go:23-26`), and the `wakeHookStrand` doc.
     - Other readers of the behavior: `orchengine/pause.go:235`, `orchengine/doc.go:384,421` and `orchrole/orch-role.md`.
     - Tests: `TestScheduler_HookWake` (`deliver_test.go:1708-1712` header, plus rows at 2196-2226 and 2361-2430) and `flags_test.go:181`.
   - The orch files only say mail reaches an idle orch through hooks or the bell. That stays true, and `orch-role.md` has no timing claim, so they are covered.
   - The `ringAnswerWindow` and `wakeHookStrand` comments and the `doc.go` text match the new code. They are covered, apart from finding 1.
   - Finding 1 is a stale test header.
   - I did not re-check the old rows at 2196-2226, 2361-2386 and `flags_test.go:181` against the new `woken` hold. That is the code-tests lens's work.
2. **`TurnEndedAt` and `NoTurn` consumers.** The diff reads them in a new way but does not change their shape. The definitions are `seam.go:75`, `shuttleengine/engine.go:482`, `creelcli/delivery.go:337` and `creelfake.go:222-236`. They are covered, and `doc.go:11` still says "zero when the pane decided".
3. **`doc.go` Bound line.** Its claim matches the code: a pane-decided reading has a zero `TurnEndedAt`, so `woken` stays true until the window ends. It is covered.
4. **PATTERN files and declared changes.** The diff touches no `PATTERN.md` or `pattern/` file. The directive text I hold declares no PATTERN changes, so there is nothing to land. Covered.
5. **Stencils and specs.** A grep for the bell line and the timing found no stencil or spec that states the old timing. Covered.
6. **Commit messages.** Neither commit carries a `Pattern-Departure:` trailer. The first commit's claims match the diff. Covered.

**Findings**

1. **LOW, `consistency`.** The `TestScheduler_HookWake` doc comment at `deliver_test.go:1710` says "The bell follows a set no waiter reaches, once per set, after the grace of 5 seconds or after a ring unanswered for 30." The diff adds a third case: within the 30 seconds the bell is also held while no turn end has been read after the ring. The comment does not mention it, so it misdescribes the test the diff extends.

**Tool calls**
1. grep over the repo for the window and grace constants and the "30-second" and "5 seconds" wording. It looked for docs or stencils outside `creelengine` that restate the timing; my context held only `creelengine` files.
2. grep for the bell line and `BellLine` mentions, plus a `git diff --stat` on `PATTERN.md` and `pattern/`. It looked for other documents describing the bell and for any PATTERN change.
3. grep on the orch files for "bell", plus a read of the `deliver_test.go` header. It checked whether other callers state bell timing, and the exact text of the test comment.
