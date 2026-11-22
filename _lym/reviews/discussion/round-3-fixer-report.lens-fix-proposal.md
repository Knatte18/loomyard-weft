```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F12 proposal that the fixer carried out, by building cases where it is wrong.

**LOW: the check treats any `beam-ready.json` as the flag, but lym also checks that the record names the beam's slug**
- Changed text: D4 step 2, "the beam qualifies only when that worktree holds no `_lym/landing/beam-ready.json`" and "A file found there means the beam does not qualify."
- Evidence: `ReadyFor` in `internal/pawlengine/ready.go:35-48` says "An absent record, and one naming another slug, read as not flagged" (`return found && record.Slug == slug`). `MarkReady` in `internal/beamcli/deps.go:242` uses that test.
- Concrete case: the beam's records branch carries a `beam-ready.json` whose `slug` names another beam. Records content is per-branch, so a records branch forked from another one can hold such a file. lym reads this beam as not flagged, but the procedure refuses the move.
- Impact: the error only refuses a move that would have been allowed. It never lets a flagged beam through.
- Fix: say the beam is flagged when the record exists and its `slug` is the beam's slug. This applies to both the local read and the `git show` read.

No other case breaks the proposal:
- A records branch that was never pushed makes `git show` fail on the missing ref. The procedure treats that as an absent file and asks the beam's orch or the operator, which is safe.
- The read through `origin/<beam branch>-weft` works in both hub layouts, given the one-repository refspec.

Tool calls:
- Ran a grep for `ReadyFor`/`WriteReady` in `internal/pawlengine` to learn what lym counts as the flag. My context showed only the `ReadyFor` call in `MarkReady`, not its body.
