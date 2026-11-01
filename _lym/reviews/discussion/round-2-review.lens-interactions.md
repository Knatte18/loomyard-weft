```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked pairs of the record's decisions that act on the same state, and checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 entry gates × D6 Stuck halt and recovery (goto `Drift-Baseline` with the flag off, then `Plan-Write` rotates the joined plan out): held.
- D1 flag flipped mid-run × D8's current-row check: held.
- D1 goto onto `Parts-Plan-Write` × the settled-segment guard: held. Tapestry's `Plan-Write` row has no recipe segment, so the new rows are admitted as long as they carry none. D4's and D6's "segment plan" is the seat segment.
- D1 settled-verdict naming × `segmentWriter`: held. `segmentWriter` always returns `Plan-Write`, because its `on_done` stays `Plan-Bouncer`. D1 states the history-based override.
- D3 rule against `archive-` part names × D4 and D6 archive preparations: held.
- D4 fresh-spawn archive × D6 fresh-spawn archive, both on `_lym/parts/`: held.
- D4 commit "on Done" × the decorator it copies, `NewPlanWrite`, and D6's Done-and-Stuck commit: broken, F5.
- D5 resume keeping finished parts × D6 Bound, where a lost unfinished writer restarts every part: held.
- D5 resume × D9 fix loop, given that Probe removes every done writer's kept pane: broken, F3.
- D5 resume counting a writer finished × D9 fix requested of that writer: broken, F4.
- D5 required writer × D6 Stuck mapping × `join` and `mapOutcome`: held as stated. D6 names the `EndedStep` mapping that `mapOutcome` lacks today.
- D5 died writer, kept as `ErrAgentDied` × D13 reinvoke and retry × D6 fresh path: held. The chair stopped by `join` means a fresh run, so every part is rewritten, as D6 Bound says.
- D5 hold deferral × `advisorWork.outstanding`: held. D5 states the change.
- D6 chair inputs listing every part overview × `Table.Validate` and the fresh archive: held. Inputs need not exist at spawn.
- D6 writer gate findings key × D3 part names: held.
- D7 `--part` × D3 shared spots inside an area × D9 chair-created shared-spot exports: held.
- D8 join-once × D9 shared-spot cards × D11 writer limits: broken, F1.
- D8 current-row bound × D6 Stuck halting at `Parts-Plan-Write`: broken, F2.
- D8 refusal wording × D5 and D9 lost-writer path: broken, F6.
- D9 chair fixes a lost writer's part × D12 and the board's "chair reads cards, not code": broken, F3.
- D13 reinvoke policy × D4 single-seat resume: held.
- Acceptance criteria D1 to D13 against their decisions: held, except D4's, which inherits F5.
- Constraints against the decisions they touch: held.

**Findings**

**F1. MEDIUM, design, D8 Bound × D11.**
D8's bound leaves the chair as the only agent kept from re-running `join-parts` inside the row. Part writers are live during `Parts-Plan-Write` and pass the current-row check, yet D11 forbids them only from editing the plan directory, not from running `join-parts`. D8 even sends each refusal's way forward to "its writer". A writer that runs it after the chair inserted shared-spot cards discards those cards silently.
- Evidence: `_lym/discussion/decision-record.md:176-179`, `:216-225`.

**F2. MEDIUM, design, D8 Bound × D6 and D1 Stuck halt.**
A Stuck at `Parts-Plan-Write` persists the run Blocked with that row still current. So `join-parts` stays admitted on a halted run, for the operator, the orch or a tackler. It overwrites the joined plan D6 just committed and leaves an uncommitted plan directory. D8's bound "only while `Parts-Plan-Write` is the current row" does not exclude the halted state, and its way forward assumes a goto is the only redo path.
- Evidence: `internal/shedengine/run.go:416-419` (persist keeps `st.CurrentProducer` with `StateBlocked`), `_lym/discussion/decision-record.md:141-146`, `:178-179`.

**F3. MEDIUM, design, D5 resume × D9 fallback × D12.**
Probe removes every done writer whose pane was kept, so after any reinvoke under D13 no finished writer answers. D9's "chair fixes that part itself" then becomes the common case after every interrupt. D12 and the board limit the chair to reading cards, not code, beyond what shared spots need, so the chair cannot do the reading a behaviour-against-contract fix may need. No decision bounds or reconciles this.
- Evidence: `internal/seatengine/resume.go:22-23`, `_lym/discussion/decision-record.md:118`, `:188`, `:229`, `:242`.

**F4. LOW, consistency, D5 resume × D9 fix loop.**
Take a done writer that was mid-fix at the interrupt. Its outputs exist, and its Accept can pass on the half-applied edit. D5 then counts it finished and tells the chair "its part stands", which contradicts the fix the chair asked for. D9 does not say the chair must treat a pending fix request as open.
- Evidence: `_lym/discussion/decision-record.md:118`, `:188`, `:225`.

**F5. LOW, consistency, D4 commit × D6 and `NewPlanWrite`.**
D4 says its decorator commits `_lym/parts/` "on Done", and its acceptance tests only Done. The decorator it copies, `NewPlanWrite`, commits on any non-empty pointer, gate-failed Stuck included. D6 states both outcomes for its own row. So D4 is ambiguous about whether a Stuck from an exhausted `parts` gate halts with a dirty tree.
- Evidence: `internal/loomshed/planwrite.go:58-63`, `_lym/discussion/decision-record.md:93`, `:141`, `:280`.

**F6. LOW, consistency, D8 refusal way forward × D5 and D9.**
Each `join-parts` refusal names "that its writer fixes it". After a resume that writer no longer answers, and D9 gives the fix to the chair, so the refusal's way forward is wrong on that path.
- Evidence: `_lym/discussion/decision-record.md:176`, `:118`, `:188`.

**Tool calls**
1. Tapestry recipe rows and `segmentWriter`: checked whether the new rows' goto passes the settled-segment guard. My context lacked the recipe's segment fields.
2. `Inputs` existence checks in the seat and shuttle engines: checked whether the chair's part-overview inputs must exist at spawn. My context lacked the input handling.
3. `Inputs` uses in the seat and shuttle engines: found the lead-inputs rule at `internal/seatengine/table.go:232-243`, after call 2 came back empty.
