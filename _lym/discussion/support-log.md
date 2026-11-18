# Support log: Hardener

## Interview

Autonomous run: no operator answered; every pick below is the writer's own and is marked auto-pick in the question ledger.

### Turn 1 — exploration

Four explore subagents mapped the ground before any design call:

- `internal/treadleengine` is a complete round loop with a judge-maintained handoff, a lossless finding ledger and pre-round targeting, but no production importer and no reach from any `main`.
- Shed recipes are YAML under `contracts/recipes/`; darn and sampler are loom recipes that take their record from the board through Record-Write, build in one row, run the Code-Review ring and land through Finalize.
- A burler round is a review half writing only its review file and a fix half spawned after it; `tool-use: true` lets the reviewer build, run and test; `fix-scope: source` commits per fix; per-round model lists exist.
- The Bouncer is a live producer with a per-round ledger keyed by finding, Go-rendered facts with recurring keys, a stalemate count, a focus file for the next round and CIRCLING escalation to the parent.
- `deploy-dev` installs into a per-worktree `.dev-bin`; `hubforge` builds wired hubs from local bares through `fabriccli.CloneAndWire` but is test-only; `tmuxkit.Main` already fails a package for stray processes; tier 2 is real tmux, tier 3 billed LLM.
- The GitHub-backed sandbox hub is one shared hub for operator dogfooding.

### Turn 2 — design calls

- The Bouncer already does what treadle was kept for, so Hardener is built on the Bouncer and burler rows, and treadle is retired (D3, D8).
- The loom contract needs exactly one build row on the on_done path; the fix row sits on on_stuck edges, so the contract reads a segmented build row at its Bouncer (D10).
- A round's independence comes from Go re-running gates on the committed tree and from a clean-room judge reading Go-written facts (D4, D6).
- Sandbox provisioning is automated as throwaway local hubs, because concurrency claims need a second hub and the GitHub hub is shared (D5).

### Turn 3 — lens review

Every lens returned a report. Duplicates across lenses are merged into one line.

- interactions, walkthrough (BLOCKING): burler's model lists clamp, so rounds past three ran one model and the different-model convergence rule could never hold. Fixed: D3's perch expands the lists cyclically to the cap, which round 1's review (F6) moved to the hard cap; criterion added.
- interactions, existing-paths, bound-breaker (BLOCKING): a process-owned lymtemp directory is swept when `sandbox new` exits. Fixed: D5's root is a fixed directory under the worktree's `.lym`, shared by every process of the run.
- interactions, walkthrough, bound-breaker (MEDIUM): the Bouncer's commit seam fires only on an approved settle. Fixed: D7 archives at the end of every fix row; the seam keeps only the final ledger, with the halt case bounded.
- interactions, existing-paths, walkthrough, bound-breaker (MEDIUM): `verify_source: tree_scope` reads `darn.yaml`'s scope and attempts. Fixed: D1 has the arm read them from `hardener.yaml`; D9 gains the keys; criterion added.
- interactions, existing-paths, walkthrough, bound-breaker (MEDIUM): `packages` and `llm_driving` were missing from D9's unit. Fixed: both are unit keys with defaults (D2, D9).
- existing-paths, interactions (MEDIUM): the loom wiring knows no Harden-Review segment, so the cap and models fell back to `loom.yaml`. Fixed: D3 adds a Harden perch reading `hardener.yaml`.
- existing-paths (MEDIUM): batten's child-recipe list, loomrecipe's lookups (including the Bouncer-row lookup the parent's verbs use) and reportengine's lists are closed. Fixed: D1 states that every closed recipe list admits `hardener`; criteria added.
- interactions (MEDIUM): round one drove a stale or missing `.dev-bin`. Fixed: D5's `sandbox new` deploys HEAD when stale; deploy-first joins the rubric criterion.
- walkthrough (MEDIUM): the facts renderer had no writer for the new per-round inputs. Fixed: D6 names each file's writer.
- walkthrough, interactions (MEDIUM, LOW): HardenFix's verify gate named no gate key. Fixed: D3 names `harden-burler-fix`.
- bound-breaker (MEDIUM): an uncommitted `loom.yaml` override does not reach pairs created later. Fixed: D5 commits it onto the sandbox hub's prime records.
- bound-breaker (MEDIUM): concurrent `lym gate test` copies contend for the hub's slot pool. Fixed: D4 adds `--copies`, running copies inside one call's held slots, with its bound.
- bound-breaker, interactions (MEDIUM, LOW): an interrupted row leaves hubs alive, charged to the next round. Fixed: D5 reaps at row start too and marks such leftovers.
- walkthrough, bound-breaker (LOW): the gate's deploy step served none of the gate's own runs. Fixed: the deploy moved to `sandbox new`.
- interactions, bound-breaker (LOW): tmuxkit's stray check covers only packages calling `tmuxkit.Main`, on Linux. Fixed: D4 states the bound; the live report names uncovered packages.
- walkthrough (LOW): a round that changes reed's socket-key derivation can leave a server the reap misses. Fixed by stating the bound in D5.
- bound-breaker (LOW): whether a hub `agents.yaml` naming a retired role still loads. This line first recorded the false premise that `internal/agentconfig` refuses no unknown role, and D8 dropped the reinterpretation; round 1's review (F1) showed `agentconfig.Load` refuses such a role, so D8 ships a reinterpretation on the `agents` module.
- board-scope (MEDIUM, decision): the operator note lists treadle to build on. Kept D8 and turned the Open risks line into a question for the parent; the operator's 2026-11-18 entry later settled it, and round 2 removed the line (Q13).
- board-scope (LOW, decision): "the sandbox work" may mean the bwrap note. Fixed: D5's rationale states the reading; no parent question, since the bwrap note is unbuilt and D5 conflicts with nothing in it.
- board-scope (LOW): the `manifest-retire` list was not addressed. Fixed: Scope's Out says no file holds it.
- leanness (LOW): Constraints restated PATTERN entries. Fixed: cut to task-specific bullets.
- leanness (LOW): the CloneAndWire fact sat in D5 and Constraints. Fixed: kept in Constraints only.
- leanness (LOW): D6's closing sentence and D8's history were narrative. Fixed: trimmed.
- leanness (LOW): the second Open risk was neither risk nor question. Fixed with the board-scope finding above.
- leanness (LOW): the first plan-writer note repeated D1 and D3. Fixed: cut.
- leanness (LOW): Interview turn 1 repeats the relevant list. Rejected: the support log is only for review, as the lens itself says.
- leanness (MEDIUM, emphasis): the tier-3 rule sits in both D4 and the rubric criterion. Rejected: no change asked; the D4 criterion tests the gate alone.

## Rejected alternatives

- Build Tenter on `internal/treadleengine` as the board draft proposed: rejected, since it duplicates the live Bouncer's ledger, circling check and focus file, and would revive an engine no `main` reaches beside the one loom uses.
- Write a from-scratch round runner, as the draft's "only Bouncer carries over" said: rejected, since burler's `tool-use` and `fix-scope: source` already give a reviewer that drives the substrate and a fixer that commits per fix.
- One agent doing review then fix in one session, as crucible did: rejected, since burler's two sessions enforce review-before-fix in Go instead of by prompt.
- A persistent LLM orchestrator session: rejected by the draft itself and again here; Go plus a per-round judge holds the loop.
- Use the GitHub-backed `lym-test` sandbox hub: rejected, since it is one shared hub needing network and a wiki, and cannot give concurrent or second-hub scenarios.
- Have the operator provision sandboxes: rejected, since an unattended campaign needs fresh hubs per scenario.
- A recipe without `contract: loom` and its own arm, as batten has: rejected, since landing, Describe, PR-Review and verify wiring all come through the loom arm.
- A fake build row (a Go setup step registered as a build engine): rejected as dishonest to the contract's meaning.
- Make the N-way concurrent run a merge gate: rejected per crucible's own finding that a timeout under an artificial CPU peg is not a defect.
- Drift rows in the recipe: rejected, since a hardening campaign cannot predict its changes.
- A new `lym harden start` verb creating the task: rejected; filing a task is the operator's, and the board refuses an agent creating one.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick) Build on treadle or on the Bouncer? Bouncer, and retire treadle (D3, D8). Rationale: same capability, already live.
- Q2 (auto-pick) Reuse the burler package or only its prompt template? The package's review and fix rows (D3).
- Q3 (auto-pick) Where is the line between Go and the LLM? Go runs the loop, deploys, runs the live gate, provisions and reaps sandboxes, renders facts and archives rounds; the LLM reviews, fixes, judges convergence from Go facts and writes the next round's focus (D4 to D7).
- Q4 (auto-pick) What must the handoff carry besides the ledger? Per-round models, live outcome, stray report, deferred findings and fix commits (D6).
- Q5 (auto-pick) How much sandbox provisioning does lym automate? All of it, as throwaway local hubs (D5).
- Q6 (auto-pick) How is a campaign started? As a board task with `recipe: hardener` and `run_settings` naming the packages (D1, D2).
- Q7 (auto-pick) How does the fix row satisfy the loom contract? A segmented build row is placed at its Bouncer (D10).
- Q8 (auto-pick) Defaults: seven rounds, three concurrent copies, Opus/Fable/Sonnet rotation, from the reed and fabric campaigns (D9).
- Q9 (auto-pick) Where does crucible's method go? Package doc for the why, rubric stencil for the rules (D12).
- Q10 (open, carried to Open risks) A hand-run `lym selfreport create` from a round cannot be blocked by hub config.
- Q11 (auto-pick) Where does the sandbox root live? Under the worktree's `.lym`, since a lymtemp process directory is swept when its owner exits (D5).
- Q12 (auto-pick) How do concurrent copies get slots? A `--copies` flag on `lym gate test`, inside one call's held slots (D4).
- Q13 (resolved) May treadle be deleted when the operator note lists it among the modules to build on? Yes (D8), settled by the operator's 2026-11-18 Added after Discussion entry, Delete treadle when Hardener is built (F10).
