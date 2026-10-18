# Orch rules move to hub-wide and own files, ORCHESTRATOR.md goes

The orch's hub-specific rules leave the code repository.
Anyone cloning the repository no longer receives the operator's choices for how lyx behaves in this hub, and an orch thread can carry rules of its own.

## What changes

- `ORCHESTRATOR.md` is deleted.
  `CLAUDE.md`, the orch role stencil, `docs/overview.md` and the `internal/orchcli` and `internal/orchengine` comments and `doc.go`s no longer name it.
- The orch role file inlines two rules files in place of it, hub-wide first.
  Under `## Hub-wide rules` (`hub_rules` marker) is the hub's file on Bolt, `<BoardDir>/_lyx/orch/rules.md`.
  Under `## This orch's rules` (`orch_rules` marker) is the orch thread's own `<anchor>/_lyx/orch/rules.md`, which for the prime is tracked on `main-weft`.
  A missing or blank file renders a one-line placeholder naming its absolute path.
  Any other read error fails the render naming the file.
  A stencil missing either marker fails the render naming it.
- `orchengine.Paths` loses `RepoRulesPath` and gains told `HubRulesPath`, `OwnRulesPath` and `PrimeRoot`, all joined in `internal/orchcli/paths.go`.
  The watcher's `isRequestedNote` resolves a relative note path against `PrimeRoot`, since neither new rules path sits in the prime's worktree root.
- New verb `lyx orch rules --from <file>` replaces the whole hub-wide file on Bolt with `<file>`'s content, then commits and pushes it.
  The own file needs no verb: `lyx fabric commit` and `lyx fabric sync` already commit it.
- `CLAUDE.md` and the role stencil now say where an orch-only rule goes: the hub-wide file through `lyx orch rules`, or the thread's own file through `lyx fabric commit`.

## Design decisions

- The verb writes through a temporary file and rename, then `fabricengine`'s `PullThenCommitWritten` with the one rules path, so Bolt is current under the board write lock before the write.
  A killed write can leave its temporary file untracked beside the rules file, where the next whole-repo Bolt commit sweeps it in.
- Before the write the verb pushes Bolt's unpushed commits, and after it pushes again, so every successful run leaves the earlier commit pushed.
  Unchanged content commits nothing.
  An empty `<file>` is accepted and clears the hub-wide rules.
- A missing `--from` file is refused before any lock is taken.
  A failed commit is reported as written but uncommitted, with the re-run as the way forward; a failed push is reported as committed but not pushed.
- The verb publishes whatever `<file>` holds, and runs only from the prime under every orch verb's existing prime-only refusal.
  The role preamble, moved above the two sections, bounds the text: the rules add to the role and override none of it.
- A rules file's own headings start at `###`, since it is inlined under a `##` section.

## Check or run by hand

- Operator step, after this lands and the deploy puts the verb in production: from the prime, draft the moved rules in `.scratch/` and run `lyx orch rules --from <draft>`.
  Board note `orch-rules-move` holds the step and the two rules (Production, Watching runs) verbatim.
  A task pair's call to the verb is refused, so the note is never promoted to a run.
  Until then the role file shows the hub-wide placeholder.
  The moved Production rule predates the Land row's automatic deploy, so whether it still holds is the operator's call.
- Restart the orch after the deploy.
  The first deployed process reseeds the role stencil's board copy without `repository_rules`, so a watcher still on the old binary refuses every role render naming that marker until it re-execs (Linux, at its first restart-safe idle tick) or is restarted with `lyx orch stop`, then `lyx orch start --continue`.
  An operator-edited board copy that keeps `repository_rules` makes the new binary refuse the same way until it is ported or deleted.
- `lyx orch rules` is bounded by the prime-only refusal; `orch-any-worktree` must bound it again when it lets `lyx orch` run outside the prime.
- A pair's weft forks from its parent's, so a new pair's `_lyx/orch/rules.md` is a copy of the parent's own file.
  Nothing reads it today; `orch-any-worktree` and `pair-from-parent` inherit it as a thread's starting rules.
- Check `docs/overview.md`'s orch row and the `ly:cli` index list `rules` consistently.
- Check `internal/orchcli/rules.go` on a push that fails with a rebase conflict: both pushes refuse with the `git pull --rebase` way forward, and confirm re-running the verb after resolving does not loop on the same refusal.
- The integration tier (`lyx gate test --tags integration ./...`) is the pre-push gate for this change.
