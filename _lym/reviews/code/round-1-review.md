---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/desktoplauncher/doc.go:19
    summary: "doc.go says a hub's launcher shows as `lym: <shortname>` on every form, but a `.cmd` launcher shows only its file name `lym-hub-<shortname>.cmd`; its `rem` line, hidden by `@echo off`, is never displayed"
    fix: "In internal/desktoplauncher/doc.go:19, say that a `.desktop` launcher shows its `Name=` key, `lym: <shortname>` for a hub and `lym: start every hub` for the all-hubs one, and that a `.cmd` launcher shows its file name and carries the same title on its `rem` line. In _lym/landing/summary.md, change \"The name shows in ... the `rem` line of the Windows `.cmd` file\" to say that the `.cmd` file carries the title on its `rem` line, while the Windows desktop shows the file name."
    origin: handler
  - id: F2
    severity: NIT
    class: consistency
    location: internal/desktoplauncher/desktoplauncher.go:138
    summary: "content's doc says title follows \"lym: \" in the launcher's displayed name and its comment line, which reads as both lines in each form; in a `.desktop` file the `Comment=` key never carries title, and a `.cmd` file has no displayed name line"
    fix: "Reword the second line of the content doc comment at internal/desktoplauncher/desktoplauncher.go:138 to: title follows \"lym: \" in a `.desktop` launcher's `Name=` key and in a `.cmd` launcher's `rem` line."
    origin: handler
---

### [LOW] doc.go claims the `lym: <shortname>` display on every form

**Location:** internal/desktoplauncher/doc.go:19

**Issue:** The new line "A hub's launcher shows as `lym: <shortname>`, and the all-hubs launcher as `lym: start every hub`" holds only for the freedesktop form, whose file manager displays `Name=`.
On `FormWindowsWSL` and `FormWindows` the desktop displays the file name, which `hubPath` builds as `lym-hub-<shortname>.cmd`.
The title sits on a `rem` line under `@echo off` (`desktoplauncher.go:169`), so no Windows user sees it.
The summary's claim that "the name shows in ... the `rem` line" is not borne out for the same reason.

### [NIT] content's doc comment blurs which line carries title

**Location:** internal/desktoplauncher/desktoplauncher.go:138

**Issue:** "its displayed name and its comment line" suggests that each launcher has both lines.
In a `.desktop` file, "comment line" reads as the `Comment=` key, which comes from `verb` and never from `title`.
The `.cmd` form has a `rem` line but no displayed-name line.

## Rejected

- code-scope's LOW finding, that the board entry's "lym: <shortname>" goal is only partly built for the `.cmd` forms: not a defect of the diff. The long "start the hub" text only ever displayed through the freedesktop `Name=` key. The Windows desktop showed `lym-hub-<shortname>` before and still does, and a Windows file name cannot hold the colon. The diff makes the change the entry asks for on every form where a launcher carries a displayed title. The inaccurate doc claim this lens's evidence points at is kept as F1.

## Focus notes

- Focus 1: freedesktop users see `Name=lym: <shortname>` (`desktoplauncher.go:149`), which meets the entry; `.cmd` users see the file name and `rem lym: <shortname>` is never displayed (F1, and the scope item rejected above).
- Focus 2: the summary's `rem` "shows" claim is F1. Its in-place rewrite claim holds: `dobbyengine`'s `addLauncher` calls `Desktop.Add` on each dobby start (`internal/dobbyengine/launcher.go:64`), and `Add` overwrites both files with `os.WriteFile` because their paths are unchanged.
- Focus 3: `cmdFile`'s doc is accurate. `content`'s doc is F2. doc.go names the all-hubs title but makes the same every-form claim (F1).
- Focus 4: a repository-wide search finds nothing outside the diff that still reads or states "start the hub <shortname>". The only matches are unrelated Slack-relay wording and the board's own title for this task.
