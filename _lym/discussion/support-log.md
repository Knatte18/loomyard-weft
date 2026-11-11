# Support log — claude-account-weights

## Interview

### Turn 1 — exploration, autonomous mode

The session ran autonomously with no advisors, so no operator answered; every pick below is an auto-pick recorded in the question ledger.
I read the board entry, which carries no `Decision for claude-account-weights:` note, and the peers.
The peers `parent-reviews-decision-record`, `hub-workspace-all-pairs` and `testmap-selector-upkeep-rules` touch none of the claude engine, the `claude` config module or reed's pane launch.

Four subagents explored:

- Upstream (Claude Code 2.1.322 installed): `CLAUDE_CONFIG_DIR` moves login, settings, transcripts and `.claude.json`.
  The status line input carries `rate_limits.five_hour` and `rate_limits.seven_day` since 2.1.80, for subscription accounts after the first API response.
  `/usage` is interactive only, and the OAuth usage endpoint is undocumented and reported to rate-limit.
- Launch: the engine's `Prepare` builds the launch and resume commands, and reed's pane line exports the `env` module first and the launch command last.
  `hubgeom`'s `paneEnv` closure knows the worktree, but naming a claude variable there would break the provider seam.
- Readers: the transcript, session-registry, resume and audit readers take `$HOME/.claude`, and only the plugins lookup honours `CLAUDE_CONFIG_DIR`; hook-driven readers use `transcript_path`.
  Lym renders no status line today.
- Config: `claude` is a per-worktree module; `CategoryMachine` opens a key to `<Bolt>/.lym/config/claude.machine.yaml`; list values are supported as a YAML sequence decoded by the module.

### Turn 2 — design settled by auto-pick

Headroom source: the status line, the only documented source of the session and weekly percentages (Q1).
Persistence: a per-worktree record plus a hub-wide counter under a lock (Q2, Q3).
Long-lived sessions follow the per-worktree rule (Q4).
Limits are waited out, never moved (Q5).
The default directory is unset in the launch rather than set, to keep `.claude.json` in place (Q6).

### Turn 3 — lens review

Every lens returned a report.

- bound-breaker 1 (BLOCKING), pre-change sessions picked on resume: fixed; a resume with no record makes no pick and records the directory holding its transcript (D2).
- bound-breaker 2 and interactions 1 and 6, re-pick of a removed account against resume and the under-lock re-check: fixed; a record is kept while its directory exists, and only a fresh launch re-picks (D2, D3 step 1).
- bound-breaker 3, accounts, counter and snapshots per hub: fixed by stating the per-hub bound in D1 and D3 and an open risk; a machine-wide per-user store has no `lymdirs` home and would depart from `PATTERN-lym-files`.
- bound-breaker 4 and interactions 3, concurrent status line writers lowering figures: fixed; per-window merge under a lock, later reset or higher percentage wins (D5).
- bound-breaker 5, existing-paths 3 and leanness 8, wrong precedence reason: fixed; D5 names the `--settings` file.
- bound-breaker 6, layer provenance for the tracked-layer refusal: fixed; walkthrough found `configengine` reports provenance, and D1 names it.
- bound-breaker 7, standalone runs: fixed; Scope states they keep today's behaviour.
- bound-breaker 8, the stat proves existence only: fixed; D3 states the bound.
- interactions 2 and walkthrough 4, `env` module directory invisible to readers and reports: fixed; D6 reads the module's value among the known directories.
- interactions 4, fallback could take a logged-out account: fixed; D3 ranges the fallback over candidates only.
- interactions 5 and walkthrough 1, unnormalised path matching: fixed; D5 passes the account name to the verb and D1 cleans every compared path.
- walkthrough 2, credentials gate on macOS: fixed; D3 skips the stat there.
- walkthrough 3, seeding a new account at the highest ratio: fixed; seeded at the lowest (D3).
- walkthrough 5, `tools/tokencount` outside a hub: fixed; D6 states what it reads there.
- existing-paths 1, id-only liveness reader: fixed; D6 looks the id up in every known directory's registry.
- existing-paths 2, deploy syncs plugins into `~/.claude` alone: fixed; D9 has the operator link the second directory's `plugins` to the default's.
- leanness 1, Notes duplicate `relevant.md`: fixed; section dropped.
- leanness 2, assumptions restate decisions: fixed; cut to the two not stated as decisions, pointing at them.
- leanness 3, rejected alternatives in D5's rationale: fixed; cut.
- leanness 4, generic constraints: fixed; cut reed-panes, test and owner-machine bullets, kept cli-cobra for the new verb.
- leanness 5, open risks restating decisions: fixed; cut.
- leanness 6, repeated usage-limit wait: fixed; stated in D8 alone.
- leanness 7, D9's procedure too long: fixed in part; compacted, kept because the board asks the task to state the wiring.
- leanness 9, "forks' parents": fixed; dropped.
- leanness 10, pinned record path: rejected; the manual check needs a known place, and `.lym` holds other caches with no `_lym` counterpart.
- board-scope: no findings.

## Rejected alternatives

- Hash of the slug for the pick: cannot keep the weighted share once accounts are skipped for headroom.
- Plain weighted random pick: shares drift over small samples; the counter is exact and as cheap.
- Headroom from lym's transcript totals against a configured token budget: gives tokens, not the plan's percentage, and the operator says the plan's figures are readable.
- Headroom from the undocumented OAuth usage endpoint: undocumented, needs reading the credentials file, and reported to return persistent 429s.
- Setting `CLAUDE_CONFIG_DIR` in `hubgeom`'s `paneEnv` closure: names a claude specific outside the claude engine, against `PATTERN-shuttle-provider-seam`.
- Accounts in the hub's tracked `claude.yaml` or in heddle's `claude-settings` target: both are pushed to GitHub, which the operator ruled out for account details.
- Moving a limited worktree to another account: the session's history stays in the first account's directory, so the moved session cannot resume.
- Re-picking the prime's and beams' account at each fresh orch start: splits one worktree's sessions over two directories, and resume needs one.
- Chaining the operator's own status line command from lym's: no requirement asks for it.
- Picking on a resume with no record: moves a pre-change session away from its transcript, and the resume check refuses.
- Re-picking when a recorded account leaves the list: strands that worktree's resumable sessions.
- A machine-wide per-user store for usage figures shared across hubs: no `lymdirs` home exists for one, and it would depart from `PATTERN-lym-files`.
- Matching the status line's `CLAUDE_CONFIG_DIR` to an account by path: a differently spelled path records nothing; the account name passed as an argument cannot miss.
- Having deploy sync plugins into every account's directory: a link of the second directory's `plugins` reaches the same with no code.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): which headroom source? Status line input, recorded per account. The only documented source of both window percentages.
- Q2 (resolved, auto-pick): where is the pick recorded? A never-tracked per-worktree file under the anchor's `.lym`, holding the name and the directory. It dies with the worktree and survives resume and hand passes on the owner machine; it is kept while its directory exists.
- Q3 (resolved, auto-pick): pick algorithm? Lowest count over weight among eligible accounts, counter under a lock, new accounts seeded at the current share. Exact long-run share, even spacing, no catch-up burst.
- Q4 (resolved, auto-pick): rule for orch and dobby? Same per-worktree rule for prime and beam worktrees. Resume needs one directory per worktree.
- Q5 (resolved, auto-pick): move on limit or report? Report and wait, as today. A moved session cannot resume.
- Q6 (resolved, auto-pick): account whose directory is `~/.claude`? Unset `CLAUDE_CONFIG_DIR` in the launch. Setting it would relocate `.claude.json`.
- Q7 (resolved, auto-pick): precedence over an `env` module `CLAUDE_CONFIG_DIR`? The account wins when accounts are configured; otherwise the `env` value stands.
- Q8 (resolved, auto-pick): invalid accounts config? Refused, failing the launch; never degraded to no accounts, which would silently put every session on one account.
- Q9 (resolved, auto-pick): is an account without a login eligible? No: a stat of `.credentials.json` gates it, without reading it.
- Q10 (resolved, auto-pick): how the second account is wired for the test: fake directories in automated tests and a one-time manual check by the operator (D9).
- Q11 (resolved, auto-pick, from lens review): a resume with no record? Record the directory holding its transcript, no pick.
- Q12 (resolved, auto-pick, from lens review): per-hub or machine-wide usage figures? Per hub, bound stated; no machine-wide lym store exists.
- Q13 (resolved, auto-pick, from lens review): plugins on the second account? The operator links its `plugins` to the default's, so deploy reaches both.
