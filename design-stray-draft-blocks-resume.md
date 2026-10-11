# A stray character in a driver's input box blocks loom resume

`stray-draft-blocks-resume` · Note · bug

Imported from [issue #554](https://github.com/Knatte18/loomyard/issues/554).

## Seen (2026-10-11, small-fixes-oct11)

After a Discussion-review escalation was accepted with `lyx loom circling accept`, `lyx loom resume` in the pair refused to wake the parked driver, and kept refusing on retries.
The driver's pane showed a fully idle Claude Code UI whose input box held one stray character, `❯ +`, which nobody had typed on purpose.
Resume read it as an operator draft and refused, so the run stayed `awaiting` at Discussion-Bouncer until the operator attached and deleted the character by hand.

Same class as #549, where a stray `%` after the prompt failed a provider start.

## Workaround

The operator attaches with `lyx reed attach` in the pair, deletes the stray character, and the orch runs `lyx loom resume`.

## Expected

A lone stray non-alphanumeric character in an otherwise idle input box does not block a resume or a start for good.
Either treat a single stray marker as no draft, or clear it after a bounded wait with no operator activity, and in any case say in the refusal which text was found so the orch can act without opening the pane.
Moving readiness detection to hooks (board task `shuttle-hook-first`) covers the idle half; the operator-draft check stays on the pane and needs this rule.