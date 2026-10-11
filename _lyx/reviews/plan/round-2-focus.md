---
round: 2
focus:
  - "Card 7 (`_lyx/plan/07-begin-batch-builds-pack.md`): settle whether any failure path of `buildKickstartPack` in `internal/websterengine` refuses `begin-batch`, and if one does, whether `contracts/specs/refusal-spec.md` has its row, a named input reaches it, and a named `TestBeginBatch_KickstartPack` row covers it; `PATTERN-refusal-way-forward` binds this."
  - "Card 7 `TestBeginBatch_KickstartPack` rows: settle whether each row's fixture reaches `buildKickstartPack`, given that it reads cards from a fresh `planparser.ParsePlan` of `deps.Plan.Dir` and that `ValidateDispatch` runs first; check each row's plan on disk and worktree code against that."
  - "Card 4 (`_lyx/plan/04-bind-trace.md`) no-line rows: settle whether each named failing input reaches its branch in `recordbatch.go`, in particular the `BindHandles`-error row, given how `planparser.RewriteRefs` writes card files and that it runs only when substitutions exist."
  - "Cards 3 and 4: settle whether `planparser.OwnedHandles`, an exported symbol used from `planglyph` and `websterengine`, has its own card or is bundled with the `BindOutcome` change, per the granularity rule."
  - "Card 6 (`_lyx/plan/06-pack-stencils-and-render.md`): settle whether every `Edit` glyph whose signature changes carries the re-sign arrow, and whether every test member that calls a changed function is listed under `Edit:`."
---

Focus follows the open findings of round 1 without pre-judging them.
