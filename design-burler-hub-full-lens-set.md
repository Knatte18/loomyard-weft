# Hub burler.yaml lists every lens and fan; the Discussion writer fan moves into the template

`burler-hub-full-lens-set` · Task · enhancement, burler, config

## Seen (2026-10-19)

The hub `burler.yaml` on Bolt is a copy that `lyx config reconcile` wrote on 2026-10-12: it lists the lenses and fans the template had then (`generic`, `correctness`, ..., fans `standard` and `full`).
Every lens and fan added to `internal/burlerengine/template.yaml` since (`interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, the `twill-*` and `review-*` fans, and the rest) is absent from the hub file and reaches runs only through the per-name merge with the template.
`lyx config burler --resolved` marks them `# template` and the copied ones `# default`.
`lyx config reconcile --layer default` reports burler with `added: null` and `removed: null`: reconcile does not carry new lens or fan names into a hub file.
So the hub file is misleading: a fan names lenses the file never defines, and a lens cannot be tuned in the hub file without first copying its prose in by hand.

The hub file overrides `twill-discussion` today with seven lenses (Bolt `3f3286b8d`, operator decision recorded in `writer-reviewer-lens-align`); as a hub-only override it is a trial nobody will remember.

## Want (operator, 2026-10-19)

- The template's `twill-discussion` fan becomes the seven lenses: `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `goal-scope`, `board-fit`, `leanness`, so the choice is the default and no longer a hub-only trial.
- A hub's `burler.yaml` lists every lens with its prose and every fan, the way it lists `generic` and `standard` today, so any lens or fan can be tuned there later.
  Reconcile carries a lens or fan the template adds into the hub file, and reports one the template drops, as it does for the keys of other modules.
- After landing, the orch runs the reconcile on the loomyard hub so its `burler.yaml` holds the full current set, with its `twill-discussion` entry then equal to the template's.
