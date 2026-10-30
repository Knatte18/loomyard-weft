# Stop-hook creel mail arrives twice and is labelled as an error

`stop-hook-mail-doubled` · Note · bug, creel

## Fixed (2026-10-30)

The `creel-mail-delivered-once` landing fixed this: a Stop delivery now prints `additionalContext` alone with no error label, the idle ring hands the mail over once, and the model and the operator each see a mail once.
The note stays until the operator confirms with the tier3 `TestSmokeCreelHookDelivery` smoke on a real session.

## Seen (operator and hub orch, 2026-10-29)

Each creel mail reaches the operator's terminal twice at a Stop hook:

- a `Stop hook feedback` block headed `Stop says: [creel] #N from ...` with the mail text;
- then `Ran 2 stop hooks` with a `Stop hook error: [creel] Mail for you arrived... additional context` line and a second `Stop hook feedback: [creel] #N ...` with the mail text again.

A normal delivery is labelled `Stop hook error`, and the hook command text reaches the model.
The installed `lym` carries the fix of `creel-hook-delivery-fixes`, so the bug stands in the current build.

## Expected

One message per mail, delivered once, labelled as a delivery and not as an error, with no hook command text in the model's context.
