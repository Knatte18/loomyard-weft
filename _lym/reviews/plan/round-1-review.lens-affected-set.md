```yaml
lens: affected-set
outcome: reported
```

Task: the affected-set lens. For each output, key, string or behaviour a card changes, I listed what reads or asserts it and which cards cover it.

**Per change: the set found, and the cards covering it**

1. **`BoundaryBlock` replaced by `HookOutput`, and the Stop/PostToolUse JSON changes (card 1).**
   - Readers: `hookread.go:116`, `cli_test.go:159-166` (`hookBlock`), `settings_test.go:206-215`, `shuttleengines_test.go:344-347`, `hookcmd_integration_test.go:340` (`block` constant).
   - Docs: `engine.go:237-243`, `capabilities.go:1-2,66-92`, `claudeengine/doc.go:133-139`, `settings.go:3,141-145`, `shuttleengine/doc.go`, `shuttleengines/doc.go`, `creelcli/doc.go`, `creelengine/doc.go:127`, `docs/overview.md:479`.
   - Covered by cards 1 and 8. The `hookcmd` constant is updated by card 8.
   - The `capabilities.go` header is named in card 1's Intent but is not a target; the other lenses own that.
2. **`hook-wait` output changes from `BellLine` to the claimed lines (card 6).**
   - Readers: `cli_test.go:942` (covered by card 6), `cli_test.go:1011` (**not covered**), `creelcli/doc.go:39` (card 6), `hookwait.go:19-32,45` (card 6), `docs/overview.md` (card 6).
   - `smoke_llm_test.go:348,366` assert that no prompt holds the bell line. They stay true, and card 10 covers that test.
   - `contracts/stencils/orch/orch-template-role.md:40` (**not covered**).
3. **The ring carries mail, and the `takes_mail` and `ack_required` fields (card 4).**
   - Readers: `server_test.go:358-376` (card 4), `server_test.go:1075-1100`, which is an unmarked Go-client hook-wait across a restart and is still valid, `pool.go:258` `OutcomeRing` comment (card 4), `creelengine/doc.go` idle-wake section (card 4).
   - Covered.
4. **`Reading.Ack` releases on a journal failure (card 3).** Readers: `server.go:470-472` (`handOut` Warn, cards 4 and 9) and `creelengine/doc.go` (card 3). No test asserts the Warn text. Covered.
5. **`hook-wait` stops resuming, and `ErrAckUnsent` (card 5).** Readers: `client.go:128,172-177` and `creelengine/doc.go` (card 5). Covered.
6. **Echo and the flag set (card 7).** Readers: `flags.go:16`, `TestStore_MailFlags` (card 7), `settings.go` merged and PostToolUse commands (card 8 for the Stop doc). Covered.
7. **Wake labels (card 2).** Readers: `settings.go` `hookCommand` and `TestBuildSettings_CreelMailHook`. Covered.
8. **Delivery log lines (card 9).** No existing reader. The `handOut` Warn is covered by card 9.

**Findings**

1. **MEDIUM, consistency.**
   - Location: card 6 (`06-hook-wait-hands-mail.md`), its `TestCreelCLI` coverage.
   - Card 6 updates only the subtest `hook-wait rings a strand as exit 2 with the bell line`.
   - It misses the subtest `a hook-wait superseded by a newer one of the strand exits 0 printing nothing`. That subtest asserts the newer waiter's stderr equals `creelengine.BellLine+"\n"`.
   - Under card 6, that waiter's stderr carries the claimed lines of #1 "mail" and #2 "more mail", so the assertion fails.
   - Evidence: `internal/creelcli/cli_test.go:1011-1012`.
2. **MEDIUM, scope.**
   - Location: no card. The file is `contracts/stencils/orch/orch-template-role.md`.
   - Line 40 tells the orch: "An idle orch is woken first by the fixed line `[creel] Mail is waiting for you.`; end that turn, after which the mail shows."
   - D3 and D4 make a ring with a waiter wake the orch with the mail itself, under `[creel] Mail that arrived while you were idle:`, and no empty turn runs.
   - The stencil now gives a false instruction to end the woken turn, and no card lists it.
   - The bell still exists for a strand no waiter reaches, so the sentence needs rewording, not deletion.
   - Evidence: `contracts/stencils/orch/orch-template-role.md:39-40`; decision record D3 and the Scope bullet on renderings.

**Tool calls**
- Grepped the repo for block/bell wording in md, go and yaml files: the manifest held no search beyond the creel packages.
- Grepped the non-creel hits and the stencils for creel wake text: to rule out false hits and find stencil readers of the wake text.
- Read the orch stencil lines 36-42 and grepped for the bell and Stop-block texts in contracts and internal: to confirm the stencil sentence that D3 falsifies.
- Grepped tests for the `handOut` Warn text and for the `ackReadTimeout` and `stopBlocks` readers: to check whether any test asserts the log line card 9 narrows.
