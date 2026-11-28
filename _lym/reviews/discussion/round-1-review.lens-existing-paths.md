```yaml
lens: existing-paths
outcome: reported
```

**Paths traced, per decision**

- D1/D2 — `loomshed.Seed` empty `CurrentProducer` starting a new tapestry run at its first row (now `Motif`) (`internal/loomshed/seed.go` Seed doc): survives.
- D2 — a tapestry run parked at Record-Seed going back by `lym loom goto --to Discussion-Write`: the engine admits it by list order (`internal/shedengine/goto.go:339-384`), loom's settled-segment guard admits a row with no segment (`internal/loomcli/settledguard.go:42-51`): survives.
- D2 — undiscussed gate's seed write `seedDiscussedRecord` (overwrites unconditionally, `internal/loomcli/discussionrecord.go:71`): reached only on entering `Motif`; Record-Seed writes nothing over it: survives.
- D4 — decision verbs: `recordWritePending` and the way-forward texts (`internal/loomcli/decision.go:86-90`, `:604-606`, `:644-661`): breaks (F1).
- D4 — `boardCopyDue` on a jacquard run: no approval row, so falls to `boardRecordStored`; board copy refreshed (`decision.go:697-705`): survives.
- D4 — `validate.go:78` / `shedrecipe/entries_gate.go:211` citations only when engine is `DiscussionWrite`: RecordSeed treated as Record-Write: survives.
- D4 — `loomrecipe.RecordEngine` on tapestry once RecordSeed joins the record engines: first match in expanded order is Discussion-Write: survives.
- D4 — `PastDiscussionApproval`/`DiscussionApprovalRow` for jacquard: no approval row, replace not held back: survives (as darn today).
- D5 — `lym loom start` refusal and its way forward: breaks (F2).
- D5 — batten's guard judged only for a start with no status file, against the slug's done motif run (`internal/battencli/startguard.go:170-183`, `:530-538`, `cleanup.go:201`): breaks (F3).
- D5 — change of the entry's recipe after start: the run reads its seed (`resolveSeedRecipe`, `sharedbootstrap.go:275`), resume passes: survives as the record states.
- D7 — profiles reconcile on a hub file whose `solid` profile was removed: breaks (F4).
- D7 — raddle's profile-block check on an operator-added profile: breaks (F5).
- D8 — dobby clearing of done motif/arras runs (`battencli/cleanup.go:201`) ahead of a jacquard start: survives as for tapestry today.

**Findings**

F1 — MEDIUM, design, decision-record.md D4
D4 says the decision verbs treat RecordSeed as RecordWrite but is silent on their way-forward texts, which are constants naming `lym loom goto --to Record-Write` (`decisionWayRecordGoto`, `decisionWayUncommittedGoto`).
A jacquard run past Record-Seed with a missing or uncommitted record is told to goto a row jacquard does not hold, which goto refuses ("names no producer").
`recordWritePending` also tests `NamePreflight`/`NameRecordWrite` literally.
Evidence: `internal/loomcli/decision.go:86-90`, `:604-606`, `:626-629`, `:654-661`; `internal/shedengine/goto.go:169-175`.

F2 — MEDIUM, design, decision-record.md D5
`lym loom start`'s refusal way forward is `lym board upsert ... "recipe":"tapestry"`, but a worktree start takes its recipe from the seed batten wrote, not the board (`resolveSeedRecipe`, `seedRecipe`): the upsert leaves the seed at jacquard and the same start refuses again.
With no seed, `lym loom start` resolves to tapestry, so the check only fires with a jacquard seed present — exactly where the named way forward does not work.
Evidence: `internal/loomcli/sharedbootstrap.go:275-280`; `internal/loomcli/decision.go:1865-1871`.

F3 — MEDIUM, design, decision-record.md D5
The primary jacquard case — a motif run then jacquard on the same slug — meets that slug's done motif run.
Batten's start guard judges its stored-record checks only when `!found`, and `ClearsMotifRun` clears a done motif run only for a raddle start ("Promoting a Sketch keeps the slug of the motif run that made it").
A jacquard start finds `found && done`, so D5's refusal is never judged and the start depends on the dobby's `admitsMotifRun` cleanup having run first.
The record does not say whether a jacquard start clears the done motif run as raddle's does, or waits on cleanup.
Evidence: `internal/battencli/startguard.go:170-183`, `:290-298`, `:530-538`; `internal/battencli/cleanup.go:201-203`.

F4 — MEDIUM, design, decision-record.md D7
The reconcile gains `defaults.jacquard: solid` (a key absent from both file and base is gained) even on a file whose `solid` profile the operator removed (held as base-removed, stays removed, `FollowTemplate`).
`runprofile.Load` refuses a default naming a profile holding no block for its recipe; the arras precedent needed a one-time reinterpretation for exactly this.
D7 states no bound or reinterpretation, so such a hub's `profiles.yaml` stops loading.
Evidence: `internal/yamlengine/follow.go:45-52`; `internal/runprofile/doc.go` (the `defaults.arras` lines and "`Load` ... checks ... that each default names a profile holding its recipe's block").

F5 — LOW, design, decision-record.md D7
An operator-added profile holding a tapestry block gains no jacquard block (a unit only the file holds is kept as is).
Raddle's check reads profile blocks from `profiles.yaml`, so a part naming that profile with jacquard is now refused `profile-unknown` where tapestry passed.
D7's "every run profile" covers only the template's.
Evidence: `internal/yamlengine/follow.go:52`; `internal/raddleparser/check.go:92`, `:428-429`.

**Tool calls**
1. grep `boardCopyDue`/`copyRecordToBoard` in decision.go — whether a decision add refreshes the stored record on a run with no approval row; context had only their callers.
2. read `CheckGotoTarget` — how goto admits targets, for D4's tapestry way forward; context had only `Goto`.
3. grep raddle profile-block check and batten's discussed-flag reads — D7's effect on filing.
4. read `gotoAdmitted` — whether admission is by list order or history.
5. grep loomcli goto guard wiring — the recipe guard on loom goto.
6. read drivewiring/`settledGuard` — whether the guard refuses `Discussion-Write`.
