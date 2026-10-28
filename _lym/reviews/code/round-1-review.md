---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/creelengine/chase.go:235
    summary: noticeUnansweredLocked calls delivery.Strands() while the store mutex is held, so a slow strand listing blocks every store verb
    fix: In stepChases and releaseGoneHolder read the hub's strands before entering s.do (as readStrands does elsewhere) and pass them into settleUnansweredLocked and noticeUnansweredLocked instead of calling s.delivery.Strands() inside the lock.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/burlercli/wiring.go:152
    summary: the three notifier wirings (burlercli, loomcli, webstercli) switched to NotifyAsker with no test reaching SetNotifier, so reverting one to NotifyParent fails no test
    fix: Add one row per runner wiring (or one shared wiring test) that builds the runner's notifier for a held agent in a beam and asserts it posts to the strand orchroute.Asker resolves, in internal/burlercli, internal/loomcli and internal/webstercli tests.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/creelengine/doc.go:290
    summary: a live doc comment names the removed shuttle send verb, a history clause D8 asks to be dropped or re-pointed at creel
    fix: Reword the sentence to "A line injected under a pause is held with its TTL frozen and typed after the resume; it is typed only at idle within its TTL." and drop the clause naming the removed verb.
---

### [LOW] Strand listing under the store mutex

**Location:** internal/creelengine/chase.go:235
**Issue:** `noticeUnansweredLocked` calls `s.delivery.Strands()` inside `s.do`, reached from `stepChases` and `releaseGoneHolder`. Every other `Strands()` call in the store and the sweep runs outside the lock. A slow tmux or reed listing stalls every store verb for its duration.

### [LOW] Notifier wirings have no covering test

**Location:** internal/burlercli/wiring.go:152, internal/loomcli/wiring.go:464, internal/webstercli/wiring.go:211
**Issue:** D3 requires every notifier wiring to resolve with the asker rule. The walk itself is pinned in `internal/orchroute` and `internal/orchcli`, but no test reaches `SetNotifier` in the three runner packages. The change is a one-line call swap, so I rate it LOW.

### [NIT] Removed verb named in a live doc

**Location:** internal/creelengine/doc.go:290
**Issue:** "where the removed shuttle send refused it" names the removed verb and records history; D8 asks docs to name the creel verb or nothing.

## Rejected

- Fork code-tests F2 (trust sentence in `claudeengine/settings.go` has no test): static prose, exempt under the rubric's test-economy rule.
- Fork code-tests F3 (carried clock fields not asserted): `internal/creelengine/journal_test.go:630-667` asserts `ChaseConsumed`, `ChaseMark` and `Chases` after a carry.
- Fork code-correctness bookkeeping note (`chaseWritten` and `chaseBase` never pruned): a per-message map entry bounded by retention and daemon lifetime; not worth a finding.

My own holistic read against the decision record found D1 to D13 and the amendments carried out: the chase clock, the two-chase limit, the `holder-gone` sweep widening, the content-free chase and the former-holder guard match D10 and its amendments. The declared `PATTERN-agent-messaging` entry is in `PATTERN.md` with the declared text.
