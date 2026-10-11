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
- Run settings as dotted keys (`loom.review_max_bounces`): the nested shape mirrors the file batten writes and the board's JSON.
- Rewrite run settings before every Run-Shed child launch: a Stuck there self-routes and never halts, `lyx loom resume` bypasses it, and a rewrite can race a live driver; Seed-Child only, with a notice on a running task's edit, was chosen after the lens review.
- Strip only in the walk that seeds the default: an interrupted walk or a skipped pair would never be stripped; a per-worktree marker replaced it.
- Refuse unknown keys in every layer: breaks `lyx config --set`'s `preserved` tolerance and a binary downgrade.
- Validate run-settings values at board write by calling each module's loader: `boardengine` cannot import `configreg` (cycle), and a generic shape check plus load-time validation suffices.
- Merge open maps per entry across layers: departs from the existing template-versus-file rule; kept consistent instead.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): does a missing key fall back layer by layer to the template? Yes (D1); the template fill already exists.
- Q2 (auto-pick): how are layers shown? `--resolved` with per-leaf layer, reconcile results per layer, menu marks (D7).
- Q3 (auto-pick): base layer full, upper layers sparse (D2).
- Q4 (auto-pick): local override path `<anchor>/.lyx/config/<module>.local.yaml` (D3).
- Q5 (auto-pick): hub-wide modules' local layer restricted to `LocalKeys`, excluding shell-command keys and hub-level facts (D4).
- Q6 (auto-pick, revised after lens review): migration strips each worktree lacking a tracked marker, once (D6).
- Q7 (auto-pick): `run_settings` nested by module, migrated and validated at write, slug-session guard (D8).
- Q8 (auto-pick, revised after lens review): batten writes a pair's local files only at Seed-Child (D9).
- Q9 (auto-pick): open maps carried whole across layers (D1).
- Q10 (open, for the parent): whether D4's narrowing of the operator's "any key" ruling stands (R1).
- Q11 (open, for the parent): whether the `kickstart_pack` incident's cause gets its own note (R4).
