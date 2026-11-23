# Case brief: Code-Burler-Review never reads its input box (transient agent-start)

## 1. The question the next judge must answer

Why does the shuttle readiness check report "no input box is on screen; a menu or picker may be open" for a live Code-Burler-Review pane whose idle input box is drawn under a left-labelled top rule, so that every start fails before the prompt is delivered? Should the box detector accept a top rule of the form `─ <label> ─`, or is the session label to be shortened instead? That is a code judgment, not a re-launch.

## 2. The facts

- Stop: `kind=producer`, `transient=agent-start`, producer `Code-Burler-Review`, engine `burler-review`, round 1. Envelope: `_lym/shed/intake-checks-already-fixed/drive-reports/drive-envelope-20261123-123138-a9da.json`. Driver report: `_lym/shed/intake-checks-already-fixed/drive-reports/drive-report-20261123-123138-946d.md`.
- Driver log, `.lym/loom/driver.log` lines 9–50: round 1 starts at 12:14:41 (strand `12ad795a97f1d1886651951fc86c6e6f`), round 1b at 12:23:00 (strand `1e47a5f6e06ee3f44ca73459cd425e4e`), and the fail-stop at 12:31:38.
- Trace `.lym/shed/intake-checks-already-fixed/steps/3ccd1cd8f7bda61c.trace.log`, the same events as the driver log, lines 12–45. Each of the five launches (attempts 1, 1b and their second starts) shows the same pane tail: the skills loaded, `● ok scribe:prose …`, then the idle caret line and its rule.
- Stderr log `.lym/shed/intake-checks-already-fixed/steps/3ccd1cd8f7bda61c.stderr.log`: empty.
- The pane tail as captured in the trace (line 13 and the others), verbatim in shape:
  ```
  ✻ Cooked for 4s · done 12:14
  ─ ly:intake-checks-already-fixed:code-burler-review ─
  ❯<nbsp>
  ─────────────────────────────────────────────────────
    ⏵⏵ bypass permissions on (shift+tab to cycle) · …
  ```
- The reader that reports the message: `internal/shuttleengine/readiness.go:18` (`readinessReasonNoInputBox`), chosen in `judgeReadiness` (`readiness.go`, the case `box.reader && !box.read`).
- `box.read` comes from `Claude.InputBoxText` (`internal/shuttleengine/claudeengine/session.go:122`), which calls `inputBoxInterior` (`session.go:300`).
- `inputBoxInterior` needs a top rule above the caret that `isTopBoxRule` accepts (`session.go:342`). That accepts only a line starting with `───` (three rule glyphs), a line ending in the cut-label marker `… ─` (`session.go:333`), or a plain rule (`isBoxRule`, `session.go:355`, every non-space rune a rule glyph). The comment at `session.go:337–340` says a named session labels its top rule at the right (`──── tst:orch ─`).
- The pane's top rule is `─ ly:intake-checks-already-fixed:code-burler-review ─`: one rule glyph, then a label on the left, then one glyph. It meets none of the three accept shapes, so `top == -1`, `inputBoxInterior` returns not-ok, and the reading becomes "no input box".
- The label is the session name plus the window name, so it is the same on every launch of this role in this run.

## 3. What I ruled out, and how

- A transient provider that was not yet up: ruled out. The pane shows the skills loaded and the idle caret line on every launch, across two burler attempts and about 16 minutes (driver log 12:14 to 12:31). The provider did become ready; the detector never reads it.
- A menu or picker open over the box: ruled out by the pane tail. Its last lines are the idle caret line and its bottom rule, with no option list. The reason text is the generic one `readinessReasonNoInputBox`, which the detector returns whenever `inputBoxInterior` fails.
- A too-short pane: ruled out. The pane holds more than the four rows `boxMinRows` needs (`session.go:62`), and the reason would read as the too-short text.
- A draft or a running turn in the box: ruled out. The box is blank (`❯` followed by nothing), and the footer shows no running hint.
- A plain re-launch settling it: not expected. The top rule's label is fixed by the role and run names, so every new strand draws the same rule. Four re-launches already failed the same way (two per attempt).
- A goto to Code-Burler-Review or Code-Bouncer: does not fix it. The defect is in the detector, not in a row's output.
- I did not run any lym verb or git command that changes anything. The reads were the stop files, the driver log, the trace and the three source files above.

## 4. Hypothesis (unverified; marked as a hypothesis)

Hypothesis: `isTopBoxRule` does not accept a top rule with the label on the left (`─ <label> ─`), and the session label for this role is long enough to be drawn that way. Claude draws the labelled top rule on the left when the label fits, and on the right or cut otherwise. The detector's comment expects the right-hand form. If that is so, the fix is in `internal/shuttleengine/claudeengine/session.go` (accept a `─ <label> ─` top rule, with a test that uses this capture), not in the run's data. Not verified: I did not check Claude's own rule layout across widths, and I did not run the detector against this capture. Settling it needs a code read and a test against the captured pane.
