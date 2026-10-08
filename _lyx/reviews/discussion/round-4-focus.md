---
round: 4
focus:
  - "Hook line integrity in `_lyx/discussion/decision-record.md` D1: the Stop and ask hook commands are rewritten to stamp `lyx_at`; settle whether the stamped line parses to the same event as the unstamped one for a payload larger than one shell buffer and one containing `%`, `\\`, `$` and an embedded `\\n` escape, given that `ParseEvents` in `internal/shuttleengine/claudeengine/events.go` skips malformed lines silently and the wait loop classifies by the last parsed event; check that D1's rationale makes no claim about write atomicity the hook's shell does not provide, and that the acceptance criteria cover the case."
  - "Output-file facts in D3: the output-file list is stat'ed once at reading time and folded over the signals in file order; settle whether the fact applies to the newest turn end only or to every turn end, whether `idle-done` then has exactly one `since` stamp for a files-only reader that keeps nothing between calls, and whether the claim that the signal-derived history is the same for every reader holds for entries resting on an output-file check."
  - "State precedence in D3 and D4: a run whose process is gone, with no registry entry, whose newest signal is a turn end with every output present, or a session end; settle which state and cause the reducer reports and whether the Decisions and acceptance criteria name a single outcome."
  - "Reducer inputs in D3: every fact the reducer's states and `unknown` definition read (including the prompt offset) is named with its source, and the orch watcher, which has no run record, can supply it."
  - "Consistency of D1 and D6 wording with D3: the stated reason `lyx_at` is advisory covers what the bound actually protects, the history's ordering and `since` monotonicity are stated, and D6's description of what the newest history entry shows matches D3's rule that an entry is recorded only when state or cause changes."
---

Focus targets the open question areas of the latest discussion draft.
Each entry names a question to settle and restates the site it depends on.
