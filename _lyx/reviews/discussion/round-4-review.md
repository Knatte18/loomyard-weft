---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:81
    summary: The notifier is a Runner seam, but the record words the receiving set per spawner; the loom wiring's one runner also carries friction reflection, landing merge-resolve and the driver, so "any other autonomous spawner holds silently" is false for them
  - id: F2
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:14
    summary: Scope says shuttle notifies the parent once, while D5 and the acceptance criteria notify once per held turn end
---

### [MEDIUM] The notifier's receiving set is worded per spawner, but the seam is per runner

**Location:** `_lyx/discussion/decision-record.md` D5 (wiring sentence, line 81) and Auto-mode assumptions (line 151).

**Issue:**
D5 puts the notifier on `shuttleengine.Runner`, and the Auto-mode assumption says "any other autonomous spawner wires no notifier and holds silently".
In the code, `internal/loomcli/wiring.go` builds one `shuttleengine.NewRunner` (line 425) and hands it to burler, webster's master starter, `reflectionShuttle` (friction reflection), the landing `mergeresolve` `Shuttle` field and the driver starter.
Setting the seam on that runner therefore also notifies for friction-reflection and merge-resolve agents, which the assumption says stay silent.
Only runners built elsewhere (`shuttlecli`, `orchcli`) are silent by construction.
Also, "batten-driven" is named as a wiring site, but `battencli` builds no runner; its runs go through the loom, webster and burler wiring.
Both places use the same wording, so the focus question (same wording in both) passes, but the wording describes a set the seam cannot produce.
The plan writer would have to pick between notifying everything on the loom runner and gating per `Spec`, with the record claiming the second.

**Fix:**
State the receiving set by runner, not by spawner: every autonomous run on a runner whose wiring sets the notifier notifies the prime's orch, which is the runner the loom, webster and burler wiring builds, and with it the friction-reflection and merge-resolve runs that share it; runners built by `lyx shuttle run` and `orch start` wire none.
Drop "batten-driven" as a wiring site, or say it reaches the notifier through those verbs' runners.
Make the Auto-mode assumption and D5's wiring sentence say the same thing.

### [NIT] Scope says "once", D5 says once per held turn end

**Location:** `_lyx/discussion/decision-record.md` Scope, third In bullet (line 14).

**Issue:**
"shuttle notifies the parent once and keeps waiting" reads as once per run; D5 and the acceptance criteria notify once per held turn end, and a second held turn end notifies again.

**Fix:**
Say "notifies the parent once per held turn end".

## Focus directive outcomes

- Label channel (D5): bounded.
  D5 and the acceptance criteria replace control characters (newlines included), cut to 200 characters counted as characters, cap the named tasks at five plus a count, and apply all three to the task labels as well as the quote.
  The criterion for a multi-line shell command pins the single-line notice.
  Not a defect.
- Delimiter and position (D5): bounded.
  The quote is the last part, the way-forward text precedes every agent-written part, a delimiter character in agent text is replaced, and the fixed prefix labels the parts as the agent's words.
  The acceptance criterion requires each part inside the delimiters with no delimiter character.
  The glyph choice is an implementation detail for the plan writer.
  Not a defect.
- D4's Bound deadline source: true for every caller.
  `Spec.Timeout` is set by the caller (config key, flag, recipe row, profile, or a remaining budget in friction reflection) and a zero value falls to `run_timeout_min` in `spec.go` and `attach.go`; "however that caller sets it" covers all of them.
  Not a defect.
- Spawner set wording: identical in both places, but the set itself does not match the runner wiring (F1).
