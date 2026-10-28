# Card 4 — Every line a claim can hand out fits one hook field

**Create:**
- `internal/creelengine#LineFits`
- `internal/creelengine#validateLine`
- `internal/creelengine#hookFieldCap`
- `internal/creelengine#Store.PostUnchecked`

**Edit:**
- `internal/creelengine#maxBodyBytes`
- `internal/creelengine#Store.post`
- `internal/creelengine#Store.Reply`
- `internal/creelengine#Store.ClaimAtBoundary`
- `internal/creelengine#Store.flaggedFilesLocked`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestStore_Mail`
- `internal/creelengine_test#TestStore_ClaimAtBoundary`
- `internal/creelengine_test#TestStore_MailFlags`

**Uses:**
- `internal/creelengine#MessageLine`
- `internal/creelengine#boundaryClaimBudget`
- `internal/creelengine#validateBody`
- `internal/creelengine_test#newFlaggedEnv`
- `internal/creelengine_test#newEnv`

**Intent:**
D1 and the added decision "A post body is capped at the hook field budget": `ClaimAtBoundary` always hands out its first message whatever its size, so only a cap at `post` keeps every claim inside one hook field.

`LineFits(from, body)` reports whether the `MessageLine` of a message from `from` with that body, rendered with the widest id an int64 takes (`math.MaxInt64`, so the answer never depends on which id the post gets), is at most `boundaryClaimBudget` bytes.
A line of at most 8 KiB is at most 8,192 UTF-16 units, the measure Claude Code caps a hook field by, so a fitting line always arrives whole.
It is exported because LOOM's decision notice (card 5) measures its own notice before it posts.
`validateLine(from, body)` refuses an empty body as `validateBody` does, and a body whose line does not fit with `creel: the message's line is <n> bytes, over the <budget>-byte limit one hook delivery carries; write it to a file and mail the file's path`, the existing way forward.
`Store.post` validates with `validateLine(sender.Name, body)` in place of `validateBody`, so a post, `--wait-reply` included, that would not fit stores nothing.
`Store.Reply` moves its validation inside its locked section, after it finds the original: a reply to an `EXTERNAL` sender, which reaches only a waiting `post --wait-reply` and is never claimed, keeps `validateBody` and its 64 KiB cap, and every other reply takes `validateLine`.
`maxBodyBytes` stays 64 KiB, now documented as the cap of a reply to EXTERNAL alone; Shared Decisions records why the added decision's "lowered to 8 KiB" is met by the line cap rather than by lowering the constant.
`forward` keeps the line it forwards: it moves a message already stored and validates nothing new.

A message an old binary stored can carry a line past Claude Code's 10,000-character field cap, `hookFieldCap`, where Claude Code shows a preview while creel would journal the message read.
`ClaimAtBoundary` skips every message whose `MessageLine` is longer than `hookFieldCap` UTF-16 units (`len(utf16.Encode([]rune(line)))`), so it is never claimed and stays for `lym creel read`; a line over the 8 KiB budget and within the field cap is still claimed alone, as before.
`flaggedFilesLocked` asks for no flag for a strand whose only unread mail is such messages, so its PostToolUse hook stops starting `lym` at every tool call for mail no claim hands out; the mail still shows in `lym creel inbox`.
`PostUnchecked`, in `export_test.go` beside `ReadThenFail`, journals a posted record and delivers it as `post` does but without the line check, as an old binary did, so a test can hold such a message.

Tests:
`TestStore_Mail`'s `Post_RefusesEmptyAndOversizedBody` row asserts a body whose line is one byte over the budget is refused with "write it to a file" and "mail the file" and stores nothing (the strand's read finds no message), and that a body whose line fits the budget exactly is accepted; a new row asserts a strand's reply over the budget is refused and stores nothing; the existing `Reply_ToExternalGoesToTheRegisteredWaiter` row sends its reply with a 9 KiB body, so it also pins that a reply to EXTERNAL over the budget is accepted and handed to the waiting `PostWait`.
`TestStore_ClaimAtBoundary`'s `StopsAtTheBudgetButHandsOutAtLeastOne` row posts its third, over-budget message through `PostUnchecked` with a 9,000-byte body, since `post` now refuses it; a new row stores, through `PostUnchecked` beside a short post, one body whose line is exactly `hookFieldCap` UTF-16 units and over 10,000 bytes, its body made of two-byte characters such as `é`, which is still claimed alone, pinning that the cap counts UTF-16 units and not bytes, and one whose line is `hookFieldCap` plus one unit, and asserts that one is never claimed, a later claim hands out nothing more, and `Read` still returns it.
`TestStore_MailFlags` gains a row: a strand whose only unread message is such a long one has no flag.

**ImpactSummary:** the post and reply validation and the claim's selection change inside creelengine; posts between the old 64 KiB and the new line budget are now refused, and every test posting such a body is in this card.
