# Support log — codex-engine

## Interview

### Turn 1 — the task and the board

Autonomous run, no advisors.
Read the board entry, the board list, the notes `lookup-agents-through-shuttle` and `claude-account-weights`, and `lym loom peers`.
The entry's goal of 2026-11-03 replaces the Codex-fan-as-replacement goal: Codex is an extra reviewer seat, an assistant, beside the Claude reviewer, which keeps the verdict; the seat runs through a real `codexengine` built by the entry's Steps 1 to 3, Step 4 (forks, orch parity) deferred.
No `Decision for codex-engine:` note exists.
No peer touches shuttle, seats or burler; `darn-writer-runs-no-gate` touches the stencils registry file, a text-only overlap.

### Turn 2 — exploration

Read the spike, hook-delivery and feasibility reports, the provider seam, the engine registry, the seat engine, the Burler review half, the reviewer and seat stencils, heddle's targets and codex-file render, and the model-spec registry.
Four Sonnet lookups covered creel delivery to a strand without creel hooks, the stencil-engine audit, the shuttle seams a new engine meets, and the Burler review wiring.
Facts that shaped the design:
- An advisor talks to its chair through creel; a Codex strand has no creel hooks, so mail would be typed into its pane, and its `workspace-write` sandbox blocks the hub store a reply writes.
- seatengine sends no line when an advisor finishes, and a chair inside `lym shuttle await` is busy, so a typed ending line cannot land during the wait.
- Every Burler seat loads skills, so a Codex seat needs a SkillLoader or shuttle refuses it.
- A `$` typed into Codex opens a popup that takes the next Enter.
- No stencil renderer chooses by engine today.
- The repository has no `CLAUDE.md`; heddle renders the instructions, and Codex loads one instruction file per directory.
- `_lym` is a junction out of the worktree, so a Codex report written there falls outside the sandbox's writable workspace.
- Codex 0.164.0 is installed; its binary holds `developer_instructions` and `writable_roots`, neither tested.
- go.mod holds no TOML library.

### Turn 3 — the design, self-decided

Every question below was decided by auto-pick; see the question ledger.
The assistant seat kind (D1) answers the creel and wait facts; the report under `.lym` with a Go copy beside the review (D2) answers the sandbox fact; skills by path (D9) answers the popup and untested discovery; the prompt-file notice (D6) avoids an untested setting; engine blocks (D11) answer the audit; the codex-file fallback (D12) answers the one-file-per-directory fact; the rendered `config.toml` (D5) avoids a TOML dependency.

### Turn 4 — lens review

Six forks (interactions, existing-paths, bound-breaker, walkthrough, board-scope, leanness) each returned a report, saved beside this log.
After the fork pass the writer read `codex --help` and `codex resume --help` (0.164.0): the positional argument is documented as "Optional user prompt to start the session".
Findings, duplicates merged:

- Assistant report and marker in the half's temp directory, unreachable on probe and resume (existing-paths 1, bound-breaker 1, interactions F1, BLOCKING): fixed; D1 and D2 make them stable per round, derived from the review path and round.
- Empty resume line falls back to the launch line in reed, relaunching a second review (existing-paths 2, BLOCKING): fixed; D6's resume line starts no Codex and exits the shell.
- Chair's await returns on a first draft the gate may still rewrite (interactions F2, walkthrough 2): fixed; D1 writes one marker on every ending, done only after the seat gate, and the chair awaits the marker alone.
- Partial report copied after a stop (bound-breaker 7, interactions F8): fixed; the copy runs only on a done marker, on fresh and resumed ends.
- Trust keyed on the worktree root while Codex keys a linked worktree on the main checkout, and the render drops Codex's entry (bound-breaker 3, interactions F4, walkthrough 1, existing-paths 5): fixed; D5 records the main checkout root too, read through `internal/dotgit`.
- D5 claims full compliance with the operator's read-modify-write decision (board-scope 4): fixed; D5 names the render as a departure, with its bound.
- D3 empty entries refused by the model-spec list check (bound-breaker 2, walkthrough 4): fixed; the assist keys take a list grammar of their own.
- Assistant choice not recorded per round, so a mid-round config edit reshapes the probed table (existing-paths 3): fixed; D2 records it beside the fans.
- Copied report outside the archive-on-exit set (existing-paths 4): fixed in D2.
- codex-file re-render misses `CLAUDE.md` (existing-paths 6): fixed in D12.
- Engine-block resolver not named for the marker check (existing-paths 7): fixed in D11.
- A missing engine block for the assistant refuses the whole round, so Codex gates (interactions F5): fixed; D11 makes it an assistant not-start.
- `PATTERN-engine-blocks` text broader than the work (interactions F6): fixed; narrowed to stencils more than one engine renders.
- Assistant fan tied to the chair's fan, no solo fallback (board-scope 3): fixed; D3 adds a fan key per segment, solo by default.
- Fan of Codex subagents before Step 4 (board-scope 2, decision class): kept as an opt-in with its bound and raised under Open risks for the parent; the default runs solo.
- Codex subagents run without shell guards (interactions F3): fixed; stated as D2's bound and in D10.
- Block v2 reading directions and the yardstick bullet missing (board-scope 1): fixed; D2 carries them as engine blocks.
- A stray Codex edit can reach the fixer's commit (bound-breaker 4): fixed; stated in D6's bound, the same exposure a Claude reviewer has.
- Codex exit behind a live pane shell reads as a timeout (bound-breaker 5): fixed; D6 exits the pane's shell after Codex, so the pane dies.
- Notice and passages as turn text: compaction and the skill-load turn precede them (bound-breaker 6, interactions F9): fixed as D6's bound; the hooks deny regardless.
- Quiet hold of an assistant unstated (interactions F7): fixed in D1 and the acceptance criteria.
- Skill classification by absolute path misses relative reads (bound-breaker 8): fixed; D9 matches the `skills/<name>/SKILL.md` tail and states the retry cost.
- Positional first-turn prompt untested (walkthrough 3): fixed; D6 cites the 0.164.0 help text and states that a turn that never starts fails the start, which shuttle retries.
- Await-forks in a fanned assistant on an engine without the reader (walkthrough 5): fixed; D11 lists the in-turn wait among the engine blocks.
- Auto-mode assumptions restating decisions (leanness 1): fixed; trimmed to the two no decision states.
- `PATTERN-fs-links` listed as holding while changed (leanness 2): fixed.
- Effort constraint repeating D4 (leanness 3): fixed; removed.
- D9's Step 2 line repeating the open risk (leanness 4): fixed; kept in Open risks only.
- D2 and D6 bounds repeating (leanness 5): fixed; the write bound lives in D6.
- Open risk repeating D5's bound (leanness 6): fixed.
- Plan-writer notes naming tests (leanness 7): rejected; the lens itself asked for no action.
- `ReviewResult` line in D2 (leanness 8): fixed; cut.

## Rejected alternatives

- The Codex reviewer as an ordinary advisor seat: needs creel delivery into Codex and a reply out of its sandbox, neither built nor tested.
- A done line typed to the chair when the assistant finishes: the chair is busy inside its await, and ending its turn to wait risks a cold prompt cache.
- The assistant as a required advisor: Codex would then gate the step, against the goal.
- Codex reviewer as a replacement for the Claude fan: the goal of 2026-10-27, set aside by the operator on 2026-11-03.
- Writing the assistant's report under `_lym` or `.lym` with Codex writable roots: rests on an untested `-c` setting; writing under `/tmp` needs none.
- Writing the assistant's report inside the worktree's own tree: an untracked file there would reach the round's fixer commit (D6).
- `developer_instructions` for the deny notice and passages: untested on Codex; the prompt file is proven.
- Skills synced into `$CODEX_HOME/skills` or `~/.agents/skills` and named with `$`: discovery untested, and the `$` popup eats Enter.
- Read-modify-write of Codex's `config.toml` with a TOML library: a new dependency for a file lym can own outright.
- A shared `assist_reviewer_model` fallback key: one segment could then not be turned off while another is on.
- The assistant always running the chair's fan: leaves no solo fallback, which the entry names, and fans Codex subagents before Step 4 by default.
- Awaiting the assistant's report file itself: returns on a first draft its gate may still rewrite.
- An empty resume line: reed falls back to the launch line and starts a second review.
- Keying trust on the worktree root alone: Codex keys a linked worktree on its main checkout.
- lym fork audit and `RequirementForks` for Codex subagents: board Step 4, deferred.
- Creel hook delivery and `codex queue` wakes: no role in scope needs mail.
- Codex `TranscriptReader` for `lym report` and `tokencount`: not in the entry's Steps 1 to 3.
- A hard link or copy of `auth.json`: a token refresh leaves a copy stale, and a logout and the next login, which create a new file, leave either stale.

## Review rounds

### Round 1

The review found the report path under `.lym` outside the Codex sandbox, since `.lym` links into the weft checkout as `_lym` does (BLOCKING): D2 moves the report under `/tmp` and keeps a copy beside the marker under `.lym` before a done marker.
Marker and outputs now key on the attempt token as well as the review path, and stale files go before the chair starts (D1).
`lym heddle list` showed no entry targeting `codex-file`; the shipped entries name Claude's tools, so D12 keeps them on `claude-file` and leaves the repository entry to the operator.
Codex 0.164.0's source showed `auth.json` written in place, recorded in D13.
D5 names the `[tui]` keys; D6 states the `.codex/` trust edge and corrects the dropped-prompt ending; D1 corrects its wait bound and the pane-mail channel; D2 names the assistant's no-parent directive; the skills-sync open risk moved into D9's rationale.
The fan before Step 4 went to the parent.

## Question ledger

- Q1 Which roles run on Codex? Auto-pick: the Burler review assistant only, all three segments; recommended by the goal of 2026-11-03. Resolved.
- Q2 How does the assistant reach the chair? Auto-pick: a new assistant seat kind with an ended marker and a file wait (D1); alternatives: advisor with creel, typed done line. Resolved.
- Q3 Where is the assistant configured? Auto-pick: per-segment per-round keys, default empty, plus a timeout key (D3). Resolved.
- Q4 Which fan does the assistant run? Auto-pick, revised at the lens review: a fan key of its own per segment and round, solo by default (D3); whether a fan may run before Step 4 is open for the parent. Open.
- Q5 Where does the assistant write? Auto-pick, revised at the lens review and round 1: under `/tmp` at a path keyed on the review path and attempt token, kept under the anchor's `.lym` before a done marker and copied beside the review by Go (D2). Resolved.
- Q6 How does Codex get the deny notice and passages? Auto-pick: the run's prompt file (D6). Resolved.
- Q7 How do skills load? Auto-pick: read by path from configured skill roots (D9); departs from the entry's Step 2 wording on tested grounds, stated in D9's rationale. Resolved.
- Q8 How is the home's `config.toml` kept? Auto-pick: rendered whole from lym's trusted-roots record under a lock (D5). Resolved.
- Q9 Shell guard text naming Claude tools? Auto-pick: a per-engine text beside the plain one in the row table (D10). Resolved.
- Q10 How are engine-specific stencil passages chosen? Auto-pick: engine blocks resolved by stencilstore for the seat's engine (D11). Resolved.
- Q11 Repository instructions for Codex? Auto-pick: heddle's codex-file falls back to `CLAUDE.md` (D12); shipped entries stay on `claude-file`, and adding `codex-file` to the repository entry is the operator's step, raised under Open risks. Resolved.
- Q12 `auth.json` link and Windows? Auto-pick: `CreateFileLink` in fslink, refused on Windows (D13). Resolved.
- Q13 Effort cap? Auto-pick: `efforts` list in `codex.yaml`, default low, medium, high (D4). Resolved.
- Q14 Live testing? Auto-pick: one llm-tier smoke test on `luna[low]`, run by the operator, capped at one run. Resolved.
