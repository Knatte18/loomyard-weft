```yaml
lens: code-scope
outcome: reported
```

code-scope lens: trace each change in the diff to a goal point of the board entry, and each goal point to the changes that build it.

**Decisions** (the board entry's `## Goal`; the record has no `Added after Discussion` entries):
- **Part 1:** an update mode for the settings golden that rewrites the file from the rendered settings and deny notice, reachable through `lym gate test`.
- **Part 2:** a way for a shellguard row to state an exception that still fails closed. If rows stay AND-only, `unquotedWordOtherThan` becomes the documented way to write one.

**Per change:**
- **C1** — the `-update` flag `updateSettingsGolden`, the `settingsGolden` type, and rendering every combination before the write-or-compare branch (`internal/shuttleengine/claudeengine/settings_test.go`, diff 002 lines 17-112). Traces to Part 1.
- **C2** — the package doc paragraph on how a row spells an exception and why a must-not-match pattern would fail open (`internal/shellguard/doc.go`, diff 001 lines 9-17). Also the `Row.Patterns` field comment (diff 001 lines 33-34) and the `unquotedWordOtherThan` doc (diff 001 lines 42-45). Traces to Part 2's AND-only branch, which makes the helper the documented way.
- **C3** — the package doc paragraph naming the golden's `-update` command (`internal/shellguard/doc.go`, diff 001 lines 19-20). Traces to Part 1: it makes the update mode findable from the package where the row changes that call for it are made. That is the issue's stated cost ("each agent making one has to write and then remove its own renderer").
- **C4** — the row `system_flag_of_a_later_dbus_send` (`internal/shellguard/rows_integration_test.go:316`). It is forced by C2: it pins the fail-closed property C2 documents, which Part 2 requires ("an exception matched anywhere in the payload would let a second, refused command in the same call through").
- **Commit `4a559d784`** (comment line breaks) only re-wraps the lines C1 to C3 added. It is part of those changes, not a separate purpose.

**Per decision:**
- **Part 1** is built by C1 and C3. C1 reaches `lym gate test` through the words after `--`: `internal/gatecli/cli.go:108` splits them off, and `tierrun.go:526` runs the go test in the tree root.
- **Part 2** is built by C2 and C4. The diff takes the alternative the entry licenses ("If rows are to stay AND-only"). The commit message of `17333cd34` and the summary's Design decisions say why the negative-pattern option was turned down. No `Row` field was required.

**Findings:** none. Every change traces to Part 1 or Part 2, or is forced by one that does. Both goal points are built, and the diff makes no unrequested change.

**Tool calls:** none. The inherited context held the diff, the record, the commit log and the gate arg-forwarding code.
