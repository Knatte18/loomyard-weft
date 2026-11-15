```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. List per change:

- **`UpsertTaskSiblings` removed, replaced by `UpsertTaskReporting`.** I found these references. `internal/boardengine/doc.go:700` still names `UpsertTaskSiblings`. No other Go code, spec or stencil does. Not covered: finding F1.
- **The `lym board upsert` envelope gains a `warning` key.** I found these places that describe upsert output.
  - `internal/boardcli/doc.go` is updated in the diff.
  - `plugins/ly/skills/board/SKILL.md:134-135` describes the `warning` key for `promote` only. It does not mention the new upsert warning. Not covered: finding F2.
  - `docs/overview.md:1188` and `contracts/specs/loom-discussion-spec.md:137` mention `--record-file` without describing warnings, so nothing there is wrong.
- **The envelope is hand-built instead of using `outputSuccessWithTask`.** The shape is the same: `task`, plus `notice` only when it is non-empty. Covered. `outputSuccessWithTask` still has other callers, since the grep shows no removal.
- **`UpsertTask` on `Board` now goes through `UpsertTaskReporting`.** Behavior is unchanged. Covered.
- **`UpsertTaskWithRecord` is left without a warning.** That is intentional, because it replaces the record itself. Covered.
- **PATTERN entries.** The diff declares no PATTERN changes. I found no entry the diff violates: no new git, config or stencil touchpoints. Covered.
- **Commit messages.** There are no `Pattern-Departure:` trailers and none are needed.

## Findings

**F1: LOW, consistency**
- Location: `internal/boardengine/doc.go:700`
- Summary: the doc still lists `UpsertTaskSiblings` among the siblings-returning methods, although the diff removed that method and replaced it with `UpsertTaskReporting`.
- Evidence: `doc.go:700` reads "SetDeps, UpsertTaskSiblings, UpsertTasksBatch and MergeTasks return the siblings." The grep finds no other `UpsertTaskSiblings` reference.
- Fix: replace `UpsertTaskSiblings` with `UpsertTaskReporting` on that line.

**F2: LOW, consistency**
- Location: `plugins/ly/skills/board/SKILL.md:134`
- Summary: the board skill documents the `warning` key only for `promote`, so an agent reading it won't learn that a plain `upsert` of a body edit on a Sketch with a stored record now warns.
- Evidence: `SKILL.md:134-135` covers the promote warning only. `SKILL.md:113-116` covers `--record-file` without any warning.
- Fix: add a sentence near line 113-135 saying that `upsert` without `--record-file` adds a `warning` key when it edits the body of an open entry whose record is stored. The warning names `--record-file` and `lym loom decision add`, and the write still lands.

Tool calls:
1. One grep for `UpsertTaskSiblings` and for docs that mention `--record-file` or record warnings. My context did not hold the stale references or the docs outside `internal/`.
