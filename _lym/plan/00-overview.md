---
format: 5
approved: true
language: go
---

# Plan: the raddle recipe

A loom recipe `raddle` runs on a Sketch and files what the operator approves: one tapestry, several tasks, a project or a beam.
A cold agent proposes the split from the Sketch, the board, a bounded read of code and the hub's own measured run costs, the operator approves it in the agent's pane unless auto-approve is on, and Go files it on the board in one write, keeping a raddle store on the Bolt that outlives prune.
Every agent session a run spawns records its tokens and cost under the run's `_lym/usage/`, which `lym report cost` fits per recipe, profile and stored-record flag, and `lym report raddle` compares each proposal's prediction with what its tasks cost.
Cards 1–8 build the usage records and the cost report, cards 9–22 the raddle pieces, card 23 the recipe and its wiring, and cards 24–27 the start guard, the beam hand-off, the raddle report and the docs.

## Card Index

1 — shuttle-usage-split — SessionUsage carries the four token counts apart, read at every session end
2 — codex-usage-reader — Codex reads a session's usage from the rollout's last total_token_usage
3 — shuttle-session-end-hook — a runner hook called once per ended session, stops included
4 — usage-record-writer — shed's one usage writer and the `_lym/usage/` record files
5 — usage-wiring — every run runner records its sessions; status commits carry the usage directory
6 — seed-and-landed-size — Seed records recipe, profile and stored-record flag; Finalize the landed diff size
7 — report-config — the report config module with cost_min_runs
8 — report-cost — `lym report cost`: per-cell fixed and per-100-line cost
9 — raddle-paths-and-keys — the raddle run's paths and the loom keys raddle and raddle_auto_approve
10 — raddle-proposal-spec — the proposal format contract
11 — discussion-part-splice — section bullets and the part-record splice in the decision record's parser
12 — raddleparser — the proposal's sole parser, D6's checks and D8's re-check
13 — board-promote-raddle — a discussed note becomes a task only as a raddle run's entry
14 — raddle-store-slug-rule — the raddle store on the Bolt and the slug rule's refusal of its slugs
15 — raddle-store-replay — the Bolt replayer claims and replays the store
16 — board-raddle-writes — filing, releasing and ending writes for Raddle-File
17 — board-raddle-verb — `lym board raddle <slug>`
18 — record-write-stored — Record-Write seeds a discussed entry's stored record
19 — raddle-checks-and-state — the raddle state, the checks on disk and the raddle gate
20 — raddle-verbs — `lym loom raddle-propose` and `lym loom raddle-approve`
21 — raddle-write-producer — the raddle stencil, seat table and Raddle-Write producer
22 — raddle-file-producer — the Raddle-File producer's seven steps and returns
23 — raddle-recipe — the recipe, its registry rows, the gate and the wiring
24 — batten-raddle-start-guard — batten refuses a raddle start with no record or a reused slug
25 — pawl-raddle-handoff — Beam-Up mails a beam's orch the Sketch and the proposal
26 — report-raddle — `lym report raddle`: predicted against actual cost
27 — raddle-docs — the glossary's raddle row and the overview's recipe lists

## Shared Decisions

### Decision: proposal-json-form

- **Decision:** `proposal.yaml` is written in YAML's JSON-compatible flow form, and `internal/raddleparser` decodes it with `encoding/json`.
- **Rationale:** D5 binds the sole parser to the standard library and `internal/discussionparser`, which holds no YAML decoder; a JSON object is a valid YAML file, so the file stays the one YAML file D5 names.
- **Applies to:** cards 10, 12

### Decision: usage-hook-in-shuttle

- **Decision:** shuttle reports each ended session through a runner hook (`Runner.SetSessionEnd`), and shed's one writer (`shedadapters.RecordSessionUsage`) is installed on every runner a run builds, reading the row from the run's status.
- **Rationale:** no shed package sees every session's end (each row family drives shuttle through its own seam), while every session ends in one runner; the hook keeps shuttle free of shed's vocabulary and the writer in shed's packages (`PATTERN-shed-shared-machinery`).
- **Applies to:** cards 3, 4, 5

### Decision: raddle-returns-and-sessions

- **Decision:** the raddle run's state (`_lym/raddle/state.json`) numbers Raddle-Write's sessions, one per call of the row; a return from Raddle-File or an operator goto into Raddle-Write is a fresh entry, which sets aside the proposal in auto mode and drops the approval in interactive mode, so the seat starts a new session.
- **Rationale:** D7 and D8 key the propose notice, the approval and the return on "the current Raddle-Write session", which the shed row history alone does not number; counting every call keeps the notice in every session that approves.
- **Applies to:** cards 19, 20, 21, 22

### Decision: filing-level

- **Decision:** filed entries take the Sketch's own `project` field as their project level, the project shape's project included.
- **Rationale:** D9 files "at the Sketch's own project level"; `hubgeom.ParentOf` answers the nearest beam, which skips a grouping project, while copying the `project` link keeps every filed entry's parent beam, read through `ParentBeam`, equal to the Sketch's. The decision record's Added after Discussion entry "Filed entries take the Sketch's own project field" records this in place of the `PATTERN-pair-parent` Constraint's `hubgeom` parent read.
- **Applies to:** cards 16, 22, 23

### Decision: pattern-headroom

- **Decision:** the two declared PATTERN lines are prescribed verbatim (cards 4 and 12); `PATTERN.md` stands at 15560 bytes against a 15564-byte headroom line, so the projected overview grows past it; the parent recorded eight `changes` decisions that shorten other entries by 321 bytes, and card 4, the growing card, prescribes those lines; every rule they drop is already stated in its background file, so no rule text moves.
- **Rationale:** the lines are the decision record's text and no card may shorten another entry without a recorded `changes` decision (Step 1's route for a design call after the Discussion).
- **Applies to:** cards 4, 12

## PATTERN changes

- `PATTERN-sole-parsers` — changes: Parsing plan, parts file, discussion, summary, recipe, halt note or raddle proposal, or classifying a plan ref: only its sole parser.
- `PATTERN-run-usage-record` — adds: Recording an agent session's tokens and cost per session under a run's `_lym/usage`: only shed's one usage writer, one file per session; a cost report reads token counts and session costs only from those files, never from a review round's usage record.
- `PATTERN-shed-recipe-registry` — changes: Adding producer, recipe row or block gate: one `Constructor` or `BlockGates` map in `shedrecipe`, bar `Implementer`'s; row values by key, side actions as `Command` rows, off switches per the background.
- `PATTERN-test-speed` — changes: Writing or running a test: via `lym gate test` by gate key; one `tiergate.Require` tier, lowest that fits; it pins behavior no test covers; skips, clocks, `t.Parallel` and `//lym:seam` per the background. (test)
- `PATTERN-stencil-ownership` — changes: Reading prompt or spec: at call time via `stencilstore`, never embedded, bar the orch role, fixed in `contracts/orchrole` with no board copy; an instruction file points at another producer's format contract, never restating it.
- `PATTERN-cwd-resolution` — changes: Resolving a cwd, worktree root or anchor: only `lymcwd`, from the worktree's weft-root `.lym-anchor`; only `fabricengine` writes or git-reads it; engines get absolute paths, never import lymcwd; hub containers and workspace files stay hub-reachable only. (test)
- `PATTERN-durable-vs-ephemeral-state` — changes: Adding file: tracked under `_lym`, never-tracked under `.lym`, same subpath, bar heddle's rendered files and `.heddle` at a warp worktree's anchor; only `lymdirs` names either, bar a `tools/` script's variable block.
- `PATTERN-remote-refs-lock` — changes: Writing remote-tracking refs by fetch, pull or push: under the repository's remote-refs lock via `gitrepo`, waiting while held; the pause gate, writes that take none and judges that hold it per the background. (test)
- `PATTERN-slug-remote-order` — changes: Touching a slug's pair or origin branches: under its teardown lock, shared for a pair push outside a removal, else exclusive, held by a teardown's detached child to its last remote step; an origin weft delete leases the tip read.
- `PATTERN-config-layers` — changes: Loading config: only `configengine` reads, stacks, locates and judges layers against the template; callers use `Load` or `LoadOrTemplate`, bar `runprofile`; only `hubreconcile.Ensure` reconciles on a binary change; only `lym config reconcile` prunes unknown base-layer keys.
