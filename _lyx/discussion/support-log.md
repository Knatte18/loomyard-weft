# Support log — config-layers-run-settings

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

1. Read the board entry and every entry touching config, batten or the orch: `burler-split-budget`, `orch-split`, `orch-wake-policy`, `doctor`, `logger-file-level`, `discussion-advisor-mode`, `lyx-cleanup`, `orch-any-worktree`.
   `burler-split-budget` adds per-perch keys that run settings will carry; `orch-split` needs run settings for a mechanical driver; `orch-wake-policy` drops the landing warning once run settings land.
2. Explored `internal/configengine`: both loaders already fill a present file's missing keys from the template (since `5aa04c279`), so the board's Go-zero claim is outdated and its first open question reduces to the per-layer rule.
3. Explored `configreg`: five hub-wide modules; darn is hub-wide because `verify` runs a shell command, and landing carries `publish_verify`.
   This drove D4's allowlist.
4. Found `configengine.StagingFile` at `.lyx/config/<module>.yaml`, the exact mirrored path a local override would take; picked the `.local.yaml` suffix (D3).
5. Explored batten: Seed-Child already reads the task's `recipe` fresh from the board, the natural home for writing run settings (D9).
6. Found the orch stencil's hand rule for `review_max_bounces: 2` (D10).
7. Settled D1–D10 and wrote the record.
8. Lens review: all six forks (premises, goal-scope, board-fit, pattern-fit, scenarios, leanness) returned a report.
   Duplicates merged; each finding with its lens and disposition:
   - scenarios BLOCKING: D6 stripped only in the seeding walk, so an interrupted walk or a skipped pair stayed full forever — fixed: per-worktree tracked marker makes the strip resumable.
   - scenarios + premises BLOCKING: a Stuck at Run-Shed self-routes and never halts — fixed: the Run-Shed rewrite is dropped; validation and write happen only at Seed-Child.
   - scenarios + premises MEDIUM: `lyx loom resume` bypasses Run-Shed, so "next launch" was false — fixed by the same change; D8 now returns a notice for a running task and names the in-pair `--layer local` route.
   - scenarios MEDIUM: Run-Shed rewrite could race a live child driver — fixed: Seed-Child runs before any child driver exists.
   - scenarios + premises MEDIUM: D5's unknown-key error in every layer broke `preserved` and downgrades — fixed: the refusal covers the local layer only.
   - scenarios MEDIUM: missing scenarios (Seed-Child crash and resume, empty run settings, empty local file, interrupted walk) — fixed: added to acceptance criteria; the concurrent board edit is covered by Seed-Child's single fresh read plus idempotent redo.
   - goal-scope MEDIUM: D4 overrides an operator ruling as an auto-pick — fixed: R1 asks parent review to confirm; per-module reasons added for gate, board and fabric.
   - goal-scope MEDIUM: kickstart incident left unowned — fixed: R4 hands the decision to the parent; the stale webster comment goes to Notes.
   - goal-scope LOW: retired key in `run_settings` blocks later upserts — fixed: D8 migrates on write.
   - goal-scope LOW: `--layer default` for a hub-wide module unstated — fixed in D7.
   - goal-scope LOW: no scenario for sparse `Set` or clearing run settings — fixed.
   - goal-scope + board-fit LOW/MEDIUM: `burler-split-budget` overlap unnamed and R2's retirement example wrong — fixed: R5 names the overlap and where its hub values go; the example is dropped.
   - board-fit MEDIUM: `orch-wake-policy` batten overlap unnamed — fixed: R6.
   - board-fit LOW: `lyx-cleanup` could discard local files — fixed: Constraints line.
   - board-fit LOW: `orch-any-worktree` widens with `--layer default`'s prime guard — rejected: the guard is the existing hub-wide one, which that note already has to relax; nothing new.
   - board-fit LOW: `doctor` and `logger-file-level` dependents — partly fixed: the early-loader note added; `doctor` is an unbuilt note with nothing to change.
   - pattern-fit MEDIUM: `.local.yaml` breaks the mirrored-path rule unnamed — fixed: Constraints names the narrowing.
   - pattern-fit MEDIUM: battenshed cannot import configreg — fixed: D9 uses injected `SeedChildDeps` closures wired in battencli.
   - pattern-fit MEDIUM: refusal-spec has no batten section — fixed: row under the shed section.
   - pattern-fit LOW: slug guard must parse through `agentname` — fixed in D8, sharing the configcli guard function.
   - pattern-fit LOW: `Migrate` outside `hubreconcile.Ensure` reads as a contradiction — fixed: Constraints states the exception.
   - pattern-fit LOW: board dir's dual meaning — fixed in D3 and Notes.
   - scenarios LOW: board-side edit from an unguarded session — fixed: D4's bound names board history.
   - scenarios LOW: silent removal of hand files in a pair — fixed: Warn per removed file.
   - leanness MEDIUM: A2–A4 restate decisions — fixed: A3/A4 dropped, assumptions reduced to pointers.
   - leanness LOW: generic PATTERN lines in Constraints — fixed: `no-denied-recovery`, `lyxdirs-single-declarer` and `sole-parsers` cut.
   - leanness LOW: D3 rationale repeats its body — fixed.
   - leanness LOW: A1 is a premise, not an auto-pick — fixed: moved to D1's rationale and R4.
   - leanness LOW: this turn's placeholder — fixed by this entry.
   - premises LOW: stale webster comment — fixed: Notes line.
   - premises LOW: D8 import claim holds — no change.

## Rejected alternatives

- Keep per-worktree files full and add only the local layer: the operator asked for the repo default layer, and full files would make it inert.
- Strip keys equal to the default on every reconcile: a later default change would erase deliberate overrides that equalled the old default.
- Local override at `.lyx/config/<module>.yaml`, moving the staging copy elsewhere: a staging file left by an aborted edit under the old binary would be read as an override.
- Read a hub-wide module's per-task override from the board task at load time instead of a pair file: couples `configengine` to the board and the slug.
- Let a local override set any hub-wide key: reopens the shell-command trust gap that made darn hub-wide.
- Let a per-worktree module's local layer set any key and only restate the bound: `reed.shell`, `shuttle.claude` and the `claude_deny_*` guards would then change untracked, unseen at landing, the same gap D4 closes for darn.
- An allowlist for per-worktree modules too: most of their keys are session tuning, so a denylist names the few dangerous keys instead of every safe one.
- Run settings as dotted keys (`loom.review_max_bounces`): the nested shape mirrors the file batten writes and the board's JSON.
- Rewrite run settings before every Run-Shed child launch: a Stuck there self-routes and never halts, `lyx loom resume` bypasses it, and a rewrite can race a live driver; Seed-Child only, with a notice on a running task's edit, was chosen after the lens review.
- Strip only in the walk that seeds the default: an interrupted walk or a skipped pair would never be stripped; a per-worktree marker replaced it.
- Refuse unknown keys in every layer: breaks `lyx config --set`'s `preserved` tolerance and a binary downgrade.
- Validate run-settings values at board write by calling each module's loader: `boardengine` cannot import `configreg` (cycle), and a generic shape check plus load-time validation suffices.
- Keep `darn.review_max_bounces` local with a floor of 1 at the local layer: a value rule for one key in one layer, where closing the key keeps D4 a key list.
- Strip keys by comparing values after `${env:...}` expansion: the result depends on the walk's environment and each worktree's `.env`, so a stripped literal could resolve differently in any other process.
- Merge open maps per entry across layers: departs from the existing template-versus-file rule; kept consistent instead.
- Merge a `models` alias key by key across layers: an override would inherit lower layers' `defaults` it cannot remove, the stale-default leak `modelspec.LoadRegistry`'s whole-entry replacement prevents.

## Review rounds

- Round 1: four findings, all fixed.
  F1 (BLOCKING): D4's bound was false for reed's and shuttle's executable, sourced-file, run-dir and guard keys — fixed: `LocalDenied` closes them to the local layer, and the board-history claim now covers only what Seed-Child writes.
  F2 (BLOCKING): pruning seed-only upper layers deleted operator aliases, lenses and fans — fixed: D2 leaves every seed-only layer untouched, and D5 and D8 accept their unknown keys.
  F3 (MEDIUM): D6's strip silently turned equal pins into inherited values — fixed: D6 states it and reports each stripped key.
  F4 (LOW): D9's removal did not name what it discards — fixed: D9 names the Stuck-window hand override.
- Round 2: three findings, all fixed.
  F1 (BLOCKING): `reed.mouse` and `reed.debug_log` set the hub's shared tmux server at boot yet stayed open to the local layer — fixed: reed's `LocalDenied` adds both, and D4 names a shared process's boot option as a category every key is tested against.
  F2 (MEDIUM): D6's leaf-wise strip inside an open map changed the resolved value — fixed: D6 strips an open map as one unit and never changes a worktree's resolution; an acceptance criterion pins it.
  F3 (LOW): D4's rationale ignored `${env:...}` markers on the denied keys — fixed: the rationale rests on persistence in the pair.
- Round 3: four findings, all fixed.
  F1 (BLOCKING): landing's `require_pr_to_base`, `squash` and `co_authored_by` and darn's `review_max_bounces` were open to the local layer, so a pair could skip PR approval and Darn-Review and reshape the parent branch's commit — fixed: `LocalKeys` keeps only model picks, timeouts and `verify_attempts`, and D4's categories add reviews, approval gates and what lands on the parent branch.
  F2 (MEDIUM): D4's rationale claimed the denied keys had no untracked route today, but the pair's `.env` binds their markers — fixed: the rationale rests on the local layer adding no new route, and R3 names `.env` for the parent.
  F3 (MEDIUM): D6 left the strip's comparison form open, and an expanded comparison could strip a literal equal to the walk's expansion of a marker — fixed: D6 compares unexpanded nodes, and an acceptance row pins the kept literal.
  F4 (LOW): seeding the default changes keys a worktree file lacks, such as a seed-only alias the prime holds — fixed: D6 narrows its claim to held keys and reports the others as `inherited`.
- Round 4: two findings, both fixed.
  F1 (BLOCKING): D4's categories classified keys by kind, so `loom.parent_review_wait_min`, whose expiry passes the parent-review gate, and burler's `lenses` and `fans`, the prose and lens set review forks are given, stayed open to the local layer — fixed: the guard category covers a gate-passing timeout and a reviewer's prose and lens set, loom and burler gain `LocalDenied` entries, and the seed-only exemption narrows to `models` aliases.
  F2 (MEDIUM): D5 refused open-map entries that D8 and D9 accept — fixed: D5 exempts a key path under an open map, and an acceptance row pins it.
- Round 2 rerun: two findings, both fixed.
  F1 (MEDIUM): D2 merged a `models` alias key by key, reversing `modelspec.LoadRegistry`'s whole-entry replacement, so an override inherited lower layers' `defaults` it could not drop — fixed: D1 and D2 carry each top-level alias whole from the highest layer holding it, D6 strips an alias as one unit, and an acceptance row pins an alias without `defaults`.
  F2 (LOW): D5's and D8's `models` exemption stopped at the alias name, refusing `opus.defaults.version` under an alias the template holds — fixed: both exempt any key path under a `models` alias, and the acceptance row names that entry.

## Question ledger

- Q1 (auto-pick): does a missing key fall back layer by layer to the template? Yes (D1); the template fill already exists.
- Q2 (auto-pick): how are layers shown? `--resolved` with per-leaf layer, reconcile results per layer, menu marks (D7).
- Q3 (auto-pick): base layer full, upper layers sparse (D2).
- Q4 (auto-pick): local override path `<anchor>/.lyx/config/<module>.local.yaml` (D3).
- Q5 (auto-pick, revised in review rounds 1 to 4): hub-wide modules' local layer restricted to `LocalKeys`, and per-worktree modules' to every key outside `LocalDenied`, excluding executables, sourced files, cross-pair keys including shared boot options and what lands on the parent branch, and guards including reviews, approval gates, gate-passing timeouts and reviewers' prose and lens sets (D4).
- Q6 (auto-pick, revised after lens review): migration strips each worktree lacking a tracked marker, once (D6).
- Q7 (auto-pick): `run_settings` nested by module, migrated and validated at write, slug-session guard (D8).
- Q8 (auto-pick, revised after lens review): batten writes a pair's local files only at Seed-Child (D9).
- Q9 (auto-pick, revised in review): open maps and `models` aliases carried whole across layers (D1).
- Q10 (open, for the parent): whether D4's narrowing of the operator's "any key" ruling stands (R1).
- Q11 (open, for the parent): whether the `kickstart_pack` incident's cause gets its own note (R4).
- Q12 (open, for the parent): whether the pair's `.env` route to marker-carrying denied keys gets an entry of its own, together with the process-environment route through the hub's shared tmux server, which reaches every pair (R3).
