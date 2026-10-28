# Card 3 — Creel quotes every sender name in the lines it writes

**Edit:**
- `internal/creelengine#MessageLine`
- `internal/creelengine#composeReminder`
- `internal/creelcli#writeMessagesText`
- `internal/creelcli#hookReadCmd`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestStore_ClaimAtBoundary`
- `internal/creelengine_test#reminderFor`
- `internal/creelengine_test#TestScheduler_Reminders`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/creelengine_test#TestSweep_RemovedHolders`
- `internal/creelengine_test#TestDaemon_Lifecycle`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli#TestWriteMessagesText`

**Intent:**
D8: a strand name keeps its `:`, so a name read inside prose is ambiguous; creel quotes every sender name it writes, a program and `EXTERNAL` included, so each line has one shape.
`MessageLine` renders `[creel] #<id> from "<sender>": <body>`, with its control-character folding unchanged.
`composeReminder`, the line the pane path still types for a strand whose engine has no hook delivery, lists each message as `#<id> from "<sender>"`.
`writeMessagesText`, the text `lym creel read` prints, opens each header `#<id> from "<sender>"`, the rest of the header unchanged.
The hidden `hook-read` verb's Long text and the package doc's notice paragraph show the quoted line form.
The sender is quoted with a plain `"` on each side and no escaping: a sender name is an agent name, which agentname forms from a closed character set holding no `"`.
Every test asserting the old form is updated in this card, found by grepping for `[creel] #`, `from " +`, `#%d from` and `unread: #`: the claim rows of `TestStore_ClaimAtBoundary`, the expected lines `reminderFor` builds and the notice and reminder lines of `TestScheduler_Reminders`, `TestScheduler_Injects`, `TestSweep_RemovedHolders` and `TestDaemon_Lifecycle`, the `read` and `hook-read` rows of `TestCreelCLI`, and every expected header of `TestWriteMessagesText`.
Those updated rows are the coverage: `TestStore_ClaimAtBoundary` pins `MessageLine`'s quoted form, `TestWriteMessagesText` pins `read`'s, and `TestScheduler_Reminders` pins the reminder's.
The smoke tests match only the prefix `[creel] #1 from `, which the quoted line still holds, so they need no change here.

**ImpactSummary:** three line formatters change their rendered text; every consumer is a test assertion or a doc in creelengine and creelcli, and no code parses the line.
