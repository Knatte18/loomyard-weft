# Card 15 — one-repository-docs

**Prosa:**
- `README.md`
- `docs/getting-started.md`
- `docs/overview.md`

**Intent:** D2's and D7's docs, and the transition seam's wait (card 11) where the docs described its skip.

`docs/getting-started.md`, in `## Build and make a hub`, describes cloning a one-repository hub:
`lym fabric clone --shortname <short> --single-repo <url>` for a repository with no hub yet, and `lym fabric clone <url>` for one whose origin already holds `lym-bolt`.
It says what the layout puts on code branches: `.lym-anchor` at the root and the records under `<anchor>/_lym`, committed beside the agents' work and stripped at every fabric merge, and the board on the orphan branch `lym-bolt`.
It warns that `--single-repo` cannot recognise the code repository of an existing two-repository hub and, run there, starts a separate hub over the same code.
It warns that switching, resetting or stashing in the prime holds or drops the prime's live records:
a prime off the branch `lym.primeBranch` names refuses every records commit until the switch back, and a `git reset` or `git stash` can drop uncommitted records.
It says a prime whose branch refuses direct pushes keeps its records unpushed, and that the prime's records push waits while the operator has unpushed commits on that branch.

`README.md`'s run table and `## Names at a glance` name the one-repository layout beside the records repository: the records on each code branch and the board on `lym-bolt`.

`docs/overview.md`, where it says the status commit skips while mid-merge, says it waits until the blocking git operation ends and then commits.
