# board: warn when a new depends_on makes a task an overlapping sibling

Four tasks once waited on the same entry, three of them editing the same files, and a newly-ready start would have launched all four together.
The orch checked a new task only against running ones, never against other tasks waiting on the same entry.

A board write that gives a task a new `depends_on` entry now warns when another open task already waits on that entry and shares a module label with it.
The write still succeeds, stores the same record and exits the same way; only the success envelope gains text.

## What changes

- `internal/boardengine/siblings.go` computes the siblings inside the write's critical section, from the board as it stands after the write, and returns them with the write's result.
  `SetDeps`, `UpsertTasksBatch` and `MergeTasks` return them.
- `SharedModuleLabels` is one exported function with no CLI formatting, so the later batten ready-overlap work can reuse it.
- `internal/boardcli` writes one line per sibling pair into the envelope's existing `notice` field, joined by newlines to any run-settings notice the same write produces.
  Each line names the task, the sibling, the shared dependencies and the shared module labels, and ends with the way forward: chain one after the other with `lyx board set-deps`, or leave both if they do not overlap.
  `set-deps`, which printed a bare `{"ok":true}`, gains `notice` only when there is a warning.
- The warning covers `set-deps`, `upsert`, `upsert-batch` and `merge`.
  `promote` and `intake import` do not write `depends_on` and are unchanged.
- `plugins/ly/skills/board/SKILL.md` gains the rule, beside its `depends_on` guidance: whenever you set `depends_on`, check the other tasks waiting on the same entry, not only running ones, and act on the warning by chaining overlapping tasks.
  The same edit restores a missing line break before "A relation between notes".
- The four verbs' `--help`, `internal/boardcli/doc.go` and `internal/boardengine/doc.go` describe the warning and the check.
- `cmd/lyx/testdata/test-budget.yaml` raises the rows for the packages whose tests grew.

## Design decisions

- A sibling Y of task X exists when Y is open (not `done`, not `abandoned`), X is not left `done` by the write, Y waits on a dependency the write added to X, that dependency is not `done`, and X and Y share a module label.
  A running Y still counts, since it collides when the entry lands.
- Only dependencies added against the stored baseline trigger a warning, so a write touching only a title or body, re-sending the same `depends_on`, or adding a shared label to a task already waiting stays silent.
  For `merge` the baseline is the union of the target's stored `depends_on` and that of every removed entry, so folding tasks together warns only on genuinely new dependencies.
  A removed entry is never a sibling.
- A module label is any label the board vocabulary does not list under `types`, so `bug` or `enhancement` alone never makes a sibling.
- Within an `upsert-batch`, two members that both newly wait on a shared dependency are reported once, and the relation is symmetric.
- Refused writes refuse exactly as before, with no warning.
- The check does not refuse anything, by design.

## Known limits

Module labels are set by hand, so the warning misses overlaps the labels do not show; path-based overlap waits for the `labels-from-diff` note, and the skill rule covers the gap by asking the writer to look.
A broad label such as `stencil` on a popular dependency can warn on pairs that do not overlap in code; the warning names the labels so the writer judges.

## To check by hand

- Run `lyx board set-deps` in a scratch board to add a dependency that another open task with a shared module label already waits on, and read the `notice` in the envelope.
  Confirm the output is unchanged when only a type label is shared.
- The `lyx board upsert` `Long` help was trimmed twice to meet the 2000-byte cap in `cmd/lyx.TestCLIForm_EveryCommand`; read the sibling-notice help line in the four verbs for clarity.
