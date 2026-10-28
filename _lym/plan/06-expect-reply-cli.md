# Card 6 — expect-reply-cli

**Edit:**
- `internal/creelcli#postCmd`
- `internal/creelcli#writeMessagesText`
- `internal/creelcli#messageFields`
- `internal/creelcli#logEntryFields`
- `internal/creelcli#TestWriteMessagesText`
- `internal/creelcli#TestCreelCLI`

**Extends:**
- `internal/creelcli/doc.go`

**Uses:**
- `internal/creelengine#Request`

**Intent:**
The CLI side of the expected-reply mark (D9, D11).
`lym creel post` gains `--expect-reply`, which sets the request's `ExpectReply` on a post to a receiver or to `--parent`, without blocking; `--wait-reply` sets it too, through the engine's rule that a wait implies the mark.
`--expect-reply` with `--reply` is refused before any call, naming `post --reply <id> <text>` without the flag as the way forward, since a reply is no question.
`read`'s header for a marked message reads `#<id> from "<sender>"…, expects a reply — reply with: lym creel post --reply <id> <text>`, the mark beside the reply command; an unmarked header is unchanged.
`read --envelope`'s message fields gain `expect_reply: true` on a marked message.
`log`'s entry fields gain `expect_reply: true` on an entry that carries the mark and `reply_to` on a reply's entry.
The `post` Long help and `internal/creelcli/doc.go` name the flag and the header mark.

Bound: the flag adds no privilege, since it marks a post the sender could already make to a receiver the daemon's usual rules admit.
What the mark sets in motion, from card 7 on, is at most two content-free `CREEL` chases to the holder and one notice to the sender per marked message, and only once the holder has read it.

`TestWriteMessagesText` gains a row for a marked message's header.
`TestCreelCLI` gains rows: `post --expect-reply` journals a `posted` event whose `log` entry carries `expect_reply`, and the receiver's `read --envelope` hands the message out with `expect_reply: true`; a reply's `log` entry carries `reply_to`; `post --expect-reply --reply` is refused before any call.

**ImpactSummary:** One new post flag and three output fields; existing output of unmarked messages is unchanged.
