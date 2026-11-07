# Claude sessions: a slash command sent with a leading space, and open sessions caching 1h

`claude-session-fixes` · Task · bug, shuttle, claude · done

Two bugs in `internal/shuttleengine/claudeengine`, bundled by the operator (2026-11-07).

## 1. A slash command sent with a leading space

### Seen

On 2026-11-07, after the orch note for a context cycle was written, the hub orch's session received ` /compact Keep every run in flight ...` as an ordinary prompt, with a leading space, so Claude Code ran no compact and the cycle did not happen.

`CompactSessionSequence` in `session.go` builds the text with no leading space and relies on the caller having proved the input box empty.
`isBlankBoxInterior` in the same file counts whitespace as blank.
So a likely cause is a space left in the input box, which reads as empty and sits in front of the pasted text.
That is not proved: where the space came from is unknown, and the prompt was seen only in the transcript.

`/clear` and `/reload-plugins` go the same way, and a slash command with a leading space fails with no error, reaching the agent as a prompt it cannot act on.

### Wanted

Find where the space came from, and have the sequence clear the box first or check that the command ran, so a cycle does not silently fail.

## 2. `lym shuttle open` sessions cache 1h

### Seen

`buildOpenCmd` in `command.go` takes no TTL and sets no `CLAUDE_CODE_PROMPT_CACHE_TTL`, while `buildLaunchCmd` and `buildResumeCmd` do.
Sessions opened with `lym shuttle open` write their prompt cache with a 1h TTL: their transcripts show `ephemeral_1h_input_tokens`.
`claude.yaml` resolves `prompt_cache_ttl` to 5m for their role, and sessions started by `lym shuttle run`, and their subagents, write 5m.
Which other lym windows launch through `OpenLaunch` besides `lym shuttle open` is not checked.

### Wanted

A session from `lym shuttle open`, and any other window launched through the open path, writes its cache at the TTL `claude.yaml` resolves for its role, as a `lym shuttle run` session does.
The operator wants 5m for these sessions.
