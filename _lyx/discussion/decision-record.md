# Decision record — sibling-overlap-check

## Goal

When a board write gives a task a new `depends_on` entry that another open task already waits on, and the two share a module label, the write still succeeds and its envelope warns, naming the sibling, the shared dependency and the shared labels.
The `ly:board` skill tells every agent that sets `depends_on` to check siblings, meaning other tasks waiting on the same entry, and not only running tasks.

## Scope

In:

- A sibling check in `internal/boardengine`, run inside the write's critical section against the board as it stands after the write.
- The warning on every verb that writes `depends_on` through the upsert fields or `set-deps`: `set-deps`, `upsert`, `upsert-batch` and `merge`.
- The sibling rule in `plugins/ly/skills/board/SKILL.md`, beside its `depends_on` guidance, naming the warning.
- `--help` text of the four verbs and `internal/boardcli/doc.go` describe the warning, and `internal/boardengine/doc.go` describes the check.

Out:

- Path-based overlap, which waits for note `labels-from-diff`.
- Batten starting one task per overlapping group at a newly-ready start, which is note `batten-ready-overlap`.
- Any refusal: the check never blocks a write.
- A copy of the rule in the orch role's stencils, since the orch loads `ly:board`.
- Warnings on a write that changes only labels, or on `list`/`get`.

## Decisions

### D1 — Sibling definition

A sibling of task X for a write is a task Y with:

- Y is not X, and Y's status is not `done`;
- Y's `depends_on` holds a dependency D that this write added to X's `depends_on`, and D is not `done`;
- X and Y share at least one module label.

A dependency is added when X's `depends_on` after the write holds it and the baseline did not.
The baseline is X's stored `depends_on` before the write; for a new entry it is empty.
For `merge`, the baseline is the union of the stored `depends_on` of the target, if it existed, and of every entry in `remove_slugs`, so folding entries into one task warns only on dependencies none of them had.
An `upsert` without the `depends_on` key leaves the list unchanged and so adds nothing.
A write that only adds a shared module label to a task already waiting on D adds no dependency and stays silent.
A module label is a label the board vocabulary does not list under `types`, so `bug` or `enhancement` alone never makes a sibling.
X's labels are the ones the write leaves on it.

Rationale: the board entry scopes the warning to a new `depends_on`.
Comparing against the baseline keeps a write that touches only a body or title, or re-sends the same `depends_on`, silent.
A done task waits on nothing, and a done dependency holds no one back, so neither is the incident.
Type labels say nothing about which code a task edits.
Running siblings stay in: a running task waiting on the same entry still collides when the entry lands, and the warning is cheap.

### D2 — One engine function, computed under the write lock

`boardengine` computes the siblings inside the same critical section that validates and writes the change, from the board as it stands after the write, and returns them to the caller with the write's result.
An entry a `merge` removes is never a sibling.
The overlap test (two tasks share a module label) is one exported `boardengine` function with no CLI formatting, so note `batten-ready-overlap` can reuse it.
`boardcli` formats the result.

Rationale: the sibling set is a fact about board state, so it is read under the same lock as the write.
A batch sees its own members as siblings of each other, since two tasks in one `upsert-batch` both newly waiting on D are the incident in miniature.

### D3 — Warning text in the envelope's `notice` field

The warning goes into the success envelope's existing `notice` field, one line per sibling pair, joined by newlines to any run-settings notice the same write produces.
`set-deps`, which today prints a bare `{"ok":true}`, gains the same `notice` field when there is a warning.
A line names X, the sibling Y, every shared dependency once, and the shared module labels, and ends with the way forward: chain one after the other with `lyx board set-deps`, or leave both if they do not overlap.
In a batch, a pair whose both members newly wait on a shared dependency is reported once.
With no sibling, the envelope carries no `notice` for it, so the output is unchanged.

Rationale: `notice` is already the board envelope's channel for a write that succeeds with something to say (`internal/boardcli/doc.go`), and agents read the JSON envelope, not stderr.
The bound: the warning changes no exit code, no refusal and no stored record; it only adds text to a success envelope.

### D4 — Verbs covered: the upsert family and `set-deps`

`upsert-batch` and `merge` warn too, since each writes the same upsert fields `upsert` does.
`promote` and `intake import` do not write `depends_on` and stay unchanged.

Rationale: the board entry names `set-deps` and `upsert`; leaving the two other routes to the same field silent would let the incident recur through them, which defeats the entry's goal.

### D5 — The skill rule

`plugins/ly/skills/board/SKILL.md` gains, in its `## Writing a body` paragraph on `depends_on`, a rule: whenever you set `depends_on`, check the other tasks waiting on the same entry for overlap, not only the running ones, and act on the warning `set-deps` and the upsert verbs print by chaining overlapping tasks.
The same edit restores the missing line break before "A relation between notes" in that paragraph.

Rationale: the board entry places the rule in the skill every board writer loads; the warning catches a module-label overlap, and the rule covers overlap the labels miss.

## Constraints

- `PATTERN-test-economy` and `PATTERN-test-speed`: extend the existing `boardengine` and `boardcli` tests where they cover these verbs; the no-warning cases are rows of one table-driven test; engine cases are unit tests with no git.
  No test pins the skill's wording.
- `PATTERN-comment-line-breaks`: Go comments one sentence per line; the skill edit keeps semantic line breaks.
- CLAUDE.md's docs-in-the-same-commit rule: the verbs' `--help`, `internal/boardcli/doc.go`, `internal/boardengine/doc.go` and the skill change in the same commit as the code.
- CLAUDE.md's test budget: a card whose new top-level tests push `boardengine` or `boardcli` past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- D4: the upsert family includes `upsert-batch` and `merge`; the board entry names only `upsert`.
- D1: done siblings and done dependencies are excluded, running siblings included.
- D1: only dependencies added against the baseline trigger the warning.
- D3: the warning reuses `notice` rather than a new key.

## Open risks

- Module labels are set by hand and miss touched packages (note `labels-from-diff`), so the warning misses overlaps the labels do not show; the skill rule (D5) covers that gap by asking the writer to look.
  When that note derives labels or paths, this check is a natural consumer of them.
- A board with many tasks on one popular dependency and a broad label such as `stencil` may warn on pairs that do not overlap in code; the warning is advisory and names the labels, so the writer judges it.

## Acceptance criteria

- `set-deps` adding D to X, where open task Y waits on D and shares module label L with X: succeeds, stores the list, and the envelope's `notice` names X, Y, D and L.
- The same through `upsert`, `upsert-batch` and `merge` carrying `depends_on`.
- No warning, and no `notice` key from the check, when: X and Y share only a type label; Y is done; D is done; Y is X; D was already in X's `depends_on` before the write; an `upsert` on X omits `depends_on`; the write clears `depends_on`; no other task waits on D.
- A running sibling Y still warns.
- `upsert` that also sets labels uses the labels as written.
- `upsert-batch` writing X and Z both newly on D with a shared module label reports the pair once.
- A `merge` that removes Y names no Y; a `merge` folding A, which waited on D, into new task B does not warn on D.
- A `depends_on` listing D twice names D once.
- A write that carries both a run-settings notice and a sibling warning shows both in `notice`.
- A refused write, such as a note given `depends_on` or an unknown dependency, refuses exactly as today, with no warning.
- The `ly:board` skill states the sibling rule beside its `depends_on` guidance and names the warning.
