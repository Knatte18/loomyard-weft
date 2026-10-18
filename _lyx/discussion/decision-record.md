# Decision record — orch-rules-files

## Goal

The orch's hub-specific rules leave the code repository.
The orch role file inlines two rule files in place of `ORCHESTRATOR.md`, hub-wide first:
the hub's rules for every orch, on Bolt at `<BoardDir>/_lyx/orch/rules.md`, and the orch thread's own rules, in its worktree's weft at `<anchor>/_lyx/orch/rules.md`.
Both are tracked, so the hub-wide file gets a verb that commits it on Bolt.
`ORCHESTRATOR.md` is deleted, with its mentions in `CLAUDE.md`, the orch role stencil, and the orch packages' code and docs.

## Scope

In:

- Two told rule paths and a told prime root on `orchengine.Paths`, replacing `RepoRulesPath`, joined once in `internal/orchcli/paths.go`.
- The role stencil renders both files in two sections, hub-wide first, each with its own placeholder when the file is absent or blank.
- A new orch verb, `lyx orch rules`, that writes the hub-wide rules file on Bolt and commits and pushes it.
- A board note, `orch-rules-move`, filed through `lyx board upsert --body-file`, holding D4's operator step and the moved rules verbatim.
- Deleting `ORCHESTRATOR.md` and rewording every reference to it: `CLAUDE.md`'s "Notes go in git" rule, the role stencil's `## Where a finding goes`, `## Scratch and notes` and closing section, `internal/orchcli` and `internal/orchengine` comments and `doc.go`s, and the tests that name the file.

Out:

- Orch threads in worktrees other than the prime: `lyx orch` stays prime-only (`orch-any-worktree` builds that).
  The own-rules path is derived from the orch's worktree anchor, so it needs no change when that lands.
- Editing the role stencil's board copy on Bolt to hold rules.
- Any change to fabric's weft forking or landing.
- The two `crucible/` mentions of "ORCHESTRATOR", which name crucible's own orchestrator prompt, `docs/research/` reports, which are measurement records, and generated content such as `_lyx/raddle/`, which is regenerated at landing.

## Decisions

### D1 — Two rule files, two told paths

The hub-wide file is `<BoardDir>/_lyx/orch/rules.md`, with `BoardDir` from `fabricengine.BoardDir(location.HubPath)`, the same base `configcli` uses for Bolt's `_lyx/config/`.
The own file is `<anchor>/_lyx/orch/rules.md` of the orch's worktree, today the prime, whose `_lyx` is the link to `main-weft`'s `_lyx`, so the file shows inside the prime's folder and is tracked on `main-weft`.
Both paths are joined in `orchPaths` from `lyxdirs.LyxDirName` and the existing `orchDirName`, and handed to `orchengine` as `Paths.HubRulesPath` and `Paths.OwnRulesPath`; `RepoRulesPath` goes.
`RepoRulesPath`'s directory is also the prime's worktree root the watcher's `isRequestedNote` resolves a relative note path against, and neither new path's directory is that root.
So `Paths` gains a told `PrimeRoot` field, joined in `orchPaths` from `location.WorktreePath()`, and `isRequestedNote` reads it in place of `filepath.Dir(RepoRulesPath)`.

Rationale: the operator decided both locations, and `_lyx/orch/` mirrors the existing `.lyx/orch/`.

### D2 — The role file renders both files in two sections

The role stencil's closing `## This repository` section and its `repository_rules` marker are replaced by two closing sections, in this order:

1. `## Hub-wide rules`, filled by a `hub_rules` marker with the hub-wide file.
2. `## This orch's rules`, filled by an `orch_rules` marker with the own file.

The preamble the current section carries (the rules add to the role, override none of it, and a contradiction is raised to the operator while the role is followed) moves above the two sections and covers both.
It gains one sentence: a rules file's own headings start at level three (`###`), since the file is inlined under a level-two section.
`RenderRoleFile` takes the two paths in place of `rulesPath`, checks for both new markers as it checks the others today, and reads each file the way `readRepositoryRules` does: a missing or blank file yields a one-line placeholder naming that file's absolute path, so the orch knows where a rule would go;
any other read error fails the render naming the file, as today.
Each file's text is inlined verbatim.
`start`, the watcher and `resume-context` pass the two paths wherever they pass `RepoRulesPath` today.

Rationale: hub-wide first is the operator's order; a placeholder naming the path replaces the "This repository has no ORCHESTRATOR.md." line, which no longer names anything.

### D3 — `lyx orch rules --from <file>` writes the hub-wide file on Bolt

The verb replaces the whole hub-wide rules file with `<file>`'s content, creating `<BoardDir>/_lyx/orch/` when absent, with a temporary-file-and-rename write so a concurrent role render never reads a partial file.
Its help text says the verb replaces the whole file, so a draft starts as a copy of the current hub-wide file at its Bolt path.
The temporary file is created beside the rules file, so a write killed between create and rename can leave it untracked in Bolt, where the next whole-repo `Bolt.Commit` sweeps it in.
The write runs through `fabricengine.NewBolt(...).PullThenCommitWritten` with one write whose path list is the one rules path, so Bolt is brought up to date with its upstream under the board write lock before the write and its commit, then `Push`.
When Bolt cannot be brought up to date, nothing is written, and the refusal names the result's `SkipDetail` as the way forward.
Unchanged content commits nothing.
Before the pull-first write, the verb pushes Bolt's unpushed commits with `Push`, whose recovery rebase carries them over a non-conflicting upstream change; the pull-first write would otherwise skip a Bolt holding an unpushed rules commit behind its upstream as diverged.
A failed first push is refused with the push error, and nothing is written.
After the write the verb pushes again, so every successful run, unchanged content included, leaves the earlier commit pushed.
An empty or whitespace-only `<file>` is accepted and committed: it is how the hub-wide rules are cleared, and the role file then shows the placeholder.
`--from` is required and resolves against the caller's working directory; a missing or unreadable `<file>` is refused naming the flag and the file, before any lock is taken.
A failed commit leaves the new content in place, and the envelope reports it as written but uncommitted, with re-running `lyx orch rules --from <file>` as the way forward, since that run commits the one path under the verb's own message.
A board write before the re-run sweeps the file into its own commit under its own message.
A failed push leaves file and commit in place and reports the commit as made but not pushed, for the next run of the verb to push.
The envelope names the hub-wide path, whether a commit was made, and whether Bolt was left with unpushed commits.

Bound: the verb commits only `<BoardDir>/_lyx/orch/rules.md`, never another Bolt path; a killed write can leave its temporary file untracked beside it, as above.
It publishes whatever `<file>` holds to Bolt and its remote, so it reads only the file it is named.
It runs only with the prime's worktree as its working directory, under every orch verb's existing prime-only refusal.
That refusal checks the directory, not the caller, so any session can reach the verb from the prime, as it can edit today's `ORCHESTRATOR.md` there.
What bounds the written text is the role preamble D2 moves: the rules add to the role, override none of it, and a contradiction goes to the operator.

The own file needs no verb: in the prime it is a `main-weft` file that `lyx fabric commit` and `lyx fabric sync` already commit.

Rationale: a hand-edited Bolt file stays uncommitted, and `Bolt.Commit` stages every change in Bolt's repo, so the edit would be swept into an unrelated board or sync commit under another message.

### D4 — Where the current rules go, and who moves them

Both rules in today's `ORCHESTRATOR.md` are about this hub, not one orch thread, so both move to the hub-wide file.
The own file of the prime starts absent.

A card cannot write either file: the only route that commits the hub-wide file is `lyx orch rules`, which this task ships, which is in production only after the deploy, and which runs only from the prime; the prime's own file is on `main-weft`, and weft content never lands from a task pair.
No landing path gives `ly:orch` a turn with this record: an alignment approve at PR-Gate lands and deploys with no orch turn, and a task's `run_settings` cannot set `alignment_pr_approve`, a guard key the local layer refuses.
So the operator moves the rules, from the prime, after this task lands and the deploy has put the new verb in production:
draft the hub-wide file in `.scratch/` with exactly this text, and run `lyx orch rules --from <draft>`.
A card files a board note through `lyx board upsert --body-file`, slug `orch-rules-move`, whose body holds this step and the text below verbatim, so the step is on the board when the task lands.
The body states that the note is an operator step, run from the prime after the deploy, and never promoted to a run, since a task pair's call to the verb is refused.

```markdown
### Production

An exception to `CLAUDE.md`'s Production rule: after your own pushed fix, on a clean `main`, you run `update-plugins.sh` yourself and say so in your report.

### Watching runs

The operator watches each run in its own terminal window, never in VS Code.
VS Code opens only when the operator asks for it, through `lyx ide` or `batten.yaml`'s `open_ide`.
```

Rationale: the content is the operator's and moves verbatim, without the old H1 and with its H2s as `###`; the only route that commits the hub-wide file is the new verb.

### D5 — Rule-placement wording in `CLAUDE.md` and the role stencil

`CLAUDE.md`'s line "A rule or workaround only the orch needs goes in `ORCHESTRATOR.md`" becomes: it goes in the orch rules, never in `CLAUDE.md`: the hub-wide file on Bolt, `_board/_lyx/orch/rules.md`, written through `lyx orch rules`, when every orch of the hub needs it, and the orch thread's own `_lyx/orch/rules.md` when only that thread does.
The role stencil's two placement sentences say the same:
a workaround only orchs apply goes in the hub-wide file; a durable orch-only rule goes in the hub-wide file when every orch of the hub needs it and in this orch's own file when only this thread does.
Both name `lyx orch rules` for the hub-wide file and `lyx fabric commit` for the own file.
The role stencil's placement sentence for the hub-wide file also says that the verb replaces the whole file, so a draft starts as a copy of the current hub-wide file, `_board/_lyx/orch/rules.md`.

Rationale: every mention the board entry names is reworded, and an orch rule never lands in a file every agent reads.

## Constraints

- Told-Geometry: `orchengine` derives no path; `orchcli/paths.go` joins both.
- Lyxdirs Single-Declarer: `_lyx` is named through `lyxdirs.LyxDirName` only.
- Durable-vs-Ephemeral State: both rule files sit under `_lyx`, mirroring `.lyx/orch/`.
- Stencil Ownership: the role stencil stays read at call time from the told stencils directory; the shipped stencil changes, and an unedited board copy is refreshed by the first process the deployed stamped binary runs.
- CLI cobra: `rules` is a verb in the existing `orch` subtree.
- Fabric git: Bolt is touched only through `fabricengine`'s `Bolt` methods, with a positive-only file list of the one rules path, as `configcli`'s hub config writes already are.
- Docs in the same commit: `internal/orchcli/doc.go` and `internal/orchengine/doc.go` describe the two files and the verb.

## Auto-mode assumptions

The support log's Question ledger holds every auto-pick; D2, D3 and D4 carry them.

## Open risks

- The deployed binary's first process reseeds the role stencil's board copy without `repository_rules`, and a watcher still running the old binary then refuses every role render naming that marker, so its cycles abort.
  On Linux the watcher re-execs onto the deployed binary at its first restart-safe idle tick, which bounds the refusal window to the time between the stencil reseed and that tick.
  The re-exec runs only after a tick that succeeded, so a watcher that reaches its tick-error cap first exits without it.
  On Windows, where `selfexec` never re-execs, and on Linux after such an exit, the operator restarts the orch with `lyx orch stop`, then `lyx orch start --continue`, from the prime.
  An operator-edited board copy that keeps `repository_rules` makes the new binary refuse in the same way until the copy is ported or deleted; today's copy is unedited.
- Between deploy and the operator running D4's step, the role file shows the hub-wide placeholder, so the two rules are absent from the orch's role for that interval; their text survives in this record and in git history.
- D3's bound rests on the prime-only refusal of every orch verb; `orch-any-worktree`, which lets `lyx orch` run outside the prime, must bound `lyx orch rules` again.
- A pair's weft forks from its parent worktree's weft, today `main-weft`, so a new pair's `_lyx/orch/rules.md` is a copy of its parent's own file.
  Nothing reads it outside the prime today; `orch-any-worktree` and `pair-from-parent`, which give pairs their own orch threads, inherit that copy as the new thread's starting rules.
- The moved Production rule predates the Land row's automatic deploy that `CLAUDE.md` now describes; it moves verbatim, and whether it still holds is the operator's call.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d3-failed-commit-recovery-is-sweep (class consistency, severity MEDIUM, open in rounds 2)
  - d4-bolt-premise-false-reminder-gap (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- `ORCHESTRATOR.md` is gone from the repository, and no tracked source, stencil or doc names it outside Scope's exclusions.
- A role render with both files present inlines the hub-wide file's text under `## Hub-wide rules` before the own file's text under `## This orch's rules`.
- A role render with either file missing, or holding only whitespace, shows that file's placeholder line naming its absolute path, and renders the other file normally.
- A role render with a rules file that exists but cannot be read fails, names the file, and writes no role file.
- A role stencil missing `hub_rules` or `orch_rules` fails the render naming the missing marker.
- `start`, the watcher and `resume-context` each render both files.
- `orchPaths` puts the hub-wide path under the hub's board dir and the own path under the worktree anchor's `_lyx/orch/`.
- The watcher resolves a relative note path against the told prime root.
- `lyx orch rules --from <file>` on a hub with no `_lyx/orch/` on Bolt creates the file with the given content and leaves one new Bolt commit touching only that path, pushed.
- The same call with unchanged content makes no commit and succeeds.
- A retry with the same content after a failed push makes no new commit and pushes the earlier one.
- A call while Bolt cannot be brought up to date with its upstream writes nothing, and the refusal names the skip's way forward.
- An empty `<file>` commits an empty hub-wide file, and the next role render shows the hub-wide placeholder.
- `--from` naming a missing file is refused before any Bolt write, and leaves the hub-wide file untouched.
- A failed write leaves the hub-wide file at its prior content, and a failed commit is reported as written but uncommitted, naming the re-run as the way forward.
- A failed push after a landed commit leaves the new content and the commit in place and reports the commit as not pushed.
- `lyx orch rules` from a task worktree is refused by the orch's prime-only refusal.
- The board holds a note `orch-rules-move` whose body holds D4's operator step and the moved rules verbatim, and says it is never promoted to a run.

## Notes for the plan writer

- `reportHubWrite` in `internal/configcli/hubwide.go` shows the uncommitted report; D3 commits through `PullThenCommitWritten`, the pull-first sibling of the `CommitWritten` that `newHubCommit` uses.
- `PullThenCommitWritten` documents an empty path list as committing nothing; confirm an unchanged single path also reports no commit rather than an error.
- The `orch` command's persistent pre-run resolves engine config for every verb; whether `rules` needs all of it is the plan's call.
- Tests: a fake commit seam in the style of `hubCommitFunc` for a failed write leaving the prior content, the uncommitted report, the pull skip, push-failure and no-change cases, and a `hubforge` hub only for the one real Bolt commit.
- The `ly:cli` index lists `lyx orch rules` once the verb is registered; no hand-written index edit is needed.
