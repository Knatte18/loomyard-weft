```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change, I list what reads, asserts or documents it and whether the diff covers it.

**Per change**

- **`Task.Discussed` and the `discussed` upsert key.**
  - Covered: `UpsertPayloadKeys`, `upsertAllowedKeys`, the replay field table, `scopeChanged` doc, the `boardcli` `Use` string, `boardcli/doc.go`, and `fakeLim`/`wire_seams_test`.
  - The upsert `Long` help drops the description of `front` and `on_land` when it merges them into one line with `discussed`. This is only a wording loss.
- **Decision record file at the Bolt root.**
  - Covered: `boardengine/doc.go`, the replay `Claims` and `CommitRuns`, and `BoardDir` doc.
  - Covered: the board sync commits through `Bolt.Commit`, which stages the whole tree, and the render manifest holds only rendered files.
  - Not covered: the stale `design-<slug>.md` description in `boardengine/replay.go:59` is still correct, since it is the design-file comment.
- **`Rename` now returns an error with the entry already saved.**
  - `Board.Rename` discards the result when `err != nil`, so the saved rename's `mentions` list is lost. See F1.
  - The only production caller is `boardcli/cli.go:240`, and its help text documents the two errors.
  - `plugins/ly/skills/board/SKILL.md` is updated.
- **`approve_seam: discussion` and the new `Env.ApproveDiscussion` closure.**
  - Covered: the `bouncerEntry` doc and switch, the `entries_gate.go` header, the `envkit.FullEnv` fixture, the tapestry recipe comments, and `loomcli/wiring.go`.
  - Not checked: other `Env` builders that run Bouncer rows through `requireSeam`. The diff touches none, and `FullEnv` is updated.
- **`lym loom decision store` (new verb) and `decision add` copying the record.**
  - Covered: `loomcli/doc.go`, the group `Long`, and the `add` `Long`.
  - Not covered: `contracts/orchrole/orch-role.md` and the parent/arbiter stencils describe `decision add` and the post-Discussion repair steps. None mentions the `record_store_error` report or `lym loom decision store`, though a parent orch is an admitted caller and the way forward now points at it. See F2.
- **The `decision add` `--from` help text, cut to fit the Long cap.**
  - The cut dropped "`--by` and `--amends-plan` apply to every entry", the one-notice-naming-headings sentence and the "two entries may not share a title or a PATTERN entry" rule. The text moved to a new `# Decision add --from` section in `loomcli/doc.go`, so an operator reading `--help` no longer sees it. See F3.
- **`lym board get --record`, `--body` exclusion and the `discussed` envelope field.**
  - Covered: the `get` `Long`, `boardcli/doc.go`, `SKILL.md`, and `outputGetTask` doc.
- **`design_prefix` refusal.**
  - The message cites `lym config board --set design_prefix=...`, and `configcli.go:477` shows that form exists.
  - `boardengine/doc.go` is updated.
- **PATTERN entries.**
  - The record declares no PATTERN change, and the diff touches no `PATTERN.md` or `pattern/` file, so declared changes landed.
  - `PATTERN-named-constants`: the prefix and suffix are declared once in `replay.go`, and `decisionFileName` forms the name once. The replay, config and board messages that hard-code `decision-%s.md` in `boardcli/cli.go:91` are descriptive text, not a name.
  - `PATTERN-loom-contract`: the approval row is decided from the recipe's rows, not its name.
- **Commit messages.** No `Pattern-Departure:` trailers appear, and the cards' commits carry none. `_lym/landing/summary.md` is absent, so no claims were checked.

**Findings**

- **F1: LOW, design.** The saved rename's result is discarded when only the record step fails.
  - Location: `internal/boardengine/rename.go:31-33` against `internal/boardengine/board.go:173`.
  - `boardCriticalSection` returns `(result, recordErr)`, but `Board.Rename` returns `RenameResult{}` on any error. The rename is saved, yet the caller and `boardcli/cli.go:240` lose the `mentions` list the operator needs to fix by hand.
  - Fix: have `Board.Rename` return the result alongside a record error, and have the CLI print the error with the result.
- **F2: MEDIUM, scope.** The parent orch's rulebook and the stencils are not updated for the new store verb and the `record_store_error` report.
  - Evidence: `contracts/orchrole/orch-role.md:79-192` has the orch run `decision add`, and `decisionWayStore` and the failed-commit way forward now name `lym loom decision store` for it.
  - I found no mention of `decision store` or `record_store_error` anywhere under `contracts/` or `plugins/`.
  - I did not check whether `PATTERN-lym-instructions-in-code` or the refusal-spec require the orch role to name every verb it may be told to run.
  - Fix: add a short orch-role line saying a `record_store_error` is cleared with `lym loom decision store <slug>`.
- **F3: LOW, consistency.** The `--from` help lost its user-facing contract to meet the Long cap.
  - Location: `internal/loomcli/decision.go` add `Long`, around the `--from` paragraph.
  - Fix: restore the one-notice and uniqueness sentences in a shortened form, or free the cap elsewhere. The text now in `loomcli/doc.go` is not what an operator sees.

**Tool calls (3)**
1. `sed` of `sync.go` plus a `grep -rln "decision add"` over `contracts`, `plugins` and `docs`. Context lacked how the board sync stages files and which contract documents describe `decision add`.
2. `grep` for `Rename(` callers and `design-<slug>` documentation. Context lacked the callers of the changed `Rename` and the other design-file docs.
3. `grep` for `decision store`, `record_store_error` and `discussed` over `contracts`, `docs/overview.md` and `plugins`. Context lacked whether those documents mention the new verb and flag.
