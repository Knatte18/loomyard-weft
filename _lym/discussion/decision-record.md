## Goal

```markdown
## Seen (operator and hub orch, 2026-11-04)

Seven tasks were released together on the loomyard prime; five were ready at once and dobby (max_runs 8) started all five.
The operator found the start very slow.
Over about three minutes the five runs left Worktree-Create one after another, never together:

- `beam-pr-lifecycle` reached Seed-IDE first, while the other four still sat in Worktree-Create.
- `dobby-zombie-children` then sat at Open-Window for over a minute.
- `shuttle-launch-delivery` left Worktree-Create about two minutes in; `drift-capture-and-migration` and `drive-reports-untracked` were still in Worktree-Create after that.

Meanwhile `lym tender status` showed network operations in flight only (several `remote-refs` and a `fetch` per slug, none under a parent lock), and a `lym fabric … push` of the prime's weft for `beam-pr-lifecycle` ran over two minutes, its `git send-pack` to `loomyard-weft.git` alone about a minute.

At the same time githubstatus.com reported Git Operations and API Requests as `degraded_performance` (minor outage); a `git ls-remote` took 0.9 s, while a branch push of `shuttle-launch-delivery` was refused with `Internal Server Error` and its retry hung over two minutes.
A create normally takes about 10 s, so the slowness is mostly GitHub's push path; the note stands for how a slow or failing push stalls every create behind it.

## Idea

- A create does not wait on a push: a branch push that is slow or refused runs and retries in the background, so a GitHub outage delays the remote copy, not the run.

- Find what serializes Worktree-Create across runs (a fabric lock, the weft push, or the network ops), and let independent creates run side by side.
- Cut the network round trips per create: one fetch and one remote-refs read per create, and no synchronous weft push of the prime on the create path.
- Find why Open-Window takes over a minute.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-04): The create change stops at the prime lock's span

Decision: This run delivers the narrowed prime lock: independent creates no longer wait on each other's network calls. Left unmade, and accepted: the board claim's Bolt push stays synchronous (it is the owner-machine claim), the weft remote-refs lock queue goes to note push-holds-remote-refs, merging the two Bolt round trips is not done, and the Open-Window delay is not diagnosed. summary.md's Known limits already state these; nothing more is asked of the fixer.

Rationale: The serialization across runs was the fault the operator saw; the remaining items are either required for correctness (the synchronous claim) or need their own design, so they stay out of this darn.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
