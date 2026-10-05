# Support log — lyx-skills-and-stencils

## Interview

The session ran autonomously, so no operator answered; every turn below is a question settled from the board entry, the related notes and the code, then recorded as an auto-pick in the ledger.

1. Scope against the board.
   The entry's operator decisions (2026-10-05 and 2026-10-06) fix the orch stencil, the note, `cycle`/`distill`, the reload order, ly-drive as a stencil and parent escalation.
   `orch-owns-the-loop` keeps `batten start`, the land command and `--continue`; `orch-cycle-hardening` and `loom-bouncer-rounds` are done and supply the compact cycle, batten notices, `parent_notice` and `lyx loom decision`.
2. How lyx loads a skill itself.
   Today three loom stencils ask the agent to load scribe skills, best-effort.
   The operator chose typed slash commands for the orch; generalizing that into a launch-spec field serves every spawn with one mechanism (D1).
3. Where the slash syntax lives.
   `PATTERN-shuttle-provider-seam` and the existing `ClearSessionSequence`/`CompactSessionSequence` put it in `claudeengine` (D1).
4. Which roles get which skills.
   The board leaves this to Discussion; the role constants across `internal/` give the list, and D2 assigns skills by what each role writes.
5. How a multi-line stencil reaches a running session.
   Shuttle's `Send` refuses multi-line text and names the file contract, so the role stencil is rendered to a file and pointed at (D4).
6. Note template name.
   The board said `orch-note.md`; the stencil family uses the `orch-template-` prefix throughout, so D5 uses `orch-template-note.md`.
7. `cycle` and `distill` against `cycle_mode`.
   Explicit verbs pick their mode; automatic triggers keep `cycle_mode` (D6).
   Writing the note first in compact mode gives it the `DEFER` handshake the doc lists as a residual.
8. Paths in.
   Start, adopt, clear and the watcher's own compaction are on the board; auto-compaction is also a path in for the orch, and the watcher already reads compaction boundaries (D7).
9. The driver's other launch modes.
   The orch drives runs through `lyx batten run` and the operator resumes with `lyx loom start`, so the fork and operator-launched modes have no user (D8).
10. Parent escalation without duplicating text in every stencil.
    A shared directive, like the `pattern-directive-*` and `friction-directive-*` families, rendered with the parent name from the origin record (D9).
11. New invariants.
    Both rules bind every future role, so they go into PATTERN with scanning tests (D10).

## Rejected alternatives

- **Keep skill loading in the stencils.** The agent may skip it, which is the failure the board records on 2026-10-05.
- **Preloading skills through a Claude Code agent definition (`--agent` with a `skills:` field).** Unverified for a main session, and it would move role setup into provider configuration the stencils do not own.
- **Inlining skill content into stencils.** Duplicates the skills and goes stale, against `PATTERN-producer-pointer-rule`.
- **Failing a launch on a skill that does not load.** Turns a missing plugin on one machine into a dead run; today's Step 0 is best-effort.
- **Typing the role stencil as text.** `Send` refuses multi-line input.
- **A separate turn for the role pointer and the note pointer.** One line keeps the order and saves a turn.
- **Keeping `cycle` mode-following and adding `distill` only as an alias for compact.** The operator named `cycle` as the clear verb.
- **Compact mode without a note.** The board says both start the same way, and the note is what a fresh `start` resumes from.
- **Putting the driver stencil in the `loom` family.** The driver is recipe-blind, and its pins forbid recipe names.
- **Keeping the orchestrator-fork driver mode.** No caller uses it once the orch spawns runs with batten.
- **Copying the parent paragraph into each role stencil.** Many copies drift; one directive does not.
- **Re-typing skills after auto-compaction in every spawned role.** Needs a watcher per spawned session, a design of its own that nobody asked for here.
- **Language-specific scribe skills per role.** lyx runs against repos in other languages; choosing by repo language is a requirement nobody stated.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): how does lyx load a skill into a spawned session? Typed `/<skill>` turns from a launch-spec skill list, realized in `claudeengine` (D1).
- Q2 (auto-pick, resolved): a skill that does not load? Logged and skipped within a bound, never a hang or a failure (D1).
- Q3 (auto-pick, resolved): which roles get which skills? D2's table: every role that writes into the repo or for the operator gets `scribe:prose`, code-writing and code-reviewing roles add `scribe:code-quality` and `scribe:testing`, discussion adds `scribe:conversation`, the orch adds `ly:board`.
- Q4 (auto-pick, resolved): how is the role stencil delivered? Rendered to a file under `.lyx/orch/`, one pointer line combined with the path-specific instruction (D4).
- Q5 (auto-pick, resolved): note template name? `orch-template-note.md`, following the family prefix (D5).
- Q6 (auto-pick, resolved): does `cycle` still follow `cycle_mode`? No: `cycle` clears, `distill` compacts, automatic triggers follow `cycle_mode` (D6).
- Q7 (auto-pick, resolved): does a compact cycle move `LastHandoff`? Yes, once its note passes the gate (D6).
- Q8 (auto-pick, resolved): does auto-compaction trigger a reload? Yes for the orch, with no note (D7); not for spawned roles (Scope).
- Q9 (auto-pick, resolved): where does the driver stencil live and which modes survive? `contracts/stencils/shed/`, loom-launched only, always parking (D8).
- Q10 (auto-pick, resolved): how does every role learn its parent? A shared directive rendered with the parent's name, with a no-parent variant (D9).
- Q11 (auto-pick, resolved): new PATTERN entries? Two, test-backed (D10).
- Q12 (open, for the hub after landing): delete the file-memory entries the orch stencil absorbs, listed in the decision record's notes.
