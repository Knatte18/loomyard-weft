# A session cycle sent /compact with a leading space, so it reached the orch as a plain prompt

`compact-leading-space` · Note · bug, shuttle, orch

## Seen

On 2026-11-07, after the orch note for a context cycle was written, the hub orch's session received ` /compact Keep every run in flight ...` as an ordinary prompt, with a leading space, so Claude Code ran no compact and the cycle did not happen.

`CompactSessionSequence` in `internal/shuttleengine/claudeengine/session.go` builds the text with no leading space and relies on the caller having proved the input box empty.
`isBlankBoxInterior` in the same file counts whitespace as blank.
So a likely cause is a space left in the input box, which reads as empty and sits in front of the pasted text.
That is not proved: where the space came from is unknown, and the prompt was seen only in the transcript.

## Why it matters

`/clear` and `/reload-plugins` go the same way, and a slash command with a leading space fails with no error, reaching the agent as a prompt it cannot act on.

## Wanted

Find where the space came from, and have the sequence clear the box first or check that the command ran, so a cycle does not silently fail.
