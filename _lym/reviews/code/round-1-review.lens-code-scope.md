```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the goal points of the board entry, the answer key here being a decision record alone.

**Decisions.** These are the goal points of `## Goal`. The record has no `Added after Discussion` entries.
- **G1:** the hub launcher's name "lym: start the hub odd" is too long, and the operator needs mainly the repository's name.
- **G2:** for now each hub's launcher is named "lym: &lt;shortname&gt;", such as "lym: odd".
- **G3:** later, once the launcher has a lym icon, the name may be the shortname alone. This is stated as future work and asks for no change now.

**Per change:**
- **C1:** `Desktop.Add` passes `shortname` as the title instead of `"start the hub "+shortname` (`internal/desktoplauncher/desktoplauncher.go:52`). Traces to G2, and through it G1.
- **C2:** the parameter `purpose` becomes `title` in `content` and `cmdFile`, and their doc comments are reworded (`desktoplauncher.go:137-171`). Forced by C1, since the argument is no longer a purpose phrase.
- **C3:** the `TestAdd_WritesLaunchersRunningMachineStart` rows expect `Name=lym: ly` and `rem lym: ly` (`desktoplauncher_test.go:105,118,127,136,167`). Forced by C1.
- **C4:** a `doc.go` line names the displayed names of the hub and all-hubs launchers (`internal/desktoplauncher/doc.go:19`). Forced by C1, as a doc of the changed behaviour.
- Every change traces to a decision or is forced by one. The diff adds no unrequested scope. The all-hubs launcher keeps "lym: start every hub", which the entry does not ask to change.

**Per decision:**
- **G1:** built by C1.
- **G2:** built by C1 for the freedesktop form only (see finding 1). C2–C4 support it.
- **G3:** deferred by its own wording. No change is expected and none was made.

**Findings:**
1. **LOW, scope.** On the `.cmd` forms (`FormWindowsWSL` and `FormWindows`), G2 is only partly built.
   - **What:** C1 changes only the `rem lym: &lt;title&gt;` line, and `@echo off` keeps that line out of the window. The name a Windows desktop shows is still the file name `lym-hub-&lt;shortname&gt;`, so no Windows user sees "lym: &lt;shortname&gt;".
   - **Why only LOW:** a colon is illegal in a Windows file name, so "lym: ly" could never be the file name. The old name a Windows user saw, `lym-hub-ly`, was never the long one the wish complains of.
   - **Evidence:**
     - `internal/desktoplauncher/desktoplauncher.go:168-170`: `cmdFile` lines `"@echo off", "rem lym: " + title`.
     - `desktoplauncher.go:107-109`: `hubPath` is `hubPrefix+shortname+d.extension()`.
     - `_lym/discussion/decision-record.md:7`.
   - **Fix:** neither the code nor a doc change can reach this. Either accept the freedesktop-only reach, or have the parent record a decision that Windows forms keep the file name.

Tool calls: none; my inherited context already held the diff, the decision record, the source and the callers.
