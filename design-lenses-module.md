# Lenses and fans move into a lenses module of their own, listed in full in the hub file

`lenses-module` · Task · enhancement, burler, loom, config · running · Discussion-Burler-Fix

## Seen (2026-10-19)

Every lens and fan lyx has lives in burler's template, `internal/burlerengine/template.yaml`: the reviewer fans (`review-discussion`, `review-plan`, `review-code`), the fixer fans (`fix-discussion`, `fix-plan`, `fix-code`), the writer twill fans (`twill-discussion`, `twill-plan`) and the older `standard`, `full`, `discussion` and `plan`, each lens with its prose.
The twill fans are no part of burler: loom's `discussion_lenses` and `plan_lenses` name a fan in `burler.yaml` (`internal/loomengine/config.go`), and the Discussion-Write and Plan-Write leads fork once per lens of it.

The hub `burler.yaml` on Bolt is a copy that `lyx config reconcile` wrote on 2026-10-12: it lists the lenses and fans the template had then (`generic`, `correctness`, ..., fans `standard` and `full`).
Every lens and fan added to the template since is absent from the hub file and reaches runs only through the per-name merge with the template; `lyx config burler --resolved` marks them `# template`.
`lyx config reconcile --layer default` reports burler with `added: null` and `removed: null`: reconcile does not carry a new lens or fan into a hub file.
So a fan names lenses the hub file never defines, and a lens cannot be tuned in the hub file without copying its prose in by hand.

The hub `burler.yaml` overrides `twill-discussion` today with seven lenses (Bolt `3f3286b8d`, operator decision recorded in `writer-reviewer-lens-align`); as a hub-only override it is a trial nobody will remember.

## Want (operator decisions, 2026-10-19)

- Lenses and fans move out of burler into a module of their own, `lenses`, with its file `lenses.yaml`; they are never in `burler.yaml`.
  Burler's reviewers and fixers and loom's twill writers all read their lenses and fans from it.
- The template's `twill-discussion` fan becomes the seven lenses `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `goal-scope`, `board-fit`, `leanness`, so the choice is the default and no longer a hub-only trial.
- A hub's `lenses.yaml` lists every lens with its prose and every fan, so any lens or fan can be tuned there.
  Reconcile carries a lens or fan the template adds into the hub file, and reports one the template drops, as it does for the keys of other modules.
- A hub `burler.yaml` that still holds `lenses` or `fans` is migrated: its entries move into the hub `lenses.yaml`.
- After landing, the orch runs the reconcile on the loomyard hub, so its `lenses.yaml` holds the full current set and its `burler.yaml` holds no lens or fan.
