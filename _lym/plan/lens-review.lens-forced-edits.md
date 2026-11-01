```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens report on the plan in `_lym/plan`. Every finding is a stale comment or test assertion that a card's change makes wrong and no card names.

1. **LOW — Card 9 (button-click-binding), `TestPinGeometryOptionsLocked`.** The `ShellPinsFollowTheOptionPins` subtest will break, and the card does not name it.
   - Card 9 puts `set-option -gu @lym_button_command` right after the follow-command unset.
   - That subtest's `wantLast` requires the last two `set-option` calls to be the follow-command unset and then the build-stamp unset (`internal/reedengine/windowsize_test.go`, `ShellPinsFollowTheOptionPins`).
   - With the new unset between them, the subtest fails. Card 9's Intent names only `wantOptions` and the counts, so it should also name this `wantLast` change.
2. **LOW — Card 5 (third-status-line), stated reason.** The `//testtiming:keep` reason above `internal/reedengine#TestPinGeometryOptionsLocked` still says "the two global status formats, status 2".
   - Card 5 changes these to three formats and `status 3`, and cards 6 and 9 add more pins.
   - No card rewrites the reason.
3. **LOW — Card 5, line 2 test message.** `internal/reedengine#TestBar_LineTwoShowsTheCheckMarkBeforeTheClock` has the error text "want the mark right-aligned after the follow button" (`bar_integration_test.go`, the running-mark assertion).
   - Card 5 moves the follow button off line 2, so the message describes a layout that no longer exists.
   - Card 5 lists only the sibling test, `TestBar_LineTwoListsSessionsPrimeFirst`.
4. **LOW — Card 1 (shell-detached-forms), package header.** The header of `internal/shell/shell.go` (line 1) lists every mechanic the `Shell` interface covers.
   - Card 1 adds output discard, zero exit and the command flag, but edits only `internal/shell#Shell` and the tests, not the header.
5. **LOW — Card 2 (buttons-config-keys), template accessor doc.** The doc comment of `internal/reedengine#ConfigTemplate` (`template.go`) lists the `${env:…}` keys and names the plain-literal ones ("width, height and the segment_colors block").
   - The new `buttons` block is also a plain literal, so that list goes stale.
   - Card 2 does not list `internal/reedengine#ConfigTemplate`.
6. **LOW — Card 4 (geometry-button-context), file header.** The header of `internal/reedengine/geometry.go` (lines 1–4) says the file declares `Geometry`, the type only.
   - Card 4 puts `ButtonContext` and `ButtonBeam` in that file and does not update the header.
7. **LOW — Card 8 (reed-button-verb), stencil-seed gate comments.** Card 8 adds a "reed button" row to `cmd/lym#TestSkipStencilSeed_HonoursTheAnnotation`.
   - The file header (`cmd/lym/stencilseedgate_test.go:5`) and the test's doc comment (`:36`) name the commands that must carry the annotation: "lym reed switch", "lym reed follow" and every creel command.
   - Neither comment is updated to include `lym reed button`.

Tool calls I made:
- grep for other `Shell` implementers, callers of `bindingArgvs` and `switchBindingCommandsLocked`, `set-option` count assertions, `Config` literals and the reedengine enforcement scans. My context lacked these search results.
- grep for set-option sequence and length assertions, enforcement allowlists, and the cmd/lym registry-style test files. My context had not covered the tests outside the reed packages.
- Read the headers and tables of `clitree_test.go`, `spawnobservability_test.go`, `sandbox_coverage_test.go`, `exitcode_test.go`, `registration_test.go`, `jsonhelp_test.go`, `cwdmutation_test.go` and `checkedcall_test.go`. I needed to know whether any pins a verb or spawn list.
- Read the `TestCLITree_EveryCommand` and `TestCLIForm_EveryCommand` bodies, to rule out a per-command table.
- Read the `TestSpawnObservability_ProductionSpawnsAreLogged` rule and grepped for import allowlists naming hubgeom or desktophook. I needed the spawn rule and to confirm card 4's new import is not pinned anywhere.
