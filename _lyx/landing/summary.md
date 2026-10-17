# Lens fan and twill fixer for Webster, Merge and Darn review

Webster-Review, Merge-Review and Darn-Review can now run the Burler lens fan and the twill fixer per round, as Discussion-Review and Plan-Review already could.
The recorded reason they stayed solo was that fork reviewers may run no git, so they could not reach a subject that exists only through git.
This change lifts that ban for read-only git and adds the keys and lenses to use it.
Every template default is empty, so no run changes until a hub sets a key.

## What changes

- New per-round keys in `loom.yaml`: `webster_fan`, `webster_fix_fan`, `merge_fan` and `merge_fix_fan`.
  New keys in `darn.yaml`: `review_fan` and `review_fix_fan`, matching darn's existing `review_*` budget keys.
  Each takes one fan name or a per-round list whose last entry serves every later round, and an empty value runs solo with the single fixer.
  Each is `CategoryGuard`, closed to the local layer like the four existing fan keys.
- The wiring in `internal/loomcli` adds the Webster, Merge and Darn rows to `RowClusterFans`, `RowFixFans` and `RowModelKeys`.
  Merge and Darn rows use Webster's review and fix model keys, which their models come from today.
- The fork stencils `burler-cluster-fan.md` and `burler-fix-self-check.md` now permit read-only git (`git diff`, `git log`, `git show`) and still forbid any command that changes the repository, index, refs or working tree.
  The engine's fork audit is unchanged and keeps failing a fork that runs a state-mutating git subcommand.
- The embedded `burler.yaml` template gains `review-code` (lenses `code-correctness`, `code-affected-set`, `code-tests`, `code-pattern-fit`) and `fix-code` (lenses `fix-code-correctness`, `fix-consistency`, `fix-proposal`, `fix-affected-set`).
  Each review lens names the rubric item it emphasizes by heading, without paraphrasing it.
  The texts are a first draft for later audits to tune.
- The recipe comments, `doc.go` files, config templates and `docs/overview.md` no longer say these three perches always run solo or that forks may run no git.

## Design decisions

- The fan grammar (`FanList`, its shape error and per-key fan validation) moves from `loomengine` into `internal/burlerengine`, because `darnengine` cannot import `loomengine`.
  Validation is told its module name, so each refusal names `loom config key` or `darn config key`.
  The lookup that names a malformed value's key moves to `configengine`, since both loaders need it.
- Merge-Review and Darn-Review get their own fan keys rather than reading Webster's, so each perch is configured separately.
- A darn fix fan key is named with its file (`review_fix_fan in darn.yaml`) in refusals, because its model keys live in `loom.yaml`.
- With a fan entry present, the review half records each round's fan names before spawning, and the fix half refuses a round whose usage record lacks them, as Discussion and Plan rounds do.
  The way forward is a `goto` to the segment's review row and a resume.
  This newly reaches a Webster, Merge or Darn round reviewed by an older binary and fixed by this one.
- The fan-conflict refusal in `burlerReviewEntry` no longer names `loom.yaml`, since a darn row's list comes from `darn.yaml`.

## Risks

- The read-only git permission rests on the stencil text and the audit's mutating-subcommand pattern; a command that looks read-only but mutates state, and falls outside that pattern, would pass the audit.
- Forks running `git diff` over a large range spend more of their tool-call cap than Discussion and Plan forks do; the cap is unmeasured for code review.
- A hub `burler.yaml` that defines a lens with the same name as a new one shadows the new text through the per-name fallback.

## To check by hand

- Set the new keys in a hub's `loom.yaml` and `darn.yaml` and run a Webster, Merge and Darn review round with a fan, then a fix round, to see forks read the diff through git and the fixer's fan work end to end; no real run was made.
