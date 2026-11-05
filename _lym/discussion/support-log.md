## Interview

Autonomous run: no operator answered, and every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

- Board: task `raddle-recipe` read; touching entries `sketch-board-type` (done), `discussed-task-split`, `plan-after-dependency`, `beam-orch-root-project`, `reparent-running-task`, `parts-plan-and-build`. No `Decision for raddle-recipe:` note on the board.
- Peers (`lym loom peers`): `pawl-orch-landing` reworks pawl rows and keeps BeamUp and Record-Write; `branch-name-recorded` and `readme-refresh` do not touch this design.
- Five explore subagents answered: motif's recipe and finish; Discussion-Write's interactive mode; the cost data lym holds; board filing paths and locks; orch notices, the glossary and Command rows.
- Findings that shaped the design:
  - A loom recipe needs a record row (`DiscussionWrite` or `RecordWrite`); a recipe with no Finalize row ends its entry as a Sketch today.
  - Interactive Discussion has no operator-approve verb; sampler ends on `lym loom sampler-done`, which its agent runs on the operator's word.
  - `lym report timing` gives time only; no lym code records a run's token cost; the token audit in `docs/research/token-audit-2026-10-22/` was computed by hand from transcripts in this repository only.
  - The board refuses a run agent that creates a task by its strand name; Go writing the board directly carries none.
  - `orchcli.NotifyOrchAt` posts nothing to a worktree with no orch strand, and a beam's orch is not up at `lym beam create`.

### Turn 2 — design picks

All questions of the ledger settled as auto-picks; the first record stated D1 to D15.

### Turn 3 — lens review

Six forks reviewed the first record; every one returned a report. Findings, merged across lenses:

- walkthrough 1, interactions F1 (BLOCKING): `lym beam create` on the Sketch's slug inside the live raddle run finds the run live and starts no pawl run. Fixed: every filed slug is new (D5), so the beam's project has no run of its own yet; the done-run clearing of the first D10 is dropped.
- existing-paths 1, interactions F7 (BLOCKING): the run's later status writes put a run status back on a re-typed entry. Fixed: the Sketch entry is marked done and filed entries are new (D9), and status writes skip a done entry.
- bound-breaker 1, existing-paths 2, walkthrough 2 (BLOCKING): Land prunes the done Sketch and deletes its record, and the stored proposal with it. Fixed: the raddle store outlives prune and keeps the proposals and a copy of the Sketch record (D10); D14 reads the store.
- interactions F2 (BLOCKING): the proposal was stored after the beam started, and a retry skipped later steps. Fixed: D9 orders store, records, board write, beam, notice, each skipped when done.
- walkthrough 3, interactions F3 (MEDIUM): nothing marked the cold proposal in interactive mode, and a return overwrote it. Fixed: `lym loom raddle-propose` stores the cold proposal once per run and posts the waiting notice (D7).
- bound-breaker 2, walkthrough 4, bound-breaker 5, existing-paths LOW, interactions F8 (MEDIUM/LOW): the re-check's baseline and its handling of landed entries. Fixed: measured from the latest passing check, only tasks and projects not in `board_checked`, done or gone entries named but passing, dependencies on gone entries dropped (D8). Status changes of a weighed entry other than done are not compared: raddle designs no code, and the filed parts plan against fresh code.
- bound-breaker 3 (MEDIUM): the agent can run the approval verb itself. Fixed: D7's bound states it, as for `lym loom sampler-done`.
- bound-breaker 4, existing-paths 3 (MEDIUM): promotion cannot set a recipe. Fixed: `lym board promote` gains a recipe set in the same write, and the entry is released (D2).
- board-scope 3 (MEDIUM): "drop the label for tapestry" bypassed raddle. Fixed: removed; only recipe `raddle` may make a Sketch a task (D2).
- board-scope 4, interactions F9 (LOW): `sketch` on a task or project. Fixed: D2 says such a task is no Sketch to `IsSketch` and the README; D9 files the project without the label.
- board-scope 1, interactions F6 (MEDIUM): pawl's Record-Write would seed the Sketch as the beam's record. Fixed: D3 states it as intended.
- board-scope 2, walkthrough 6 (MEDIUM/LOW): no marker for the hand-off sent. Fixed: BeamUp records it in its pawl run's state (D11); Open risks names the peer's re-invoke policy. Review round 3 replaced that risk with a Constraints Fact.
- interactions F4 (MEDIUM): `Added after Discussion` entries have no id. Fixed: every part record carries all of them (D9).
- interactions F5 (MEDIUM): a part typed `sketch`. Fixed: D6 refuses it.
- interactions F10, walkthrough 5 (LOW): the tapestry shape reused the Sketch slug. Fixed by new slugs (D5).
- existing-paths 4 and LOW (MEDIUM/LOW): usage only on done sessions, and the file key. Fixed: read at every session end the provider can read, one file per session id, overwritten on resume (D12).
- leanness 1, 2, 4, 5: Fixed: Auto-mode assumptions and Scope "In" shortened to pointers, restating PATTERN bullets cut, the audit history moved here.
- leanness 3: Rejected as worded; the `pawl-orch-landing` risk now names the re-invoke policy D11's marker must hold under, which is no text-only overlap.
- leanness 6: Fixed: D12 opens with why recording cost is in scope.
- leanness 7: Fixed: D3 no longer repeats the Open risks line.
- leanness 8: Rejected: the D4 stencil rule is a requirement the plan writer needs, and its `Review:` criterion checks it.

## Rejected alternatives

- A Raddle-Write row that writes a fresh decision record instead of Record-Write: the Sketch already is the record, and the loom contract accepts Record-Write.
- A review ring (Bouncer and Burler) on the proposal: the operator's approval reviews it, and auto-approve is meant to run alone once tuned.
- A proposal in markdown: Go files it field by field, and a YAML structure has one clear parser.
- A dedicated `lym raddle <slug>` verb to start the run: promotion with a recipe already releases a task to dobby.
- Filing parts as drafts: the operator's approval is the release, and drafts would ask for a second one. The `beam` shape's parts are filed unreleased only until Raddle-File's step 5 releases them, with no second approval.
- Re-typing the Sketch entry itself into the tapestry task or the project: its slug carries the live raddle run, so a beam could not start and the run's status writes would land on the filed entry.
- Keeping the Sketch entry open as a raddled Sketch: the raddle store holds what it would keep, and the board stays free of an entry nothing runs.
- Running `lym beam create` from dobby or the parent orch after teardown: a new project slug lets Raddle-File create the beam itself.
- Feeding the agent the hand-run token audit's figures: they belong to this repository's history and a fixed date, and the entry asks for the hub's own measured costs.
- Mailing the beam's orch from Raddle-File: no orch strand exists then, so the notice would be dropped.
- One usage file per run appended by every session: concurrent gate fixers and forks would need a lock; one file per session needs none.
- Leaving darn's handling of a discussed entry to a later task: darn parts are filed discussed and must build from their part record.
- A per-model price table under the report key `prices`: it duplicates the engine's pricer outside the provider seam, and a session total loses the one-hour cache-write rate and per-message price cards.
- Marking the run's entry done in step 3's board write: origin then reads the live run as ended, so a resume is refused, the hub reconcile tears its pair down and a weft replay sets aside its later record commits. The final done write of D9 step 7 works because step 3 now leaves its own marker, the approved proposal's hash in the raddle store.
- Every filed slug existing as step 3's done marker: a released part can land and be pruned before a retry.
- Keying the raddle store by an id never reused: `lym board raddle` and D11 look up by slug and would need an index; refusing a reused slug keeps the slug the key.
- Moving the round usage record's token reading onto the one usage writer: it changes the review machinery for a reading no cost report needs.
- Setting the beam flag in step 3's write instead of filing the beam's parts unreleased: `lym beam create` writes the flag with the beam's owner and color in one owner write (`internal/beamcli/create.go`), and a flag set before it puts a beam with no owner on the board.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): How does a raddle run satisfy the loom contract? Record-Write as the record row, seeding the stored record (D1, D3).
- Q2 (auto-pick): How does a Sketch start a raddle run? `lym board promote` with recipe `raddle` in one write, released (D2).
- Q3 (auto-pick): Model and effort of the agent: loom key `raddle`, default `opus[high]`; the entry asks for Opus and a tuning apart from Discussion-Write (D4).
- Q4 (auto-pick): Proposal format: one YAML file, a spec and a sole parser (D5).
- Q5 (auto-pick): Which part recipes may a proposal name? `tapestry` and `darn`, the two the entry names (D5, D6).
- Q6 (auto-pick): How does the run learn the operator approved? The agent runs `lym loom raddle-approve` on the operator's word, as with `lym loom sampler-done`; the hash binds the approval to the file (D7).
- Q7 (auto-pick): What does "checked against the board again" check? The weighed entries' stamps, new tasks or projects sharing a module label, free slugs, at approval and at filing (D8).
- Q8 (auto-pick, revised in review round 2): What happens to the Sketch entry? Marked done in Raddle-File's last step and pruned at Land; the raddle store keeps its record and proposals (D9, D10).
- Q9 (auto-pick): Filed tasks released or draft? Released (D9).
- Q10 (auto-pick): How do parts carry their slice of the Sketch? A part record spliced by `internal/discussionparser`, stored and flagged discussed (D9).
- Q11 (auto-pick, revised in review round 3): How does the beam's orch receive the Sketch and proposal? BeamUp mails it, then records a sent-marker; an interruption between the two may repeat the mail (D11).
- Q12 (auto-pick): Where do measured costs come from? Per-session usage records, landed diff size, and `lym report cost`; the hub has none today (D12, D13).
- Q13 (auto-pick): How are choices checked after landing? `lym report raddle`, predicted against actual, and cold against approved proposal (D14).
- Q14 (auto-pick, revised in review round 1): Prices for dollars: each session priced per message through the engine's `MessagePricer` and recorded with its usage, not a config price table (D12, D13).
- Q15 (auto-pick, lens review): Filed slugs: always new, so no filed entry collides with the Sketch's runs (D5).
- Q16 (auto-pick, lens review, revised in review round 1): How does interactive mode mark the cold proposal? `lym loom raddle-propose` stores it on the run's first pass; the waiting notice goes out on the first pass of each Raddle-Write session (D7).
- Q17 (auto-pick, review round 1): What stops a Sketch whose label was dropped from promoting with another recipe? The promote rule keys on the `discussed` flag and stored record, not the label (D2).
- Q18 (auto-pick, review round 1): How is a removed entry told from a landed one? It is not: the board keeps no removed entry's status and already reads a missing slug as pruned done, so D8 states the drop as what its guard lets through (D8).
- Q19 (auto-pick, review round 1, revised in review round 2): Where does a halt after filing show? On the run's entry like any halt, since the entry stays open until step 7; a retry finds step 3 done by the raddle store's hash (D9).
- Q20 (auto-pick, review round 2): When are a beam's parts released? In a step after `lym beam create`, so no parent-level dobby starts them outside the beam (D9).
- Q21 (auto-pick, review round 2, revised in review round 3): How does the raddle store avoid a reused slug? It stays keyed by slug; batten refuses a Sketch whose slug the store holds for another Sketch, filed slugs must be free in the store, and the board's new-slug rule refuses a store slug on every write that creates, renames or promotes an entry (D2, D5, D10).
- Q22 (auto-pick, review round 2): Does the round usage record keep its token reading? Yes, as a round summary; the PATTERN binds per-session tokens and cost under `_lym/usage` and cost reports read only those (D12).
- Q23 (auto-pick, review round 3): Does BeamUp mail the hand-off or record the marker first? It mails first, so an interruption repeats the hand-off rather than losing it (D11).
