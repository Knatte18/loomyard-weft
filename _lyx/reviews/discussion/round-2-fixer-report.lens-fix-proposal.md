```yaml
lens: fix-proposal
outcome: reported
```

I checked the proposals the fixer carried out as written (F3, F4, F5/F6, F9, F10, F11, F12, F13).

- **MEDIUM**, D5's flag-off sentence and the matching acceptance criterion (F6): "so a capture left running is ignored, never compared against".
  This fails if the flag goes off and then back on.
  Turn the flag off while a capture runs: `Drift-Await` records Skipped and Webster builds in the same worktree.
  Turn the flag back on before `Drift-Verify`: the row sees a scope record and a passed capture record, and compares against it.
  That baseline may have been written while Webster's build shared the tree.
  Fix: either `Drift-Verify` refuses a capture with no `awaited_at`, or the record says only that a flag-off skip gives up regression, without claiming the capture is never compared.

- **MEDIUM**, D4's new activity paragraph (F3): `agentActivityFor` reads the capture lock.
  D4 says a released lock is how every reader tells a dead capture, and that a second capture refuses while the lock is held.
  If a reader tests the lock by taking it for a moment, the watchdog's frequent check can collide with a capture starting.
  The capture, or `lyx drift capture`, would then refuse spuriously.
  The record does not say how readers test the lock without holding it, and the new frequent reader makes the collision far more likely.
  `agentActivityFor` is wired once for every loom-family loop (`internal/loomcli/arm.go:334`), so this check runs in darn too.

- **MEDIUM**, D6's `lyx drift waive` bound (F1 as proposed, with D5's defer).
  Under scope `partial`, a broken harness makes the capture fail and `Drift-Await` Stuck.
  That is no decision halt, so the arbiter may run `lyx drift defer` and resume.
  `Drift-Verify` then halts on no passed capture, also no decision halt under `partial`, and the arbiter may run `lyx drift waive` and resume.
  Regression is skipped for the whole run and the parent never rules.
  The record bounds `lyx drift accept` under `partial` explicitly (D7: "the agent's judgement ... is the only bound"), but states no bound for waive.
  Fix: make the no-passed-capture halt a decision halt under every scope, or name the arbiter's waive under `partial` as an accepted bound.

- **MEDIUM**, the rewritten Plan-reviewer Constraint (F11): "a Plan reviewer's build or `lyx gate test` ... runs beside the capture within the hub's slot count".
  Only `lyx gate test` takes a gate slot.
  A reviewer's plain build (`go build`, `dotnet build`) takes none, so the "within the hub's slot count" bound does not hold for a build.

- **LOW**, D3's new re-entry text (F10): "the agent is spawned to write `baseline.md` alone, against the recorded scope".
  No refusal enforces "alone".
  `lyx drift scope` refuses only outside `Drift-Baseline` and on bad flag combinations, so the respawned agent can overwrite the recorded scope.
  Either `lyx drift scope` refuses an existing scope record on re-entry, or the sentence drops "alone".

- **LOW**, D3's refused-capture Stuck (F10): "A capture D4 refuses, because HEAD left the branch point or the tracked tree is dirty, ends the row Stuck naming `lyx drift defer`".
  When only the tree is dirty and HEAD is still the branch point, cleaning the tree and resuming would capture again.
  The halt names only deferral, which gives the regression up.

F5, F12, F13 and F16 (the fixer's pairing of the superseded lines with D5 and Q12) held up against every case I built.

Tool calls: I grepped `internal` for where `agentActivityFor` is wired and for `step_idle_timeout_min`, to see whether the watchdog and the activity reading reach darn; my context did not show where it is wired.
