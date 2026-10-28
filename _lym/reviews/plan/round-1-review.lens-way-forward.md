```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. For each refusal or recovery step a card adds, I checked whether its next step can succeed from the state the refusal leaves.

## Per refusal: next step, progresses or not

1. **Card 1: a local `code_reviewer_fan: nosuchfan` is refused by `ValidateFanKeys`.** Next step: "define the fan in lenses.yaml, or set the entry to """. Card 1's Intent adds that the value sits in the pair's local loom file. Setting that local entry to `""` reloads cleanly, since an empty entry never reaches `ResolveFan`. **Progresses.**
2. **Card 1: `TestPrepareRunSettings`' three new refused rows (`plan_fan`, `discussion_fix_fan`, `plan_fix_fan`).** These are existing guard-key refusals reached after loom's Migrate; the card adds only test rows. Next step: rename or remove the key in profiles.yaml or in the entry's run_settings. That works from run_settings or a profile block. **Progresses.**
3. **Card 1: the other five fan keys stay refused in the local layer.** This refusal already exists and the card does not change it; its next step is unchanged. **Progresses.**
4. **Card 1: a hand-edited local file holding `webster_fan` or `webster_reviewer_fan` is refused as a key outside the template.** The card keeps this as it is ("until a reconcile migrates it"). `lym config reconcile --apply` migrates the key, which is now admitted. **Progresses.**
5. **Cards 3 and 5: a hub fan names a deleted lens (`goal-scope`, `board-fit`, `code-pattern-fit`) and fails `ValidateFanKeys` at every loom config load.** The refusal's own next step, "set the entry to """, progresses: it runs the round solo. The cards name a different recovery: drop the name from the fan with `lym config lenses --set` or the editor. The editor progresses. `--set` progresses only when one call fixes every broken fan (finding 1). **Progresses, with one gap.**
6. **Card 3: a fan named only by `discussion_lenses` or `plan_lenses` that names a deleted lens fails when the Discussion-Write or Plan-Write lead resolves it.** Next step: set the key to `""` or fix lenses.yaml (`discussiontable_test.go:521-524`). **Progresses.**
7. **Cards 2 and 4.** Neither adds a refusal, stop, reset or recovery step. The emphasis-line and writer-fan failures are test assertions, not runtime refusals.

## Findings

**1. LOW · consistency · cards 3 and 5 (Intent, recovery paragraph)**

Summary: fixing one fan at a time with `lym config lenses --set` is rolled back while another fan still names a deleted lens. The cards' recovery step does not say that every broken fan must be fixed in one call.

- `setChecked` and `newLoadFailure` (`internal/configcli/configcli.go:273-296`) undo a write whenever the module's `CheckLoad` fails with an error text different from the one before the write.
- `lenses.CheckLoad` reports every broken fan in one joined error (`internal/lenses/config_test.go:569-584`, "two broken fans return both failures in name order").
- Example: a hub whose `lenses.yaml` tunes both `twill-discussion` and `review-discussion` with `goal-scope`. Fixing one fan by `--set` changes the error text, so the write is put back. The operator stays at the same failure until one `--set` carries every fan (`--set` is repeatable) or they use the editor.
- Proposed fix: cards 3 and 5 say the `--set` recovery fixes every fan naming the deleted lens in one call, with one `--set` per fan, or uses the editor.

Tool calls:
1. `lym config lenses --help` and a grep for `CheckLoad` in configcli, hubreconcile and configreg: to learn whether `--set` checks the load after writing. My context lacked configcli's write path.
2. `internal/configcli/configcli.go:270-300` and `335-350`: to read how `--set` compares the before and after load errors, which decides whether a stepwise fix progresses.
3. A grep of `internal/hubreconcile` for `Ensure` and loader calls: to check whether a broken loom load would block every `lym` command before the recovery runs. `EnsureOrWarn` exists, so it does not block.
