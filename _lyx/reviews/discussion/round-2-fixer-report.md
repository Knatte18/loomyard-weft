# Fixer report — discussion, round 2

## Changes

- F1 (BLOCKING), fixed as proposed.
  Premise checked: `Describe` has `on_done: Merge-In` after Webster in `contracts/recipes/loom-recipe.yaml`, `tools/` holds no `forkaudit`, and `scopeChanged` in `internal/boardengine/runlock.go` compares `DependsOn`.
  `decision-record.md`: deleted D7, the Scope In bullet on `tools/forkaudit` and the forkaudit acceptance criterion; added a Scope Out bullet naming the card arm (marker row ahead of `file`, `card` trace value, span source from `card-pack` results) as a follow-up board task, filed by the orch, whose `depends_on` names `glyph-audit-tool` and `kickstart-per-card`; Open risks keeps only the interim `file` classification with a trace disagreement.
  `support-log.md`: added the Rejected alternatives bullet the fix names, carrying D7's rationale (marker row ahead of `file` because of the shared `## Start here` heading) for the follow-up's body, and a round-2 entry under Review rounds.
  The follow-up task itself is the orch's to file; D7's full bullets stay readable in this record's previous revision.
- F2 (MEDIUM), fixed as proposed.
  Premise checked: `armDurableSinkLocked` in `internal/logger/sink.go` creates the directory, a per-process trace file and a header line.
  D3's Bound now names the durable sink, trace file and header, the Info line, the Warn on quarry failure and the exit sweep, and keeps the no-state, no-lock bound.
- F3 (LOW), fixed as proposed.
  Premise checked: line 60 of `contracts/stencils/webster/webster-body-implementer.md` ends the always-allowed list with `lyx loom lint-pattern` and `lyx quarry resolve`.
  D4's sandbox sentence now places the marker after `lyx loom lint-pattern`, before `and`, pins its fill, and drops the deferring parenthetical.

## Disputed

None.

## Deferred

None.
