```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, I traced what calls, reads or asserts it, and checked whether the diff leaves any of those wrong.

**Per change, the set found:**

1. **`loomengine.ClassifyHalt` gains a `transient` parameter.**
   - The only production caller is `internal/loomcli/halt.go:74`, and the diff updates it. The only test caller is `internal/loomengine/anomaly_test.go:272/274`, also updated. Covered.
2. **New anomaly kind `transient-stop` (`AnomalyTransientStop`).**
   - `internal/haltnote/haltnote.go:64-65,124-125` writes and parses the `anomaly:` value as an opaque string. No change is needed. Covered.
   - `internal/reportengine/stops.go:150` passes the note's anomaly through as the stop class, with no switch over kinds. No change is needed. Covered.
   - `internal/loomcli/doc.go:12-16` lists the kinds, and the diff adds the line. Covered, but see the finding below.
   - `contracts/stencils/friction/friction-template-reflection.md:48-53` lists the drop rules, and the diff adds the rule. The template's input list already names "The loom driver's repair and re-step records", so the new rule's condition can be read from them. Covered, but see the finding below.
   - No kind list exists in `_bolt/_lym/specs`, `PATTERN.md`, `pattern/`, `internal/loomengine/doc.go` or `contracts/specs`. Nothing is missing there.
3. **`haltNote.Transient` field, set in `failedHalt`.**
   - Both callers of `failedHalt` (`internal/loomcli/halt.go:144`, `internal/loomcli/arm.go:569`) get it automatically. Covered.
   - The interrupted note at `internal/loomcli/looparm.go:317` never sets it, and an interrupted stop takes precedence anyway. Correct.
4. **Parent-lock wait log lines in `internal/landingshed/parentguard.go:61,64,81`.**
   - Nothing reads these messages. The similar strings at `internal/loomcli/reviewstatus.go:176,189` and `internal/fabriccli/anchordeps.go:564` are separate outputs, and the diff leaves them unchanged. Covered.
5. **Stencil wording, commit messages and PATTERN rules.**
   - The stencil change is checked against the amended entry, which allows "one with its own anomaly kind". It drops or weakens no earlier rule.
   - The range has no `Pattern-Departure:` trailer.
   - `Root-Cause:` is present on `ba7351c0b` and names the defect the diff removes.
   - No PATTERN changes were declared, so none needed to land.
   - PATTERN-shed-driver is untouched, because the transient mark is still read only from shedengine's status.

**Findings**

- **MEDIUM, consistency.** Location: `internal/loomengine/anomaly.go:34-36`, `internal/loomcli/doc.go:15-16` and `contracts/stencils/friction/friction-template-reflection.md:51`.
  - All three say a `transient-stop` halt is "a network or host failure ... the driver re-steps". The new kind fires for any non-empty transient class (`internal/loomcli/halt.go:127`, `Transient: st.Transient != ""`).
  - That includes `agent-start`, an agent session that never became ready (`internal/shedengine/transient.go:25`). This is neither a network nor a host failure.
  - The driver does not re-step `agent-start` or `host-resources`. `networkFailure` is true only for `git-transport` and `github-api` (`internal/sheddriver/driverules.go:307-316`). Any other transient class falls to "a transient error that outlived its re-step: the tackler decides" (`internal/sheddriver/driverules.go:296-297`), after the loop's single re-step (`internal/shedverbs/loop.go:463-466`).
  - The reflection agent therefore reads a drop rule whose stated reason is wrong for those classes.

Tool calls:
1. Grepped `ClassifyHalt` callers, `Anomaly*` users, reportengine anomaly handling, and read reflection template lines 30–56. The context had only the diff hunk of the template and no caller search.
2. Grepped docs, `pattern/`, `PATTERN.md`, haltnote, friction doc and specs for kind lists, to find rule documents that enumerate anomaly kinds.
3. Grepped readers of the new log messages and transient mentions in the friction stencils, to find any consumer of the new log lines.
