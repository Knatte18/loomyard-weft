```yaml
lens: board-scope
outcome: reported
```

Task: board-scope lens review of the raddle-recipe decision record and support log.

**Findings**

1. MEDIUM — D3 against peer `pawl-orch-landing` and the pawl recipe.
   - Summary: D3 makes every Record-Write row seed a discussed entry's stored record. Pawl runs Record-Write on the beam's project entry, and D9 turns the Sketch into that project with its record and `discussed` flag kept.
   - Consequence: a raddle-made beam's pawl run would seed the whole Sketch as the beam's project record, which `PATTERN-project-record` treats as the beam's scope. The record never says so.
   - Evidence: the peer's D1 keeps Record-Write in pawl (Preflight, Record-Write, Beam-Up, …); D9 says the "entry becomes a project of the same slug, keeping its record".
   - Class: design. State whether pawl's Record-Write takes the Sketch as the project record, or exempt it.

2. MEDIUM — D11 against peer `pawl-orch-landing` D11.
   - Summary: the peer gives Beam-Up a re-invoke interrupt policy as an idempotent row that spawns no agent. Raddle's D11 promises the hand-off only "at the first start" and none at a later start, but names no durable marker that tells a re-invoked or re-stepped Beam-Up the mail was already sent.
   - Evidence: the peer record's D11 ("Beam-Await, Beam-Up and Beam-Down get the re-invoke interrupt policy … idempotent"); raddle's D11 acceptance line "a later start sends none".
   - Class: design. Name the record that marks the hand-off sent.

3. MEDIUM — D2 against the entry's "Only this run turns a Sketch into tasks".
   - Summary: D2's refusal names "dropping the label for tapestry" as a way forward. That gives the operator a path that turns a Sketch into a tapestry task without raddle, which the entry rules out.
   - Note: note `discussed-task-split` lists tapestry from Plan as one raddle outcome (way 1 under triage), not a bypass.
   - Class: scope. Either drop that way forward, or raise it with the parent as a `decision` if an existing hand route must stay.

4. LOW — D2 against done task `sketch-board-type`.
   - Summary: that task defines a Sketch as a note carrying the `sketch` label ("A Sketch is not a task and never runs"). D2 puts the `sketch` label on a task for the length of a raddle run, so `IsSketch` reads false while the label stays.
   - Not a contradiction of landed behaviour, but the record should say that a task carrying the label is not a Sketch to the board's readers, such as the README's Sketch heading and `IsSketch`.

**Goal points of the entry**

- One tapestry from Plan, several darns, or a mix: met (D5, D9).
- Order as `depends_on`: met (D5, D6).
- Project or beam: met (D9).
- Only this run turns a Sketch into tasks: partly met, see finding 3.
- Own cold Opus role, tuned apart: met (D4).
- Bounded reading, designs nothing: met (D4).
- Per-part decisions, packages, contract, deps, risks, criteria, recipe and profile: met (D5).
- No two parallel parts change one package: met (D6).
- May answer "one tapestry": met (D5 shape `tapestry`).
- Measured fixed and per-size cost from the hub's own records, not prompt figures: met (D12, D13, D4).
- Expected cost against the next-best split: met (D5, D6).
- Choices checked after landing: met (D14).
- Asks nothing while working: met (D4, D7).
- Interactive by default, revised until approval: met (D7).
- Auto-approve flag, off by default: met (D7).
- Files only on approval: met (D9).
- Board re-check at approval: met (D8).
- Project or beam hands the Sketch and proposal to the beam's orch: met for a beam (D11). A plain project has no orch, and D9 notifies the parent orch instead.
- Runs through batten like `motif`, own worktree, no code landed, pane and notices: met (D1, D7, D9).
- Glossary gains `raddle`: met (D15).

**Entries, peers and commits checked**

- `sketch-board-type`: finding 4, otherwise no conflict.
- `discussed-task-split`: no conflict. D3 closes its open darn line, which no other entry owns.
- `plan-after-dependency`, `beam-orch-root-project`, `reparent-running-task`, `parts-plan-and-build`: no conflict.
- Peer `pawl-orch-landing`: findings 1 and 2.
- Peers `branch-name-recorded` and `readme-refresh`: no conflict.
- Parent commits: `lym loom peers` lists none.
- Decision notes: the board shows no `Decision for raddle-recipe:` note, so none was read and none was out of reach.

**Tool calls**

- One Bash call listed the hub directory and grepped the `pawl-orch-landing` decision record for its decisions and its Beam-Up and Record-Write rows. My context held that peer's title and status only, not its record.
