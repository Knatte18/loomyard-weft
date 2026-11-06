```yaml
lens: fix-consistency
outcome: reported
```

Task: check fix F1 (restoring the drop window when the notice post fails) for agreement with its surroundings.

**Findings**

1. **LOW** — the doc comment on `noticeDropLocked`, `receive.go:317`.
   - The comment says a failed notice "leaves the window as it was".
   - The code restores `notices` and sets `undisclosed = counted`.
   - `undisclosed++` for the current drop is kept, and a window newly created for this call is kept too.
   - So the window is not left as it was. It holds the failed drop in `undisclosed` and has the same `notices` as before.
   - Suggested wording: "a notice that fails to store is logged, spends no notice slot, and its drops are counted into the next notice".

2. I found no other disagreement.
   - The twin phrases about a failed notice in `doc.go:299`, `chase.go:232` and `outbound.go:131` concern other notices that follow a record.
   - They say the notice "never undoes the record", which still holds here. The drop stays journaled.
   - The `dropWindow` comment at `pool.go:68` is unaffected.
   - Nothing in the docs states a "five notices per window" rule that the restore contradicts.

**Tool calls**
- One `grep -rn` over `.go` and `.md` files for phrases like "undoes nothing", "notice slot" and "drop notice", to find twin comments and docs about a failed notice. My context did not hold the other files.
