---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:74
    summary: D4 adds lyx loom card-pack to the sandbox rule's allowed list through existing markers, but that list is static text in webster-body-implementer.md shared by every layout and the recovery prompt, so the byte-unchanged decision and its acceptance criteria cannot both hold
    fix: In D4, replace the sentence at line 74 with a decision that the sandbox paragraph of webster-body-implementer.md gains one new optional marker after `lyx quarry resolve` in the always-allowed list, filled with `, lyx loom card-pack` (exact punctuation pinned in the plan) only under the per-card layout and empty in the file, inline, pack-off and recovery prompts, so their bytes stay unchanged; change line 66 from fills the body's existing markers to fills the body's existing markers plus that one new optional marker; add to D7 that the `card` marker row matches only text the per-card fragments carry, so a file-layout prompt never matches it.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:129
    summary: D7 rests on tools/forkaudit existing, but glyph-audit-tool has not landed (its board status is running, Plan-Burler-Review, and main has no tools/forkaudit), this entry's depends_on is empty, and the record gives the plan writer no step when the tool is absent at Plan-Write
    fix: At line 17, replace taken from the landed `glyph-audit-tool` tool at Merge-In with taken from task `glyph-audit-tool` once it has landed on the parent and been taken in at Merge-In. In the Open risks bullet at line 129, name the hold as this entry's board `depends_on` entry naming `glyph-audit-tool`, set by the orch, and add that a Plan-Write which finds no `tools/forkaudit/` on the parent stops and reports to the parent rather than planning D7 against a missing tool; until then the per-card forks still classify in forkaudit as `file` with a trace disagreement, which the bullet states.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:57
    summary: D3's Bound says card-pack writes no file, but D6 has every call log an Info line, which internal/logger appends to the worktree's trace file under its LogsDir
    fix: Rewrite the text after Bound on line 57 to say it writes nothing but its one `webster card pack` Info trace line to the worktree's log, mutates no webster or plan state and holds no lock, so it adds no route around any gate.
  - id: F4
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md:130
    summary: The Open risks bullet on the `## Start here` confound is measurement deliberation the plan writer cannot act on, and D4's rationale already carries the reason the section stays
    fix: Delete the Open risks bullet at line 130 and append its sentence to the support log's Turn 3 Q9 bullet; it carries no requirement or constraint, so nothing moves into Decisions or Constraints.
---

### [BLOCKING] The sandbox-list addition contradicts the unchanged-bytes decision

**Location:** `_lyx/discussion/decision-record.md:74`

**Issue:** The sandbox rule, including the sentence listing `lyx loom lint-comments`, `lyx loom lint-pattern` and `lyx quarry resolve` as always allowed, is literal text in `contracts/stencils/webster/webster-body-implementer.md`, not a marker.
`composeForkTemplate` and `composeRecoveryTemplate` (`internal/websterengine/render.go`) both join that same body.
D4 says the per-card layout fills only the body's existing markers (`fresh_read`, `card_read`, `locate`, `kickstart_pack`), none of which sits in the sandbox paragraph.
So the only way to add `lyx loom card-pack` to that list under D4 as written is to edit the shared body text, which changes the file, inline, pack-off and recovery prompts too, against D4's line 75 and the acceptance criteria for `kickstart_pack_per_card: false` and for the recovery prompt.
The plan writer has no stated way to satisfy both.
It also interacts with D7: if `lyx loom card-pack` reached every prompt through the shared body, a `card` marker row matching `lyx loom card-pack` would class every file-layout fork as `card`, merging the arms D7 exists to keep apart.

### [MEDIUM] D7's dependency on an unlanded tool has no actionable fallback

**Location:** `_lyx/discussion/decision-record.md:129`

**Issue:** Scope line 17 calls `glyph-audit-tool` landed; `lyx board get` shows it running at Plan-Burler-Review, and `git ls-tree main tools/` has no `forkaudit`.
The Open risks bullet says the parent holds this run before Plan-Write, but this entry's `depends_on` is empty, so nothing in the board or the shed enforces that hold, and the record does not tell the plan writer what to do when it reaches Plan-Write without the tool.
The sibling record's own Open risks bullet confirms the cost of a miss: its marker table classes any prompt with a `## Start here` heading as `file`, so until D7's row exists every per-card fork lands in the file arm with a trace disagreement, not in a separate `trace:card` arm.
D7's marker order and span source are otherwise consistent with the sibling's D3 table and with this record's acceptance criterion.

### [LOW] D3's Bound omits the trace-line write

**Location:** `_lyx/discussion/decision-record.md:57`

**Issue:** `internal/logger` opens its sink with `os.OpenFile(..., O_APPEND|O_CREATE|O_WRONLY)` under the worktree's `LogsDir`, so the D6 Info line is a file write.
The Bound's writes no file is therefore false as stated; the correct bound is that the only write is that append.

### [NIT] Measurement-confound note sits in Open risks

**Location:** `_lyx/discussion/decision-record.md:130`

**Issue:** The bullet records why a comparison between arms is not clean, which is deliberation for whoever reads the measurement, not a risk the plan writer can mitigate in code.
D4's rationale already states why the `## Start here` section stays, so the plan writer loses nothing when the bullet moves to the support log.

## Focus verdicts

- D3 and D6 bound: apart from F3, not a defect.
  A fork can already Read every card file under `_lyx/plan/`, done or in another batch, and run `lyx quarry resolve`, both allowed today; `card-pack` composes those two reads and exposes nothing `steerWebsterForkDeny` withheld.
  The claudeengine settings carry no deny on `lyx loom`, so the verb is reachable from a fork as D3 claims.
- D1 default and mid-Webster runs: not a defect.
  The mixed-layout risk is stated, the begin-batch trace line carries `batch`, and forkaudit classifies per fork from its prompt, so the plan writer has nothing further to build for it.
- D1 precedence against D5 and the criteria: consistent.
  `buildKickstartPack` already decides off, glyph-less language and empty before any layout, and the per-card choice only replaces the `KickstartPackFile` branch; every combination of the three keys has a stated outcome and a criterion.
- D4 fragment texts: firm enough to build and test, except the sandbox list (F1).
  The other unchanged-bytes claims hold as long as `renderLocate` and `renderReadRules` keep returning today's fragment names for the existing layouts, which a layout argument replacing the two bools allows.
- D7: see F2.
- Relocation: F4 is the one move; no requirement sits only in the support log.
