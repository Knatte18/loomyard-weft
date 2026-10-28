```yaml
lens: code-tests
outcome: reported
```

Per-behaviour list (test that pins it, or none):

1. **A landing send that read its box empty, then sees other text, returns nil with no key sent.** Pinned by the row "a box that read empty and then holds the operator's draft…" in `checkLandingStartConfirmation` (`session_test.go:~601`). It also asserts `sentCalls` is empty, and it runs for both `ClearSession` (`:700`) and `CompactSession` (`:787`) on the fake clock. Pinned.
2. **A landing send whose box holds foreign text before any empty read still fails `ErrForeignBoxText`.** Pinned by the row "…before any empty read fails as foreign text" (`wantErr`).
3. **A non-landing send that reads its box empty returns nil at once** (the `box == "" && !awaitsLanding` branch). Pinned by the existing parserless row, "an engine that parses no session signals confirms on the empty box at once", and by the other paste-path tests. Pinned.
4. **An empty read followed by the compaction session start succeeds.** No row has the empty read before the start line. The "matching source" row appends the line at the paste, so the session-start check returns at the first read, before the box read. Because that check comes before the box branch, `submitted` cannot change the outcome. A regression here is unlikely, and the criterion is met by the existing row. Not pinned separately; LOW at most.
5. **Once `submitted`, an unreadable box sends no key and the window's end returns nil.** No row scripts `{"", true}` followed by an unreadable answer (`{"", false}`). D2 and the `send.go` doc comment both claim this behaviour. None.
6. **Once `submitted`, a box that holds the sent text itself, its placeholder, or the sent text beside other text sends no key and fails nothing.** The draft row uses only foreign text, which takes the `default` case. If the `!submitted` guard were moved to guard only the `default` case, the Enter and `ErrSentTextInDraft` paths would come back after submission and every test would still pass. An operator typing the same `/compact` again is a realistic input for that regression. None.
7. **Exit by the read cap (`read >= maxReads`) with `submitted` set returns nil.** The draft row ends on the window closing; no row forces the cap to bind first. D2 and the doc comment name the cap. None.
8. **D3: a strand without `CreelHooks` gets its notice and reminder.** Pinned by the existing `TestScheduler_Reminders` rows and the reminder rows that use `creelfake.LiveStrand`, which is now a plain pane-mail strand. Pinned.
9. **D3: a `CreelHooks` strand gets no notice or reminder, only its injects typed.** Pinned by the rewritten row "AHookStrandGetsNoNoticeOrReminderButItsInjectsAreTyped" (`deliver_test.go:~246`). Pinned.
10. **D3: only a `CreelHooks` strand is rung or belled, and a plain live strand never gets the bell.** Pinned by the edited "OnlyAStrandMarkedCreelHooksOnAnUnpausedPaneGetsTheBell" and by "APaneMailStrandIsNeverRung" (`deliver_test.go:1836`). Pinned.
11. **D3: hub delivery lists a Claude strand without `CreelHooks` for the pane path, and one with it for the wake.** Pinned by the "Strands carries each strand's CreelHooks mark…" subtest (`delivery_test.go:281`). It seeds `ly:hooked` and `ly:unmarked` as claude, and `ly:codex` and `ly:hand`, and asserts each `CreelHooks` value.
12. **Removed coverage:** the dropped "unflagged strand gets nothing" case tested the third class of strand that D3 deletes, so removing it loses nothing. The `PaneMail: true` removals in the `Strands()` expectations are consistent with the field being gone.

Findings:

- **LOW, scope.** `internal/shuttleengine/session_test.go:~598`, the `checkLandingStartConfirmation` table. No row covers behaviours 5, 6 and 7 above (unreadable box after an empty read, box holding the sent text or its placeholder after an empty read, and the read cap binding first). Each is a branch of the new `submitted` logic that D2 and the `send.go` doc comment state. Add rows to the same table, as extra `afterPaste` scripts: `{"",true},{"/compact",true}`, and `{"",true},{"",false}`. Evidence: `send.go:627-650`; the table at `session_test.go:589-600`.

One note outside my lens: `internal/creelcli/smoke_tmux_test.go:76` now has two comment lines after the first one was replaced, so the doc comment no longer starts with `TestSmokeCreelFakeProviderPane`. The first line should read "TestSmokeCreelFakeProviderPane runs the real daemon on the real socket over a hub whose one strand is the fake provider." The line "The strand carries the provider tag `script` and no `CreelHooks` mark…" duplicates the line after it.

Tool calls (1):
1. `sed`/`grep` on `delivery_test.go:281-296` and the `checkLandingStartConfirmation` call sites. My context lacked the seeded providers of the subtest and the call sites of the shared helper. The `ReadInputBox` grep returned nothing; I did not read how `inputBoxEngine` repeats its last answer.
