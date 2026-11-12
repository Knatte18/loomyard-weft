```yaml
lens: fix-claims
outcome: reported
```

Task: check the fix-claims lens, meaning every bound, behaviour and code fact the fix added to the decision record, against the code at HEAD.

**Findings**

1. MEDIUM. D4: "When the setup's operator changes, the relay invites the new operator into every channel of the bridge map, as it does into orch channels."
   The orch-channel mechanism can't be reused as the sentence implies, because it depends on a field the bridge map doesn't have.
   Evidence: `internal/slackengine/keeper.go:364` says `inviteOperators` "invites the setup's operator into every open entry that records another operator or none". It relies on `ChannelEntry.Operator` (`internal/slackengine/channelmap.go:34`) to tell which channels still need the invite. D4's revised field list for the bridge map ("id, local orch, peer address, delivery cursor, operator cursor, thread cursors and thread floor") has no operator field. So "as it does into orch channels" claims a behaviour the record gives the relay no means to carry out: it can't tell an operator change from a channel already invited.

2. LOW. D8: "An envelope message posted longer ago than creel's journal retention, seven days, is skipped, since creel's dedupe holds ids only within it."
   "Only within it" overstates the case. Retention carries unread and chaseable messages forward past seven days (`carryRecords`, `internal/creelengine/fold.go:704-721`), so the dedupe can hold some ids longer. The direction the bound needs does hold: an id delivered within retention is still held, because day files are deleted only before the cutoff day (`journal.go:68`, `:343`). The reason given is inexact, though the rule it supports is sound.

**Claims checked and found to hold**
- `FindChannel` lists archived channels (`api.go:284`).
- The relay never unarchives (`keeper.go:196`).
- The holder mails its relay notices to the prime orch (`holder.go:529-530`).
- Journal retention is seven days (`journal.go:68`).
- A fold that doesn't know an event or carried `of` ignores it (`fold.go:453-455`, `:624-626`) but tracks its id in `maxID` (`fold.go:372`). That makes the "id only" and "starts no worker" claims hold.
- Orch-channel threads are read through thread cursors and a thread floor (`channelmap.go:102-113`).

**Tool calls**
- grep of `fold.go` for `case eventCarried`: I needed to know how an older fold treats an unknown carried `of`.
- grep for the `applyCarried` start: same question.
- Read of `fold.go:602-626`: the default branch of `applyCarried`.
- grep for `maxID` in `fold.go`: to confirm an ignored record still advances the id high-water mark, which the "id only" claim depends on.
