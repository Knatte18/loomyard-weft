# File a GitHub self-report for the doubled Stop-hook mail delivery

`stop-hook-selfreport` · Note · bug, creel, selfreport

## Seen (operator and hub orch, 2026-10-29)

The Stop hook delivers each creel mail twice to the operator's terminal: a `Stop hook feedback` block with the mail text, then `Ran 2 stop hooks` with a `Stop hook error` line naming the hook command and a second `Stop hook feedback` with the same mail text.
The hook command text reaches the model, and a normal delivery is labelled `Stop hook error`.
The task `creel-mail-delivered-once` fixes it and carries issue #900.

## Idea

File a GitHub self-report with `lym selfreport create` for the doubled delivery, as the operator asked on 2026-10-29: what was seen, the installed `lym` that carries the fix of `creel-hook-delivery-fixes`, and the expected behaviour of one message, delivered once.
Check first whether #900 already covers it, and if so fold this note into it.
