```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal — proposals challenged: F1, F2, F4, F5, F6, F8, F9, F10, F11, F12 (F7 was fixed differently, so out of scope).

**MEDIUM — F1: D6 "`MultiLLMProducer` maps a result in which a seat ended the step, a `SeatResult` with `EndedStep` set, to `shedengine.Stuck`"**
Summary: the mapping turns every ended writer into an operator-waiting Stuck, including a writer that never started or died for a reason the run classes as transient and would otherwise retry.
Evidence: `internal/seatengine/engine.go:208` — a required advisor "that fails to start, or whose run later ends without being accepted, ends the step", so a start failure also sets `EndedStep`. `internal/shedtransient/shedtransient.go` `Class` classes `shuttleengine.ErrNotStarted` (`TransientAgentStart`), `proc.IsSpawnAgain`, and `ErrAgentDied` within `spawnRefusalWindow` of an EAGAIN (`TransientHostResources`) as transient — the review itself notes "`ErrAgentDied` can also be classed as transient". Concrete case: a part writer's spawn is refused by the host for EAGAIN; under D6 as fixed the run halts Stuck at `Parts-Plan-Write` instead of being classed transient and retried. A narrower change maps to Stuck only a writer that ended done with a failed gate or Accept error, or was not found at resume with missing or failing outputs, and keeps a start failure or death as an error carrying the writer's start error or `ErrAgentDied`, so `shedtransient` still classifies it. The D6 acceptance ("a writer's exhausted gate or failed Accept leaves the run Stuck") already only covers those cases, so it does not contradict the narrower mapping.

**MEDIUM — F6: D3 refusal "a shared spot that uses an export of a part coming at or after the first part that needs the spot"**
Summary: the "at" half refuses a spot whose cards can validly be placed inside the very part that both feeds and needs it, which D9's placement rule allows.
Evidence: concrete case — part A creates handler H; a registry shared spot registers H (uses A's export); A is also the first part needing the registry (a later card of A uses the registration entry, the spot's export). D7 bars A from targeting the shared-spot path, so this is exactly the split shared spots exist for. D9 places the spot's cards "after the cards that make what the spot uses and before the first card that uses what it creates" — the chair's `insert-card` can put it between A's H-creating card and A's registration-using card. The refusal as written rejects this valid, common split and sends `Parts-Write` to reorder or merge for no reason. Refusing only an export of a part strictly after the first needing part keeps the guarantee; Open risks line 3 would then need to say the guarantee is at part order, with within-part placement still the chair's call.

**LOW — F2: D8 refusal way forward "names `lym loom goto --to Parts-Plan-Write`"**
Summary: the callers this refusal now stops are a Plan-Burler-Fix fixer or Webster after the row; for them the named way forward is a fresh rerun of the row, whose D6 preparation archives every part directory and `join.md` and rotates the plan, discarding the reviewed plan.
Evidence: D6 "moves every part directory and `join.md` into a new `_lym/parts/archive-<stamp>/`" and runs `NewPlanDirRotator`; D9 "A seam found after the shared-spot cards are in is fixed in the joined plan by the chair, not by a rejoin." The only way forward offered to a later agent is a destructive redo; the refusal should first say the joined plan is edited in place past `Parts-Plan-Write`, keeping the goto only as the operator's full redo with what it discards named.

No findings for F4, F5, F8, F9, F10, F11 and F12: no case breaks them.

Tool calls:
- Read `internal/shedtransient/shedtransient.go` lines 1–60 to confirm which agent deaths and start failures the run classes as transient; my context held only the review's line-32 citation.
