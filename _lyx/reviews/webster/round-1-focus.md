---
round: 1
focus:
  - "Cards 5-7: do placement, anchor and focus decisions all read `listSessionWindows` as the plan's shared decision says, with no code that sorts or renumbers windows?"
  - "Cards 6-7: is `isOwnWindow` widened to cover `overview` in every liveness, reconcile and resolver path, while only placement, name, unseen mark, one-per-session refusal and focus rules ask `isOverview`?"
  - "Card 7: does every attach and automatic switch fall back to the overview and never land on the shell, including when a viewed window ends and when a hidden window is the only one left?"
  - "Card 5: does the fix for sessions from an older build leave window order correct on a session created before the change, and is it covered by a test?"
  - "Card 4: does the watchdog skip re-apply and the zoom bracket only for the last applied or last failed box, and does a changed box still re-apply?"
  - "Cards 1 and 3: does `internal/selfexec` stay a leaf with no upward imports, and do the overview panes re-exec only on a changed binary, with no loop on an unchanged one?"
  - "Card 8: do Alt+Up and Alt+Down step in window index order across visible windows only, and does the `shell` button replace VIEW everywhere VIEW was referenced?"
  - "Test economy: do new tests in cards 5-9 duplicate coverage already given by a public-surface test, or leave a added behavior unpinned?"
---

Focus is set from the plan's two shared decisions and the cards that carry the most behavior change.
