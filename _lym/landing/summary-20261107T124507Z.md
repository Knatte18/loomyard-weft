# Clear the box before slash commands; give open sessions the role's TTL

Two fixes in shuttle's Claude path.

## A slash command sent behind a space

On 2026-11-07 at 08:53:53Z the hub orch's session got ` /compact Keep every run ...` as a prompt, and no compaction ran.
The records show what happened:

- The orch's events file holds a `UserPromptSubmit` for that text, and its prompt opens with an ASCII space.
  The orch's 53 successful `/compact` sends fire no `UserPromptSubmit`; nor do `/reload-plugins` or `/clear`.
- Creel journaled inject 860 as `injected` 0.4 s after it started sending.
  `confirmPasted` counted any turn start past the paste as a landing.
  So the prompt's turn start confirmed the command, and the cycle only went on when the session compacted 4.5 minutes later.
- lym did not write the space.
  `CompactSessionSequence` has no space, reed loads the paste buffer verbatim from standard input,
  and the paste went in as one bracketed paste.
  The space was already in the input box.
  The idle reading counts whitespace as blank, so it read the box as empty.
  Nothing records what typed the space, so its source is still unknown.

The fix stops whitespace in the box from turning a command into a prompt, whatever typed it,
and stops a command that ran as a prompt from passing as a success:

- `pasteVerified` plays the engine's `InputBoxClearer` keys right before the paste of a send that starts no turn.
  For Claude that is one `C-u`.
  This covers `/clear`, `/compact`, `/reload-plugins` and `/color`.
  The clear goes only into a box the guard has just read as empty, after the operator-activity hold,
  and C-u interrupts no turn.
  `InputBoxClearer`'s contract now lists this third case.
- `confirmPasted` fails such a send with the new `ErrCommandTakenAsPrompt` when a turn starts after the paste.
  `mapDeliveryError` passes the error through as a failed send.
  Creel then requeues the inject and types it again once the session is idle.
  It no longer journals the inject as `injected`, or as `unconfirmed`, which the orch watcher would have waited out to its handoff timeout.

## `lym shuttle open` sessions cached for 1h

`OpenLaunch` set no `CLAUDE_CODE_PROMPT_CACHE_TTL`, so opened sessions wrote their cache at Claude Code's 1h default.
It now resolves the TTL from `OpenSpec.Role` through `claude.yaml` with the same resolver `Prepare` uses.
It logs the value, and it refuses an invalid TTL set before it writes anything.
Both launch lines open with one `ExportEnv` statement.
A per-command `WithEnv` prefix would not work here:
on POSIX it would cover only the resume half of `resume || fresh`,
and on pwsh it would split the `||` chain.
`Runner.Open` has one caller, `lym shuttle open`, so no other window launches through this path.

## Tests

- `checkPastedSessionCommand`, run for clear, compact, reload-plugins and color, pins the `C-u` sent right before the paste.
  It also pins that a held command sends no key.
  A new row pins that a turn start the paste causes fails the command with `ErrCommandTakenAsPrompt`.
- `checkLandingStartConfirmation` now expects that one clear key before the paste, and no key after it.
- `TestOpenLaunch` checks every line after the TTL export.
  New rows pin a mapped role's TTL, an unmapped role's default and the refusal of an invalid TTL.

```yaml
pattern_departures: none
```
