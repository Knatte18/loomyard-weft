```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked every claim the fix added or rewrote against the code at HEAD.

**MEDIUM: D5's call wall time uses the earliest `WaitStart`, which is zero on a build-step record**
- **Changed text:** D5, "A call's wall time is the span from the earliest `WaitStart` to the latest `End` over the records sharing its call id".
- **Summary:** For a call with a build step, the earliest `WaitStart` is the zero time, so the span covers about two thousand years. The median then picks the long cap.
- **Evidence:**
  - `internal/gatecli/build.go:65` writes the build record with `Start`, `End` and `Build: true` but no `WaitStart`.
  - Only `run.go:487` and `build.go:85` set `WaitStart`.
  - `internal/gatecli/report.go:147` already guards `!record.WaitStart.IsZero()`.
  - A call with a carried tier is left out of the median. A build-step call is not.

**MEDIUM: D3's new Bound "at most the long cap (D5) plus the grace" does not hold for a configured role cap above 55m**
- **Changed text:** D3 Bound, "holds its slots until the latest deadline a front wrote, at most the long cap (D5) plus the grace after the front that wrote it".
- **Summary:** A role's own cap can be above the 55m long cap. Then the latest deadline is that cap plus the grace, which breaks the stated bound.
- **Evidence:**
  - D4 validates a cap only as "positive and below the lifetime its role resolves to".
  - A role on the 1h lifetime may set `keep_warm_cap_roles` to 58m, for example.
  - D5 fixes the long cap at 55m.

**LOW: The Constraint calls `runRowCommand` a landing config's row command**
- **Changed text:** Constraints, "such as a landing config's deploy or row command".
- **Summary:** `runRowCommand` runs a Shed recipe row's command for `lym batten run`, not a landing config entry.
- **Evidence:** `internal/battencli/wire.go:834` wires `Run: runRowCommand` with `Rerun: lym batten run &lt;slug&gt;` and a `shedrun.ScratchDir` log path. Only `runDeploy` (`land.go:101`) comes from the landing flow.

**LOW: D6 says D7 keeps a `--wait` gate line "for every card"**
- **Changed text:** D6, "A `--wait` gate line, which D7 keeps for every card".
- **Summary:** Not every card has a gate line. D6's own second bullet and `cardverify.Runner`'s other-line path cover cards whose line is not a gate call.
- **Evidence:**
  - D7 says "a card's `**Verify:**` line keeps its required `--wait`", which applies to a card whose line is a gate call.
  - `internal/cardverify/runner.go` `runLine` slots any line `IsGateCall` rejects.

These claims hold:
- `execution.report`'s record of a cancelled tier, and the absent record for a refused call or one cancelled in its slot wait (`run.go:460-497`).
- Slot locks freed when the holder dies (`gateslot/doc.go:21`).
- `verifyrun.Run`'s group SIGTERM, then SIGKILL.
- A `--wait` gate child exits 128 plus the signal number on a group signal (`gatecli/doc.go:243`, `run.go:616-619`).
- Nil env inherits the strand at both batten sites.
- The claude engine sets neither `ENABLE_PROMPT_CACHING_1H` nor `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`.
- The 300000 sits in the `DefaultAwaitCap` doc comment.

Calls made:
1. A grep of `internal/gatecli` for signal exit codes and `WaitStart` writers, of `internal` for `ENABLE_PROMPT_CACHING_1H`, and of battencli for `runDeploy`/`runRowCommand` callers. My context had neither the gate's signal path, the build record nor the batten callers.
2. A grep for build and carried record construction plus `internal/battencli/wire.go:825-840`, to confirm the build record's missing `WaitStart` and what `runRowCommand` runs.
