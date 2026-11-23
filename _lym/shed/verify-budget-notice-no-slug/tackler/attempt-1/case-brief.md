# Case brief: Code-Burler-Review seat never becomes ready (run verify-budget-notice-no-slug)

## 1. The question the next judge must answer

Why does the readiness check read "no input box is on screen" for every launch of the `code-burler-review` seat, and is the cause a rendering fact of the pane (so the check must accept it) or an artifact of how the pane was captured? Decide whether `internal/shuttleengine/claudeengine/session.go` `isTopBoxRule` must accept a labelled top rule whose label starts at the left, and whether that change is in scope for this run.

## 2. The facts, each with a pointer

- Stop envelope: `_lym/shed/verify-budget-notice-no-slug/drive-reports/drive-envelope-20261123-124255-c162.json` records `kind: producer`, `transient: agent-start`, producer `Code-Burler-Review`, `restep_hold: false`, `state: failed`.
- Driver report: `_lym/shed/verify-budget-notice-no-slug/drive-reports/drive-report-20261123-124255-7aa6.md` lines 5 and 16, the failure after two consecutive died outcomes (attempt 1 and 1b).
- Step trace `.lym/shed/verify-budget-notice-no-slug/steps/6a312f3086395dfd.trace.log` lines 13, 16 and 22: the warnings that the segment colour could not be typed because "no input box is on screen; a menu or picker may be open". The same text appears on all four launches: runs `f5c7a1c851125ff115a5047e47fe430d` (attempts 1 and 2), `c7790ea1659a40883db5ba68f91c2898`, `9708f9ce45b3afec192342eee737ea9f`, `950175ba133702bbea14e7330f5e28bf`. Each launch sends "the start prompt was held busy" twice, then tears the strand down.
- Trace lines 22 and 27, the pane's last lines as captured. The top-rule line reads ` ly:verify-budget-notice-no-slug:code-burler-review ─` (no leading `───` run, no `… ─` cut). It is followed by `❯ `, then a plain `─────` bottom rule. Those are the lines the box reader sees.
- `internal/shuttleengine/readiness.go:121-122`: a pane whose box is not read gives `readinessReasonNoInputBox`, which is the held reason the trace shows.
- `internal/shuttleengine/claudeengine/session.go` `inputBoxInterior` (the caret `❯` is at `internal/shuttleengine/claudeengine/startup.go:181`): the box is bounded by the nearest rule above and below the caret. `isTopBoxRule` accepts a top rule only if it starts with `───` and ends with `─`, or ends with `… ─` (`cutLabelRuleEnd`), or is a plain box rule. The captured top-rule line matches none of these, so `top == -1`, `ok` is false, and the box reads as absent.
- The session file's comment at `session.go` (the `isTopBoxRule` doc) says a named session labels its top rule at the right, as `──── tst:orch ─`. The captured line is not of that shape.
- Missing artifacts: the per-run `startup-capture.txt` files named in the log do not exist on disk (the run dirs `f5c7…`, `c779…`, `9708…` are gone; `950175…` remains only as a manifests dir). `.lym/loom/driver.log` is empty. The child traces under `.lym/logs` were not read in full; the one for this step is `trace-20261123T112534Z-6a312f3086395dfd-2793244.log`.

## 3. What you ruled out, and how

- A menu or picker really being open: the captured pane shows no menu, only the prompt, the status line and the box rules.
- The draft or non-breaking-space path: the caret line `❯ ` is blank-ish, and `InputBoxText` trims it. The failure is upstream of that, in finding the box at all.
- A transient session start that a relaunch clears: four launches over about 13 minutes, across two round attempts (1 and 1b), each reproduced the identical capture and the identical held reason. A plain re-launch has already failed four times.
- The git side: this stop is not about a commit or branch being absent from a remote, so the `ls-remote` rule does not apply.
- Not verified: whether the logged top-rule line has lost its leading rule glyphs in logging, or whether the pane really drew it that way. The raw capture is gone, so this cannot be settled from the files here.

## 4. Hypothesis (a hypothesis, not a finding)

Hypothesis: the seat's pane draws its top rule with the session label at the left and no leading `───` run, because the label is long (`ly:verify-budget-notice-no-slug:code-burler-review`). `isTopBoxRule` only recognises right-labelled and cut-labelled rules, so `inputBoxInterior` finds no top rule and the box is read as absent on every launch. This would make it a code gap in the readiness parser, not a transient. The alternative is that the log writer trimmed the leading run, in which case the readiness parser is fine and the seat's pane needs a fresh capture to show it. The next judge should settle which by reading a raw styled pane capture for this seat, then decide whether the parser fix is in scope for this run.
