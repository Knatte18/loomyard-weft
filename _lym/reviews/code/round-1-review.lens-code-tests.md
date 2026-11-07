```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour**

| # | Behaviour | Pinning test |
|---|---|---|
| 1 | One `C-u` is sent right before the paste of a send that starts no turn (clear, compact, reload-plugins, color) | `checkPastedSessionCommand`, "an empty box pastes the command once", through `checkClearBeforePaste`. It asserts the only key is `SendKey:C-u` and that the call log shows it directly before `paste:`. |
| 2 | The clear key plays in the landing-confirmation path too, and no key is sent after the paste | `checkLandingStartConfirmation`, every row, through `checkClearBeforePaste`. |
| 3 | A held command (draft in the box, or operator active) sends no key and no paste | `checkPastedSessionCommand`, "a drafted box holds the command" and "an active operator holds the command". Both assert no `PasteSubmit` and no sent keys. |
| 4 | A turn start after the paste fails the send with `ErrCommandTakenAsPrompt` | `checkPastedSessionCommand`, "a turn the paste started fails the command as taken for a prompt". It asserts `errors.Is`. |
| 5 | A failed clear key fails the send before any paste ("nothing was pasted") | None. No row injects a `SendKey` error, so this branch of `clearBoxBeforeCommand` and its `pasteVerified` return are never reached. |
| 6 | An engine with no `InputBoxClearer` skips the clear and still pastes | None. The diff adds `InputBoxClearer` to `parserlessPasteEngine`, so no paste-path engine without the capability is left in the tests. |
| 7 | `mapDeliveryError` passes `ErrCommandTakenAsPrompt` through, so creel requeues instead of journalling `injected` or `unconfirmed` | None. The diff changes only a comment there. The delivery tests' error-mapping tables (`delivery_test.go:407-409` and `:461`) have no row for it, and no creel test sets this error. |
| 8 | `ErrCommandTakenAsPrompt` leaves no pending-send record, so a retry replays the clear | None. Row 4 doesn't assert `loadPendingSend` is empty, and no test retries after the error. |
| 9 | `OpenLaunch` exports the role's mapped TTL ahead of both lines | `TestOpenLaunch`, "the role's mapped TTL is exported ahead of both lines". `openLines` fails unless both `Cmd` and `ResumeCmd` open with `Chain(ExportEnv(key, ttl), …)` at 5m. |
| 10 | An unmapped role gets the default TTL | `TestOpenLaunch`, "an unmapped role exports the default TTL". |
| 11 | An invalid TTL refuses the open before anything is written | `TestOpenLaunch`, "an invalid TTL is refused", for an invalid default only. |
| 12 | The resume line is `OrElse(resume, fresh)` under the one export | `TestOpenLaunch`, the mapped-TTL row and "resume falls back to a fresh start". |
| 13 | Every existing line check in `TestOpenLaunch` now runs on the lines after the export | The `check` callbacks in `TestOpenLaunch` take the stripped `fresh` and `resume`. |
| 14 | The TTL export reaches the claude process in both shell dialects | Partly. `TestOpenLaunch` builds `sh` once, so it covers one dialect. Nothing runs the line, so it pins the text only. |

**Findings**

1. **LOW, design.** The failed-clear-key path has no test (row 5). Location: `internal/shuttleengine/send.go:645-653`, `pasteVerified` at `send.go:623-627`. A fake reed whose `SendKey` fails is the missing row; `checkPastedSessionCommand` is the table to add it to. Without it, a regression that pastes after a failed clear passes.
2. **LOW, design.** No paste-path engine without `InputBoxClearer` is exercised (row 6). Location: `internal/shuttleengine/send.go:646-649`; the capability was added to `parserlessPasteEngine` at `internal/shuttleengine/session_test.go:581-587` and `:615`. Dropping the `ok` check would pass every test.
3. **LOW, design.** The creel requeue of `ErrCommandTakenAsPrompt` has no test (row 7). Location: `internal/creelcli/delivery.go:460-461` and `:462-477`. The change description and doc comment claim the requeue, but only the default branch of `mapDeliveryError` carries it. A new `mapDeliveryError` case mapping the error to `ErrNotLanded` or `ErrBusy` would change the outcome with no failing test. One row in the `delivery_test.go` table at `:461`, expecting the error passed through unchanged, would pin it. Add a row; do not add a new test function.
4. **NIT, design.** The test for row 4 doesn't check pending-send state (row 8). Location: `session_test.go:717` and `:748-753`. The claim that a retry replays the clear is unpinned. It follows from the code and no `savePendingSend` call, so this is NIT, not a coverage gap.
5. **NIT, design.** The invalid-TTL row covers only the default, not a bad `prompt_cache_ttl_roles` entry. Location: `claudeengine/command_test.go`, the "an invalid TTL is refused" row. The resolver is shared with `Prepare` and tested there, so I'm not raising it as a gap.

**Test economy**
- None of the new rows duplicates another behaviour's coverage.
- `checkClearBeforePaste` is a shared helper, not a per-helper test.
- No test asserts stencil or doc prose.

**Tool calls**
- None. The inherited diff and reads covered every behaviour, so I read no more files.
