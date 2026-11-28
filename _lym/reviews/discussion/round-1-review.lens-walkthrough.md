```yaml
lens: walkthrough
outcome: reported
```

**Walkthrough list**
- D1 — a jacquard run of a raddle-filed part, start to Plan-Write: purpose met (Preflight → Record-Seed writes the stored record and stored-record log, commits; Plan-Write reads it).
- D2 — fresh tapestry run, undiscussed and discussed entries: purpose met (undiscussed: Motif gate holds, Discussion runs, Discussion-Bouncer's empty on_done → Jacquard, always gate, Preflight, Record-Seed finds both files and writes nothing; discussed: gate seeds and skips to SkipTo=Jacquard, Record-Seed writes nothing).
- D2 — in-flight run parked at Preflight, undiscussed: purpose met (Preflight's on_done is now Record-Seed, which stops it per D4).
- D3 — expanding the new tapestry: purpose met (Motif exit = Jacquard; Jacquard last with empty on_done → inlined with exit "", Friction-Reflect keeps ""; no name or segment clash; engine-block row OnDone=Preflight, SkipTo="" never taken under always; contract path Motif→Discussion-Write→Discussion-Bouncer→Jacquard→Preflight→Record-Seed→…→Webster→…→Finalize→Friction-Reflect).
- D4 — tapestry run stuck at Record-Seed, way forward `goto --to Discussion-Write`: purpose met (`gotoAdmitted` admits every row before Record-Seed in expanded order, `internal/shedengine/goto.go:380`; goto runs Discussion-Write ungated; resume → Record-Seed writes nothing).
- D4 — jacquard run whose record was lost after start, way forward cancel/upsert/release/run: purpose met (tapestry run's Motif gate finds no stored record, Discussion runs).
- D4 — decision verbs on a jacquard run past Record-Seed with an uncommitted record: purpose NOT met (F2).
- D5 — `lym loom start` in a jacquard-seeded worktree with no stored record, then its way forward: purpose NOT met (F1).
- D5 — `lym batten run` of an entry its own motif run discussed, then set to jacquard: purpose NOT met (F3).
- D6 — raddle part named jacquard, filing, start: purpose met (part carries discussed + record; passes batten guard; seeds at Record-Seed).
- D7 — existing hub with an operator-added profile holding a tapestry block: purpose NOT met (F4).
- D8 — jacquard run in `lym report`: purpose met as stated.
- D9 — discussed tapestry run and a jacquard run: purpose met (both record stored_record=true, `internal/loomcli/sharedbootstrap.go:245`; both land in the jacquard/true cell).
- D10, D11 — no mechanism to run; held.

**Findings**

F1 — BLOCKING, design, D5 (`_lym/discussion/decision-record.md` D5 way-forward bullet): the way forward named for `lym loom start`'s refusal cannot clear it.
`lym loom start` takes the recipe from the worktree's own seed — `resolveSeedRecipe(existingSeed, seedFound)` (`internal/loomcli/sharedbootstrap.go:105-109`, `:275-280`) — and a board upsert to `"recipe":"tapestry"` never rewrites that seed, so the next `lym loom start` reads jacquard and refuses again.
The way forward is only effective for batten's no-status start; this breaks D5's own claim and `PATTERN-refusal-way-forward`, which the record cites as a Constraint.

F2 — MEDIUM, design, D4 (D4 "The decision verbs treat RecordSeed as they treat RecordWrite"): on a jacquard run, the RecordWrite path's refusals send the agent to a row jacquard lacks.
Taking the RecordWrite path reaches `checkDarnRecord`/`missingRecordRefusal`, whose ways forward are fixed strings naming `lym loom goto --to Record-Write` (`internal/loomcli/decision.go:88`, `:90`, `:629`, `:661`), and `recordWritePending` checks only Preflight and Record-Write by name (`decision.go:604-605`).
On jacquard `goto --to Record-Write` is refused "names no producer" (`internal/shedengine/goto.go:169-175`).
D4 does not say these texts and the pending test take the seed row's name.

F3 — MEDIUM, design, D5 (D5 and acceptance criterion "an entry with a stored record starts"): a jacquard start of an entry its own motif run discussed finds that run's done record and is refused.
`refuseAtStart` clears a done motif run only for a raddle start (`clearsMotifRun` → `internal/battencli/startguard.go:178`); `found` stays true, D5's added guard (judged only for no status file) is never reached, and `checkRunStatus` returns `doneSlugRefusal` "has already completed" (`internal/battencli/arm.go:533-534`, `:945-949`).
The only automatic clearers of a done motif run are raddle's start and `lym beam create` (`internal/battencli/cleanup.go:124-142`, `internal/beamcli/create.go:211`).
The board entry names "a discussed entry" as a jacquard input; the record is silent on this case.

F4 — MEDIUM, design, D7 (D7 "Each profile of `internal/runprofile/template.yaml`…"): an operator-added profile gains no jacquard block, so raddle refuses jacquard parts naming it.
`FollowTemplate` carries in only template units (`internal/yamlengine/follow.go:41-52`); a profile in neither template nor base is the operator's alone and gets no jacquard block.
Before D6 a part naming that profile under tapestry passed; after D6 the same part under jacquard is refused for a missing block (the check D7's rationale cites).
D7 states no bound or way forward for such profiles.

Tool calls:
1. Read `internal/battencli/arm.go:520-560` — how `batten run` handles a done status (context lacked it), for F3.
2. grep `doneSlugRefusal`/`admitsMotifRun` callers — whether anything clears a done motif run before a jacquard start, for F3.
3. Read `internal/battencli/cleanup.go:120-145` and `ClearMotifRun` callers — who clears done motif runs, for F3.
4. Read `internal/shedengine/goto.go` `CheckGotoTarget` — whether D4's `goto --to Discussion-Write` is admitted.
5. Read `internal/shedengine/goto.go` `gotoAdmitted` — which rows goto admits, for D4.
6. Read `internal/loomcli/sharedbootstrap.go:85-125` — where `lym loom start` reads its recipe, for F1.
