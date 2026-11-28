# Fork audit names the broken git rule; a review halt names a way forward

A cluster review round halted on a fork that experimented with git in a throwaway repository of its own.
The fork ran an `ln`, then pushed from that repository.
The audit counted the push, because every push after an `ln` counts (the link may lead out),
but the message said the fork ran git "outside a repository it created", which the push did not.
A reader of the halt could not tell why the command counted, and the halt named no way forward.

## What changes

- `mutatingGitCommands` now records, for each command it counts, the rule its first counted invocation broke.
  The rules are named constants in `internal/burlerengine/cluster.go`:
  a state-changing command outside every created repository,
  a push after `ln`,
  a push under or after a push-redirecting config key,
  a push after a `git remote` change in an unresolvable directory,
  a push with `--repo`,
  a push with `-u`, `--set-upstream` or `--recurse-submodules` from outside,
  a push with no destination,
  a push to a remote not pointed into a created repository,
  and a push to a URL or path outside every created repository.
  The single `pushRedirected` flag became `pushRedirect`, the rule of the first call that made every later push count.
- The violation message reads `ran a git-mutating command (...), counted as <rule> — ...`,
  so the incident's message now says `counted as a push after ln, since a link may lead out of every repository the fork created`.
- `auditClusterRound` appends the way forward to a review round's violation, as `auditFixRound` already did for a fixer's self-check.
  `fixViolationWayForward` is renamed `violationWayForward`, since both paths use it.
  For the incident it says no target change was found, names the repository the push came from,
  and asks the reader to check that the destination is neither the target's repository nor one of its remotes before resuming.
- The fork briefs in `burler-cluster-fan.md` and `burler-fix-self-check.md` now state the rule the audit enforces.
  A fork never uses a write or edit tool, never changes a file of the target, and runs only reading git commands on any repository it did not create.
  It may change state in a throwaway repository it creates with `git init`, `git clone` or `git worktree add`.
  A push from that repository may go only to another such repository, by path or by a remote the fork pointed there,
  and after any `ln` or push-redirecting config key every push breaks the rule.
- The package doc's fork-discipline section names the rule-carrying message, the review-path way forward and the `ln` rule.

## Design decisions

- The brief states the allowance, with its limits, rather than forbidding throwaway repositories.
  Checking how git behaves on a scratch repository is legitimate evidence for a review of git-handling code, and the audit already permits it.
  The brief now matches the audit instead of forbidding more than the audit enforces.
- A violation that left the target untouched still halts the run rather than rerunning the round.
  For a push the audit could not place, as in the incident, "untouched" covers only the target's working tree:
  the push may have reached one of the target's remotes, which only a person can check.
  The way forward tells them what to check.
  Violations with no push at all, a nested spawn or a named fork, break the fork contract itself and stay fail-loud.
- The `ln` rule itself is kept: a link can lead a push anywhere, and the audit resolves paths lexically.
- The assistant fan's brief (`burler-assist-cluster-fan.md`) is unchanged, since no fork audit covers an assistant's subagents.

## Tests

- `TestMutatingGitCommandsNameTheBrokenRule` pins the rule named for each kind of counted command,
  including that the first call that makes every later push count wins over a later `ln`.
- `TestAuditClusterRound` gains a regression row with the incident's command sequence.
  It asserts the message names the push-after-`ln` rule and the review error ends with the no-target-change way forward.
  The review-path commit row now asserts its rule and the revert way forward.
  Every row's way forward is checked on both the review and the fix paths.

## Merge danger

**Door:** two-way
**Blast radius:** burler
