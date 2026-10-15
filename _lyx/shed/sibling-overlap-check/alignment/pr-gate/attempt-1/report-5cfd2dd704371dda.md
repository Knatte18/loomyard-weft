---
verdict: APPROVED
---

## Alignment at PR-Gate: sibling-overlap-check, PR 649 at adecac96f

### Against the board entry's goal

The board entry asks for two parts: `set-deps` and `upsert` warn when a new `depends_on` makes a task a sibling of another task sharing a module label, naming the sibling and the shared labels while the write succeeds; and the `ly:board` skill gains the rule to check siblings, not only running tasks, beside its `depends_on` guidance.

The diff delivers both.
`internal/boardengine/siblings.go` adds `addedSiblings`, computed in `writeReportingSiblings` inside the write's critical section, and the exported `SharedModuleLabels` for later reuse by note `batten-ready-overlap`.
`internal/boardcli/cli.go` formats the pairs into the envelope's `notice` on `set-deps`, `upsert`, `upsert-batch` and `merge`, joined after any run-settings notice; a refused write returns its error with no siblings.
`plugins/ly/skills/board/SKILL.md` gains the sibling rule in the `depends_on` paragraph and restores the missing line break before "A relation between notes".
Path-based overlap and batten's per-group start stay out, as the entry rules.

### Against the decision record

- D1: the baseline is the task's own loaded `depends_on` plus the `depends_on` of every entry the write removed; done written tasks, done dependencies, done or abandoned siblings and type labels are excluded; running siblings warn.
- D2: one engine function under the write lock, the overlap test exported with no CLI formatting, `boardcli` formats.
- D3: `notice` reused, one line per pair, a batch pair once with both sides' dependencies; `set-deps` gains `notice` only when there is a warning.
- D4: the upsert family and `set-deps` covered; `promote` and `intake import` unchanged.
- D5: the skill rule as recorded.
- Docs land in the same change: the four verbs' `--help`, `internal/boardcli/doc.go` and a `# Siblings` section in `internal/boardengine/doc.go`.
- The acceptance criteria each have a row in the table-driven `internal/boardengine/siblings_test.go`, including the refused-write rows.

The Discussion-Review carry-over `d1-no-status-condition-on-written-task` is resolved in the record (D1 now requires X's status after the write not to be `done`) and in the code (`addedSiblings` skips a done X, and the test has rows for a done written task and for a merge whose `set_status` marks it done).
The two open risks (hand-set labels miss overlaps; a broad label may over-warn) are accepted as advisory by the record and need no change here.

No change goes beyond the goal; no goal point or recorded decision is missed or broken.

### Overlap with peers and parent commits

`lyx loom peers` lists no parent commits; `own_row` is 22.

- `engine-registry` (row 12): shares only `cmd/lyx/testdata/test-budget.yaml`, a bookkeeping file, at different rows. No design overlap. Landing order: this run (row 22) lands first; `engine-registry` takes the change at Merge-In.
- `creel-core` (row 12): shares only `cmd/lyx/testdata/test-budget.yaml`, at different rows. No design overlap. Landing order: this run lands first; `creel-core` takes the change at Merge-In.
- `plan-test-file-path` (row 12): no changed paths in common. No overlap.

### Board entries made obsolete

None.
`batten-ready-overlap` and `labels-from-diff` are named out of scope by the entry and stay open.
