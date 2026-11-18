```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking that every claim the fix added or rewrote holds against the code at HEAD.

**[MEDIUM] D2 "or, with none live, nowhere, leaving the choice to tmux"**
- **Changed text:** the new D2 sentence "A viewer on the window the checklist leaves is moved by reed's rule for a viewed window that ends: to the newest live agent window, or, with none live, nowhere, leaving the choice to tmux."
- **Problem:** in case 2 the runs strand's overview window already exists. In that case reed's rule falls back to the overview window, not to nowhere.
- **Evidence:**
  - `relevantWindow` returns the newest live agent window, "else the overview strand's window, else the empty string" (`internal/reedengine/ownwindow.go:453-470`).
  - `ReplaceStrand` with a non-overview spec passes `selectOnEnd=true` (`internal/reedengine/replace.go:71`), and `killStrandPanes` then calls `selectRelevantWindowLocked(st, endingWindows)` (`internal/reedengine/strand.go:854-856`).
  - In D2 case 2 (runs strand present, dead `loom-status` in its own window), a viewer on the dead checklist window with no live agent therefore lands on the beam overview.
  - "With none live, nowhere" holds only in case 1, where no runs strand exists yet.

**[MEDIUM] D2 "The runs strand's new overview window is never selected"**
- **Changed text:** the new D2 sentence "The runs strand's new overview window is never selected, and a viewer on any other window stays where it is."
- **Problem:** this is false on D2's own fallback path.
- **Evidence:**
  - D2's first failure bullet keeps today's replace: "a `loom-status` strand still present is replaced by the runs strand, as today".
  - That replace has an overview spec. When it ends the viewed window, `ReplaceStrand` selects the new overview window: `if endedCurrent && replacingWithOverview { e.selectStrandWindowLocked(strand.PaneID) }` (`internal/reedengine/replace.go:97-98`).
  - The "never selected" claim holds only for `AddStrand` (`openOwnWindowLocked` doc, `internal/reedengine/spawn.go:255`).

The other claims I checked hold:
- **Grouped placement in D3:** `planOwnWindowPlacement` places a strand apart when its group is not shared, or when `liveCount >= size`. A strand that opens its group's shared window opens it under the group's name (`openOwnWindowLocked(st, s, group, group)`, `spawn.go:197`).
- **Window-name collision in D3:**
  - Own-window options, selection and layout target the window id.
  - Status-bar buttons use the range `window|#{window_index}` and only display `#{window_name}` (`bar.go:36,43`).
  - The shell window is found through the console pane (`shellWindowOf`).
  - `OpenWindow` skips windows carrying `@lym_own` (`doc.go:389`).
- **D4 criterion:** its new wording matches the D4 text.

Tool calls:
- Bash, `sed` on `internal/reedengine/strand.go:837-870` plus a grep for window-name lookups in reedengine: my context did not hold `killStrandPanes`'s body, and I needed to rule out reed finding an own window by name.
