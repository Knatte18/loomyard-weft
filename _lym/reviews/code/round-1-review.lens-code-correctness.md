```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function and branch on concrete inputs.

**Per-function list**

- `logBootWindowLocked` (lifecycle.go:567), fresh boot, answer `"@0 %0 bash\n"`: `TrimSpace` then `SplitN(" ", 3)` gives 3 fields, so it logs Info with window `@0`, pane `%0` and name `bash`. Correct.
- `logBootWindowLocked`, window name with spaces, e.g. `"@0 %0 my win"`: `SplitN` limit 3 keeps `"my win"` whole. Correct.
- `logBootWindowLocked`, empty answer with a nil error: `fields` is `[""]`, length 1, so it logs Debug and returns. Correct.
- `logBootWindowLocked`, `display-message` fails: `err != nil`, so it logs Debug and returns, and the boot continues. Failure is non-fatal. Correct.
- `logBootWindowLocked`, target `=&lt;session&gt;:`: `serverPIDLocked` (lifecycle.go:1059) already passes this same target to `display-message -p` on both platforms, so it reaches the session's current window and active pane. A fresh session has exactly one window, so the target is unambiguous. Correct.
- `logBootWindowLocked` call site (lifecycle.go:545): it runs only after the boot loop confirms `hasSession`. Early returns (session already up and usable, or any boot error) never reach it, so it logs only on a real fresh spawn. In the zero-pane husk case, `kill-session` then a fresh boot falls through to the log, which is correct. The log runs before `guardBootSessionLocked` and `pinBootOptionsLocked`. If either of those later fails, the Info line has already been written. That is harmless, and the line states a fact.
- `windowListFormat` and `findWindowLocked` (window.go:169), row `"@4 %9 0 1 batten:slug"`: the hidden flag is `"1"` and the name matches, so it returns the live window. Correct.
- `findWindowLocked`, row `"@4 %9 0 0 batten:slug"` (hand window): the hidden flag is `"0"`, so it continues and returns nil. `OpenWindow` then skips both the reuse and the kill, and opens a new window. Correct.
- `findWindowLocked`, empty output: no line yields 5 fields, so it returns nil, nil. Correct.
- `findWindowLocked`, name containing spaces: `SplitN(…, 5)` keeps the full name in field 4. Correct.
- `findWindowLocked`, duplicate names (a hand window plus a hidden window of the same name): tmux allows duplicate window names, and the filter skips the unhidden one and selects the hidden one. Correct.
- `findWindowLocked`, `list-panes` fails: it returns a wrapped error, which `OpenWindow` propagates. Correct.
- `OpenWindow` dead-window branch, input `"@4 %9 1 1 name"`: `existing.paneDead` is true, so it runs `kill-pane` on `%9` and then creates a new window. Correct.
- `OpenWindow` dead hand-window branch, input `"@4 %9 1 0 name"`: `findWindowLocked` returns nil, nothing is killed and a new window opens. Correct.
- `OpenWindow` failure between `new-window` and `@lym_hidden` (window.go:116-119):
  - The chain is one tmux invocation, and tmux stops a command list at the first failing command.
  - A failure at `set-option` therefore leaves a window with the temporary name `lym-opening-…`. That window is neither adopted nor replaced, and the new lookup skips it for lack of the marker.
  - Before the diff, the old lookup skipped only `@lym_own` windows and would have skipped it too, as the temporary name never equals `name`. So the diff introduces no regression here.
  - A new `new-window` creates the window with the temporary name before any marker is set, so the hidden marker is not set "from creation" in a strict sense. The window is invisible to name lookup throughout, so no wrong adoption or kill follows.
- `openOwnWindowLocked` log (spawn.go:328): `name` is reassigned to the overview title before the log for an overview, and the added field holds the name the window was opened with. Correct.
- `moveShellWindowStrandsLocked` log (ownwindow.go:439): `strandLabel(s.Name)` is the same expression passed to `break-pane -n`. Correct.
- Hidden-marker writers: `@lym_hidden` is set only at window.go:118, and the reconcile `kill-window` at reconcile.go:262 is outside this diff. I found no window that carries `@lym_hidden` and is also reused by name wrongly.

**Findings:** none in this lens.

**Tool calls:** none made. The manifest already held every definition this lens needed.
