# Burler lens fan and twill fixer for Discussion and Plan review

Puts the Burler lens fan and a new "twill" fixer into production for Discussion-Review and Plan-Review, so one Burler round reviews and fixes more thoroughly and fewer rounds are needed.
Both are chosen per round, so a hub can pay for them in round 1 only.
Defaults leave today's behaviour unchanged: with nothing set, every round runs solo review and the single fixer.

## What changes

- **Four new fans in the embedded `burler.yaml` template:**
  - `review-discussion` (`interactions`, `existing-paths`, `bound-breaker`, `walkthrough`);
  - `review-plan` (`affected-set`, `guard-reach`, `way-forward`);
  - `fix-discussion` and `fix-plan`, the fixer's self-check lenses (`fix-consistency`, `fix-claims`, `fix-proposal`, plus `fix-way-forward` or `fix-affected-set`).
- **Fork prompts move from Go prose into stencils** under `contracts/stencils/burler/`:
  - the explore step's fork-reading pre-read renders in fanned rounds only;
  - `clusterRulesBlock` survives only as the renderer of the solo and fanned cluster-rule parts;
  - the fanned part drops the off-lens BLOCKING escape.
  A solo round's explore and review steps, and a single round's fix step, render the same bytes as before.
- **Twill fixer:** a round with a fixer fan spawns its fixer with fork subagents authorized.
  The fixer checks its own change set with read-only forks, using the new `burler-fix-self-check.md` and `burler-fix-check-rubric.md` stencils, and records the result in a `## Self-check` section of its fixer-report.
  Its forks are audited like the reviewer's: a fork that writes, runs a mutating git command, calls the Agent tool or is spawned with a name fails the fix half.
  A fork count that differs from the lens count only logs a warning.
  `lyx burler run` takes a `fix-fan` key in its profile.
- **Per-round keys in `loom.yaml`:** `discussion_fan` and `plan_fan` now take a scalar or a list, and the new `discussion_fix_fan` and `plan_fix_fan` take the same shape.
  Entry N serves round N and the last entry every later round; an empty entry means solo review or the single fixer.
  An empty list, or an entry naming an undefined fan, is refused at run start before round 1.
  All four keys stay closed to the local layer.
- **`fan_review` is retired.**
  A fanned round's reviewer, forks included, runs on the segment's own reviewer list.
  `MigrateConfig` drops the key with a reconcile line, and `RetiredKeys` lets an unreconciled layer still load.
- **Round usage record:** the review half now records the round's reviewer fan and fixer fan before spawning, under `fans_recorded: true`.
  The fix half, a resumed reviewer and the Bouncer read those names, so a hub edit reaches only rounds whose reviewer is spawned after it.
  The fixer half's lenses and counted forks are recorded too.
- **Bouncer:** the judge asks for `exclude_lenses` only when the judged round ran a fan and the next round resolves to the same fan.
- **Docs and specs:** `internal/burlerengine/doc.go`, `internal/shedadapters/doc.go`, `internal/burlercli/doc.go`, `internal/configcli/doc.go`, `docs/overview.md` and the refusal-spec rows for the changed and new refusals.

## Design decisions to know

- A Discussion or Plan fix half whose round has no `fans_recorded` record (an old binary's record included) or whose record does not decode archives the round's review, fixer report and acceptance record and refuses.
  The way forward is to `goto` the segment's review row and resume, which spawns a fresh reviewer and records the names.
  The Bouncer never refuses on such a record; it counts the round as having run no fan.
- A fix fan set in both `loom.yaml` and the recipe's BurlerReview profile is refused at construction.
- Webster-Review and Merge-Review get no fixer key and always run the single fixer.
- The hub's own `loom.yaml` is not changed here; the orch sets it after landing.

## Check by hand

- The Discussion and Plan fans and the `fix-proposal` and `fix-affected-set` lenses are unbenched, as are `walkthrough`, the widened `existing-paths` and the fork prompt without today's Go boilerplate.
  The reviewer forks' 6-call cap against `affected-set`, and the fixer forks' cap against `fix-affected-set`, are unmeasured.
- A hub whose `burler.yaml` defines a lens named like a new one (`interactions`, `walkthrough`, ...) shadows the new text through the per-name fallback.
- Each of these review findings was fixed or disputed and no fresh reviewer has seen the result:
  - the fixer-lens-list sentence in the design record may overreach (`fixer-lens-list-sentence-overreach`);
  - the usage record is authoritative without a hard write on every path (`usage-record-authoritative-without-hard-write`);
  - the fixer's fork cap against `fix-affected-set` (`fixer-fork-cap-vs-fix-affected-set`).
- The design record also lists findings still open: absent recorded names can loop the refusal (`d2-absent-names-refusal-loops`), the Bouncer's tolerant reading against the fix half's refusal (`d2-bouncer-refusal-vs-tolerant-reading`), and the pre-spawn write against a live-reviewer resume (`d2-prespawn-write-vs-live-resume`).
  Confirm the code in `internal/shedadapters/burlerreview.go` and `burlerfix.go` handles each.
- Codex: a twill fixer needs forks, so a hub moving a Discussion or Plan fixer to an engine without forks sets that fixer fan to `""`.
- Not run by hand: a real Discussion or Plan run with the fan and twill fixer in round 1.
