---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: contracts/stencils/burler/burler-fork-reading.md:21-24
    summary: the waves name no call for the failure, restart and resume paths, callees or other-process paths that the checklist requires, so a literal reader reads them late or serially
    fix: In wave 1 of `burler-fork-reading.md` (line 22), add to the list of parallel calls one search per changed symbol for its callees and the paths that reach it from another process, and one search per change for its failure, restart and resume paths. Keep wave 3 for the hops those results open.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/burler/burler-fork-reading.md:22
    summary: wave 1 says to read each file the target names and drops the checklist's at the definitions of the symbols it cites, which can be read as licensing whole-file reads against the stencil's own not-whole-files rule
    fix: In wave 1 of `burler-fork-reading.md` (line 22), change "the read of each file the target names" to "the read of each file the target names, at the definitions of the symbols it cites".
---

### [LOW] Waves leave part of the checklist without a named call

**Location:** `contracts/stencils/burler/burler-fork-reading.md:21-24`, against the checklist at lines 11-17.
**Issue:** The checklist asks for callees, for every path that reaches a changed symbol from another process, and for the failure, restart and resume paths of each change.
Wave 1 names callers, tests, docs, fixtures, writers, readers, the board entry and the `pattern/` files.
Wave 3 mentions a spawned process or a restart path only as a hop the second wave opened.
A handler following the waves literally issues no first-wave call for callees, other-process paths or failure and resume paths, and reaches them late or one a message, which is the serial gather this change exists to remove.
The commit's own claim is that the waves gather the checklist, so a checklist item no wave names is a gap in that claim.

### [LOW] Wave 1 drops the at-definitions limit on file reads

**Location:** `contracts/stencils/burler/burler-fork-reading.md:22`, against lines 11 and 28.
**Issue:** The checklist reads "every file the target names, at the definitions of the symbols it cites", and a closing rule says to read definitions and call sites, not whole files, because a fork re-reads its inherited context on every turn.
Wave 1 says "the read of each file the target names" with no such limit, and it comes first in the order the handler follows.
Read alone it licenses whole-file reads, which weakens an existing rule of the stencil.
The closing rule still stands, so the weakening is mild.

## Rejected

- doc.go loose tally (`internal/burlerengine/doc.go:286`, "a few waves"): rejected, the sentence matches the stencil and "a few" pins no count that a stencil edit would falsify.
- Commit message figures: rejected as a finding, I recomputed them from `calls.tsv` and they hold. Calls before the first spawn run to 31, 60 and 64 turns at 1.10, 1.03 and 1.08 calls a turn, inside the stated 1.03 to 1.10. The Sonnet range in the message agrees with the report's per-round figures.
- Scope against the board entry's Idea (one instruction change, measured with the audit scripts): rejected, the diff makes one stencil change plus a one-sentence doc note and nothing beyond it. Measuring the change is a later step of the entry and lies outside a diff.
- Test economy: rejected, the added stencil prose is static and carries no assertion. The tests that touch this stencil, `contracts/stencils/markers_test.go:29` and `internal/burlerengine/template_test.go:76,184`, pin the heading, the marker placement and the marker set, none of which the diff changes.
- Missing doc or PATTERN update: rejected, `PATTERN-stencil-ownership` holds, since the prose sits in the stencil read at call time, and the directive declares no PATTERN change.
