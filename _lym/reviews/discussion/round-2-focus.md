---
round: 2
exclude_lenses: []
focus:
  - "Waiting and ordering, in decision-record.md D8 and D10: for each wait (own window in KWin, parent window or placement, per-user lock), what signal does it read, does reading it need a KWin script run and may that run outside the lock, what are its bound and wake rate under PATTERN-daemon-wakeups, and can a child place itself against a parent that is not yet placed, whatever order the lock is taken in?"
  - "The per-user lock in D10: is its location and the accessor naming its path stated, consistent with PATTERN-lym-files, and does it serialize placements started from different hubs?"
  - "Startup order, in D12 and D8: internal/machinecli/start.go runs runBeamStarts before openWindows, and beam start brings up dobbies whose batten reopens run lym reed window (internal/battencli/arm.go). Do the record's ordering claim and parent-wait bound hold for a first start with no remembered places?"
  - "The places file in D9, with D6 and D11: how concurrent writers, inside and outside the lock, keep each other's entries; which fields each write changes; what an overflow to another desktop records when a smaller screen is later replaced by the larger one; and how renumbered order keys relate to those of windows not yet reopened."
  - "Failure paths, in D1, D3, D10 and D12: on a pre-launch check failure, a post-launch rootTile probe failure, a script run timeout, a lock or window timeout, and a child ended by runproc.End mid-apply, what happens to the Konsole command line, the loaded KWin script, the desktop's tiles and window_command (which lym reed window always runs today and lym machine start never does), and which side does mode off fall on?"
  - "KWin and Konsole facts the design rests on, in D2, D3, D4, D6 and Constraints: is each recorded with an observed result and where it is pinned? Cover building a grid and calling manage on a desktop that is not current, moving a window to the target desktop and tiling output, kscreen-doctor output names against workspace.screens names, how the rootTile probe distinguishes a two-argument rootTile, Konsole's pid and caption, and why --separate is needed."
  - "Tiling geometry in D6: what happens to a tile that runs under the panel (~/.local/bin/arrange-to-tiles.sh clamps such windows), and does the window-count rule agree with its stated rationale about that script?"
---

Every lens that ran has findings at MEDIUM in its area, so none is excluded.
