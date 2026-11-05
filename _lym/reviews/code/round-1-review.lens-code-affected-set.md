```yaml
lens: code-affected-set
outcome: reported
```

**List: per change, the set found, covered or not**

1. **`promoteTo` defaults a plain promote of a discussed note with a stored record to raddle.**
   - Set found:
     - Production callers: `Board.PromoteTo` and `Board.Promote`, plus the `boardcli` `promote` verb. No other production caller of `promote` or `promoteTo` exists. `Store.Promote` passes a nil `recordStored`, so it is unchanged.
     - Docs: the `PromoteTo` comment (`board.go:413-417`), the `promoteTo` comment (`store.go:764-767`), `boardengine/doc.go:322-325`, `boardcli/doc.go:78-80`, the `promote` help text (`cli.go:170-177`), the flag text (`cli.go:203`), `docs/overview.md:369` and `SKILL.md:98-99`.
     - Tests: `TestDiscussedPromotion` (`sketch_test.go`), `slugrule_test.go:219,248`, `raddlefile_test.go:54`.
   - Covered: all of these were updated or still hold.

2. **`SketchPromoteCommand` is the JSON-slug way-forward command, used by every refusal.**
   - Set found: `sketch.go:114`, `labels.go:67` and `startguard.go:172`.
   - Their assertions are updated in `sketch_test.go` (`wayForward`), `labels_test.go:31` and `startguard_test.go:458`.
   - Covered.

3. **The help, flag and doc wording changed from "only through `--recipe raddle`" to "plain promote, `--recipe raddle` accepted".**
   - My repo-wide grep (call 1) finds no remaining text that names `--recipe raddle` as the required form.
   - `PATTERN.md` and `pattern/` show no diff in the range, and the directive declares no PATTERN change, so nothing is owed there.
   - The one commit has no `Pattern-Departure:` trailer, and none is needed.
   - Covered.

**Finding (leaves a member of the set wrong)**

- **MEDIUM, `design`.** The new refusal text points a dead-end case back at itself, and the doc comments that describe that case are wrong.
  - Location: `internal/boardengine/labels.go:66-67`, `internal/boardengine/doc.go:327`, `internal/boardengine/labels.go:52`.
  - The case: a discussed note that carries the `sketch` label but whose record file is missing.
  - Why it fails:
    - A plain promote gives raddle only when `recordHeld` is true (`store.go:783-785`), and here it is false.
    - The task therefore keeps recipe `""` and the `sketch` label, and `checkSketchLabel` refuses it (`labels.go:57-67`).
    - The prior is a discussed note, so the refusal names `lym board promote '{"slug":…}'` "which gives it recipe raddle". That is the command just run.
    - Before the diff the named command, `--recipe raddle`, succeeded in this case.
  - The wrong comments:
    - `doc.go:327` still says a note with a missing record file "is led to the plain promote as well".
    - The `labels.go:52` comment says the plain promote "gives it recipe raddle".
    - Neither holds for this case.
  - Evidence: `labels.go:60-67`, `store.go:783-790`, `doc.go:327`.
  - Fix direction:
    - Name `--recipe raddle` in the label rule's refusal for this case. The label rule cannot see the record file.
    - Alternatively, let the label rule's refusal say "which gives it recipe raddle" only when the record is stored.

**Not a finding:** `docs/overview.md:369` and `SKILL.md` say a plain promote of a Sketch "makes it a task with recipe raddle", which holds for a Sketch with a stored record.

**Tool calls**
1. `grep -rnE` for stale or duplicate promote wording across the repo, excluding `_lym` and `.lym`. My context had only greps for `recipe raddle` and promote-near-sketch phrasing, so this checked other phrasings of the old rule.
