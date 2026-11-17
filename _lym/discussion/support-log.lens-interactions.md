```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that share state, plus each acceptance criterion against the decisions it touches.

**Pairs checked**
- D1 × D7 (the README sections the projection names, and the details wrapping): held. D7 wraps sections but does not rename them, so D1's section values still match the README.
- D1 × D3 (`board-data.js` through `RenderToDisk`, the manifest, the sync and the replay): held.
- D2 × D3 (fixed `board.html` written on every render, and replay claims): broken under mixed binaries. See F1.
- D2 × D5 (asset written only on the next board write; open refuses when it is absent): held. `lym board rerender` runs `Board.boardCriticalSection`, which renders and syncs, so the named way forward does create and commit the file.
- D2 × D4 (the page's missing-data line, and both files rendered together): held. Both files come from one render and one commit, so a page without data arises only from a hand edit. The line covers that case.
- D5 × D6 (the `board open` task runs the verb from the worktree's folder): held, assuming board verbs resolve from a pair anchor as D5 states.
- D6 tasks × D6 keys (`alt+shift+0` in a task pair, which has no `beam start` task): held. The record bounds it.
- D6 × the existing `reed attach` and `ctrl+alt+a` (one block writer, rewritten whole): held.
- D3 × the owner-set write path (`renderOwnerWrite` commits the names `Render` returns): held.

**Criteria checked**
- D2 "a second render of a changed board leaves its bytes unchanged": consistent with D2.
- D3 "the sync commit holds both … when they changed": consistent. `board.html` rarely changes.
- D6 "seeding a beam writes the same three": consistent with the `isBeam` degradation bound.
- D5 refusal criterion against D2: consistent.

**Findings**

- F1, MEDIUM, D2 × D3: two machines whose lym binaries ship different viewers flip `board.html` back and forth.
  - Evidence: D2 writes the binary's own embedded asset on every render, and D3 makes a replay re-render with the local binary. A hub shared by two owner machines on different lym builds therefore rewrites and commits `board.html` on each machine's board write. The record says the file "changes … only when a new lym binary ships a changed viewer", but that holds only for a single binary version.
  - Fix: state the bound, that mixed-version hubs churn `board.html` until both machines are upgraded, or have Render keep an existing `board.html` whose embedded version stamp is newer.
- F2, LOW, D5 × the `rerender` verb: D5's refusal names `lym board rerender`, but that verb is annotated `clihelp.AudienceInternal`, and `lym board open` is an operator verb.
  - Evidence: `internal/boardcli/cli.go`, `rerenderCmd` carries the internal audience annotation.
  - Fix: confirm an internal verb is acceptable as an operator's way forward, or say why it is.

Tool calls:
- `grep` of `Board.Rerender` in `internal/boardengine/board.go`: my context did not show whether rerender commits, which D5's way forward depends on.
