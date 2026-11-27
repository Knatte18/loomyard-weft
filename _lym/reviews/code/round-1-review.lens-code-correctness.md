```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff changes, using one concrete input per case.

**Cases run:**
- **`Desktop.Add`, Freedesktop, `Add("/opt/lym dev/lym", "ly")`:** the hub launcher is `content(lymPath, "ly", "ly")` and `verb` is `machine start ly`, so `Name=lym: ly` and `Comment=Runs lym machine start ly on this machine`. The all-hubs launcher keeps `content(lymPath, "start every hub", "")`, giving `Name=lym: start every hub` and `verb` `machine start`. Correct.
- **`Desktop.Add`, empty shortname:** gives `Name=lym: ` and the file `lym-hub-.desktop`, and `Shortnames` skips the empty name. The old code also wrote `lym-hub-.desktop`, named `lym: start the hub `, so the diff introduces nothing new here. The only production caller, `dobbyengine` `addLauncher` (`internal/dobbyengine/launcher.go:44`), returns early on an empty shortname, so this input never reaches `Add`. Not a defect.
- **`Desktop.Add`, failure paths:**
  - When the hub launcher's `content` returns the WSL quote error, `Add` returns before taking the lock or writing, so the desktop stays empty. Correct, and unchanged.
  - The all-hubs launcher's `content` cannot fail on a path the hub call already accepted, since the check is `lymPath+shortname` and the all-hubs shortname is empty. Correct.
  - When a write fails, `Add` returns the wrapped error, which can leave one of the two launchers written. That behaviour predates the diff and is outside it.
- **`content`, `FormFreedesktop`, `title="ly"`:** gives `Name=lym: ly`, and `Comment=` comes from `verb`, not from `title`. Correct.
- **`content`, `FormWindowsWSL`:**
  - Shortname `ly` with Distro `Ubuntu-24.04`: `cmdFile("ly", wsl.exe -d "Ubuntu-24.04" -- bash -lc "'…' machine start ly")`. Correct.
  - Shortname `a"b`: refused before `cmdFile` runs. Correct.
- **`content`, default (`FormWindows`):** gives `cmdFile("ly", "\"C:\\…\\lym.exe\" machine start ly")`. Correct.
- **`cmdFile`:**
  - Title `ly`: the lines are `@echo off`, `rem lym: ly`, run, `pause`, then a trailing empty line, joined with CRLF.
  - Title `50%`: the rem line becomes `rem lym: 50%%`. `%` is still doubled across the whole file, the same as before the rename.
  - Correct in both cases.

**Findings:** none.

**Tool calls:** none. Everything I traced was already in the inherited context.
