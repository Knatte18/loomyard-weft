---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: docs/research/creel-hook-delivery-2026-10-25/report.md:97
    summary: The Codex acceptance criterion is unmet because the report never states whether PostToolUse fires or whether the model sees hook output from PreToolUse or PostToolUse
    fix: Leave the report as written and defer to the run's parent, since the missing result needs a live Codex run the fixer cannot make. The parent either has the Codex run done after the usage limit lifts and the Codex section filled in, or records with lyx loom decision add that the criterion is amended to a stated not-tested outcome.
    origin: handler
  - id: F2
    severity: LOW
    class: consistency
    location: docs/research/creel-hook-delivery-2026-10-25/report.md:35
    summary: Several reported figures rest on logs that are not committed, while the Method says only that raw logs of two runs sit beside the report
    fix: In the Method bullet at line 35, add a sentence saying that the m1 to m3 table, the 146-message race counts, the cost table and the idle 19 ms and 3.0 s figures come from runs whose logs are not committed, and that only the 30-message and SendMessage figures can be re-derived from the two TSV files.
    origin: handler
  - id: F3
    severity: LOW
    class: consistency
    location: docs/research/creel-hook-delivery-2026-10-25/report.md:51
    summary: The stated median post-to-claim delay of 4.1 s is the upper of the two middle values; the committed log gives a median of about 3.9 s
    fix: In report.md line 51 change the median from 4.1 s to 3.9 s, the mean of the 15th and 16th sorted delays (3.748 s and 4.118 s) in claude-mid-turn-30.tsv.
    origin: handler
  - id: F4
    severity: LOW
    class: consistency
    location: docs/research/creel-hook-delivery-2026-10-25/report.md:121
    summary: The Recommendation says under 1 ms per call with no mail, but the Hook cost table measures 1.0 to 1.2 ms absolute and only about 0.6 ms over a bare process start
    fix: In report.md line 121 replace under 1 ms per call when there is no mail with about 0.6 ms over a bare process start when there is no mail, matching the Summary table row and the Hook cost table.
    origin: handler
  - id: F5
    severity: NIT
    class: consistency
    location: docs/research/creel-hook-delivery-2026-10-25/report.md:92
    summary: The continuation line starting It also tried is not indented under its list item, unlike the sibling continuation lines
    fix: In report.md line 92 indent the line by two spaces so it matches line 93 and stays inside the list item that starts on line 91.
    origin: lens:code-pattern-fit
  - id: F6
    severity: NIT
    class: consistency
    location: docs/research/creel-hook-delivery-2026-10-25/report.md:83
    summary: Several lines hold two independent clauses and are not broken at the clause boundary that scribe prose asks for
    fix: In report.md lines 61, 83, 100, 101, 115, 122, 126 and 130, break each line at the comma or semicolon that joins two independent clauses so each clause starts its own line.
    origin: lens:code-pattern-fit
---

### [MEDIUM] Codex acceptance criterion not met

**Location:** `docs/research/creel-hook-delivery-2026-10-25/report.md:97`
**Issue:** The amended entry requires the report to state whether `PostToolUse` fires and whether the model sees hook output from `PreToolUse` or `PostToolUse`.
The Codex section says not tested live because of a usage limit; the binary strings show the event exists, which says nothing about firing or model visibility, and the one live attempt (a `!` shell command) fired neither hook.
The gap is stated honestly: the Summary row says not tested, and the Recommendation says to decide after the live run, so no claim about Codex goes beyond what the runs show.
The deliverable still misses a listed acceptance criterion, and only the parent can decide between a rerun and an amended criterion.

### [LOW] Figures resting on uncommitted logs are not marked

**Location:** `docs/research/creel-hook-delivery-2026-10-25/report.md:35`
**Issue:** Checked against the committed TSVs, the 30-message post-to-claim minimum (0.4 s) and maximum (6.9 s), the SendMessage 31.7 s and 1.9 s, and the 1.8 s gap all hold.
The m1/m2/m3 table, the 146-message race (130, 16, 98), the cost table, the idle 19 ms and 3.0 s, and the 25 ms claim latency have no committed log behind them.
A reader cannot tell which figures they can re-derive.

### [LOW] Median stated as the upper middle value

**Location:** `docs/research/creel-hook-delivery-2026-10-25/report.md:51`
**Issue:** With 30 delays the median is the mean of the 15th and 16th values, 3.748 s and 4.118 s, about 3.9 s.
The report states 4.1 s.

### [LOW] Recommendation overstates the no-mail hook cost

**Location:** `docs/research/creel-hook-delivery-2026-10-25/report.md:121`
**Issue:** The Hook cost table measures the no-mail hook at 1.0 to 1.2 ms against 0.53 ms for a bare process start, so the cost over a bare start is about 0.6 ms.
The Recommendation says under 1 ms per call, which the absolute figure does not support.

### [NIT] Unindented continuation line in the Receiver treatment list

**Location:** `docs/research/creel-hook-delivery-2026-10-25/report.md:92`
**Issue:** Line 92 sits at column 0 between list lines 91 and 93, which are indented; it renders as a lazy continuation but breaks the list structure.

### [NIT] Lines with two independent clauses

**Location:** `docs/research/creel-hook-delivery-2026-10-25/report.md:83`
**Issue:** Each listed line is one sentence, so the CLAUDE.md one-sentence rule holds, but the scribe prose rule also breaks at an independent-clause boundary.

## Rejected

- Scope against the acceptance list: the diff changes only the report and two TSV files under the allowed directory, so no scope finding.
- The `~/.claude.json` and `~/.codex/config.toml` side effects: the report discloses both and they are not hooks, so the CLAUDE.md hook rule is not broken.
- The issue number and the 16:45 local-time claim: not checkable from the diff and not required by the entry, so no finding.
- Test economy, affected set and the code-correctness lens: the diff holds no code, tests or stencils, and the three forks reported none.
