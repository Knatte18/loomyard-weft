---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: _lyx/plan/01-engine-siblings.md (Intent, TestWriteSiblings seed sentence and refused rows)
    summary: Card 1 says every seeded entry X through E is a kind task so no seed hits a note refusal, yet its first refused row needs a note as the set-deps subject
    fix: In card 1's Intent, change the sentence "Every seeded entry, X, Y, Z, A, B, C, D and E, is a `"kind":"task"` carrying the type label `bug`, plus the module labels a row names, so no seed hits a note refusal." to say that every seeded entry is such a task except the note N of the note refused row, which is seeded as a `"kind":"note"` carrying `bug` and given `depends_on` [D] by set-deps; and in the refused-rows bullet name that subject N instead of leaving it implicit.
---

### [NIT] Note refused row contradicts the all-tasks seed rule

**Location:** `_lyx/plan/01-engine-siblings.md`, Intent, the `TestWriteSiblings` seed sentence and the refused-rows bullet.

**Issue:** The card states that every seeded entry, named X, Y, Z, A, B, C, D and E, is a `"kind":"task"` so that no seed hits a note refusal.
The refused-rows bullet then prescribes "set-deps giving a note `depends_on`", which needs a seeded note as its subject, and `Store.SetDeps` reaches the note refusal in `validateWrite` (`internal/boardengine/store.go:333`) only when that subject is a note.
The row stays reachable, but an implementer reading the seed sentence literally would seed the subject as a task, and the row would then succeed instead of refusing.

## Review notes

Judged against `_lyx/discussion/decision-record.md`; no rework round exists (`_lyx/loom/rework/` holds no round directories) and `.lyx/webster/plan-conflict.md` is absent, so the decision record alone is the answer key.

Focus items, each settled:

- `TestWriteSiblings` as a new top-level test: justified.
  `TestSetDeps`, `TestUpsertTasksBatch` and `TestMergeTasks` build `boardengine.NewStore("")` in every subtest (`internal/boardengine/store_test.go:816-1000`, `:525`), so they never pass through `Board` or `boardCriticalSection`, where the siblings are computed.
  `TestRunLock` is the only Board-level table over these verbs and pins run-lock refusals, a different behavior.
  The refused rows assert only a non-nil error (or `errors.Is(err, ErrRunLocked)`) and nil siblings, leaving message text and rollback to the tests the card names, so no row repeats them.
  The budget row for `internal/boardengine` is raised by one in the same card.
- Refused rows reach their branches: the note row and the unknown-dependency row fail inside `fn` in `Store.SetDeps`'s `validateWrite`; the run-lock row passes `fn` (no store validation looks at run status) and is refused by `checkRunLocks` (`internal/boardengine/board.go:104`) after the mutation, the same path `TestRunLock`'s "set deps" row reaches.
  No upstream check rejects any of the three inputs earlier.
- `addedSiblings` against D1 and D2: the merge baseline (X's own `before` list joined with every `before` entry absent from `after`) equals D1's union for `merge`, since `Store.MergeTasks` removes exactly `remove_slugs` and rewrites no other entry's `depends_on` (`internal/boardengine/store.go:904-979`), and reduces to the stored list for the other verbs.
  X's status and labels as the write leaves them, the `done` dependency dropped, abandoned and done Y excluded, running Y kept, Y≠X, and the once-per-pair batch report with joined `Deps` all match D1-D3 and the Acceptance criteria; the "no other task waits on D" row does exercise the Y-is-X exclusion, since X itself waits on D and shares its labels.
  No row or rule adds scope beyond the answer key.
- Card 2 against D3-D5: every Acceptance criterion is carried (the four verbs' notice, two-line notice, bare `{"ok":true}` without sibling, both notices in one `notice` via `upsert-batch`, the skill rule and the restored line break before "A relation between notes" at `plugins/ly/skills/board/SKILL.md:130`).
  The claim that `joinNotices` gets two non-empty parts only via `upsert-batch` holds: `prepareRunSettings` returns a notice only for an entry held past Seed-Child (`internal/boardcli/runsettings.go:67-69`), and `scopeChanged` (`internal/boardengine/runlock.go:108-121`) refuses any added `depends_on` on a held entry, so `upsert` and `merge`, whose one upserted entry carries both, never produce both.
- Re-sign arrows: present on `SetDeps`, `UpsertTasksBatch` and `MergeTasks`, whose results change; correctly absent on `UpsertTask` (signature kept), `storeVerbs` and the three tests.
  Every Board-level caller of the three re-signed methods is listed: `storeVerbs` (`internal/boardcli/cli.go:463,748,806`), `TestRunLock`, `TestPriorityRoundTrip` and `TestEntryDates`; grep finds no other.
- Card order and `verify:`: card 2 uses only card 1's `Sibling`, `UpsertTaskSiblings` and re-signed methods.
  The `## verify:` section runs untagged and `integration` tests over `internal/boardengine/...`, `internal/boardcli/...` (whose `TestCLI` is integration-tagged) and `cmd/lyx/...` (the test budget), compiles `tmux` and `llm`, in the template's order; `plugins/` holds no Go.

## Rejected

- Keeping `Board.UpsertTask` as a delegating form: after card 2 its callers are tests across several packages; re-signing it would force edits in each with no behavior gain, and no rule or decision requires it, so it is not raised.
- `stepSiblingWarning` asserting the notice by substrings: the tokens it checks are exactly what D3 requires a line to name, and the exact `Deps` and `Labels` are pinned at the engine; not raised.
