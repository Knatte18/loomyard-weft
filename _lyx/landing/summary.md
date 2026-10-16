# Verify tiers from hub config; loom and shed stencil wording fixes

Hub-generic loom and webster stencils named this repository's Go verify tiers, so a .NET or Python hub received them in its plan writer, Plan-Review and implementer prompts.
This change moves the tiers into hub config and fixes the wording defects the stencil intent audit of 2026-10-15 found.

## Verify tiers from `language.yaml`

`language.yaml` gains a second key, `verify_tiers`: an ordered list of strings, each one rule of the hub's verify tiers (their order, which tiers are compiled rather than run, a nested-module form, raw-run limits).
It is Guard-classified like `skills` and outside the module's `LocalKeys`, so a local override cannot set it.
The loader refuses what `skills` refuses plus an entry holding a line break, with the existing way forward (`lyx config language` from the prime).
An absent key or an absent file resolves the empty list.
`languageconfig.Load` now resolves both keys, so an invalid `verify_tiers` refuses every `language.yaml` consumer at spawn.

The stencils render the entries verbatim as bullets, or `none` for the empty list; every surrounding sentence, each fallback included, lives in the stencil:

- `loom-template-plan.md`: the `## verify:` paragraph runs the hub's tiers in the order and form the list gives.
  With no list it builds the code, then runs the tests of every package a card targets.
- `loom-rubric-plan-review.md`: the `Verify coverage` bullet flags a targeted package left unrun and a section out of the list's order, and does not flag a tier the list calls compiled.
  `shedadapters.ReadRubric` fills the key, and its callers (the Bouncer judge and seed rows, the Burler review row) are told the list.
- `webster-body-implementer.md`: the raw-run section is retitled to raw test runs.
  Both implementer value maps (the fork's and the recovery strand's) carry the list.
- `loom-plan-spec.md`'s `## verify model`: names `verify_tiers` and landing config's `publish_verify` instead of Go commands.

The Go-coded mechanics are unchanged: `lyx gate test`, the card and batch gates, the shell guard's raw-go row and the plan gate's `verify-module-wide` and `verify-nested-module` checks.
The stencil sentences that describe what they refuse stay.
`internal/languageconfig/doc.go` carries this hub's rules as its example of a Go hub's value; the config reference docs, `refusal-spec.md` and `docs/overview.md` follow.

## Wording fixes

- Rejection findings are a reviewer's claims, not the operator's.
  `loom-template-rework.md` calls them the findings of the alignment agent or the parent's review, to be weighed against the code and the decision record.
  A finding that would overturn a Decision is planned only after the writer asks its parent to record it with `lyx loom decision add`.
  On a decline, or with no parent recorded, the writer stops the whole rework and writes no `coverage.md`, so the missing output halts the run.
  Any other finding is planned directly.
  The Plan-Review rubric says the same and flags a card carrying a Decision-overturning finding with no `Added after Discussion` entry.
- The shed driver no longer mentions a hand commit or an operator push; a fix to a halted run goes through `reject` or a backward `goto`.
- The discussion advisor reads the draft record to check it, no longer "for its format".
- The plan and rework writers never ask the operator; their only messages are the parent requests their text names.

## Design decisions

- `verify_tiers` is prompt guidance only.
  An entry cannot loosen a Go-coded guard, but an entry calling a tier compiled rather than run lets the plan's `## verify:` section leave that tier unrun.
  That section is also the chained command Publish and Finalize run after a parent merge, so at landing only `publish_verify` bounds the skip.
  Inside the run, the card and batch gates still run the untagged and `-tags integration` tests over the batch's packages.
  The file is board-only, in the same trust class as `publish_verify`.
- The list is prose, not a structured schema.

## To check by hand

- Set this hub's `verify_tiers` with `lyx config language` from the prime, using the example in `internal/languageconfig/doc.go`.
  Until then plan writers here get the neutral rule, may omit the `go vet -tags tmux`/`-tags llm` compile steps, and Plan-Review checks no order.
- The list is read fresh at each spawn, so a mid-run edit of `language.yaml` can make a later Plan-Review round check a plan against rules it was not written to.
- Finding: the test helper in card 4's tests, which re-signs the implementer stencils, may lack wrappers (`internal/websterengine/template_test.go`, `contracts/stencils`); check it by hand.
- Finding: the intents do not state the bound on the tier skip described above; check that the stencils and `doc.go` say it.
- The plan-level `## verify:` command was not run when the plan was executed.
  The new scan test in `contracts/stencils/govalues_test.go` is not in a card's guard `-run` list, so only the unit tier ran it; run `lyx gate test ./contracts/stencils`.
