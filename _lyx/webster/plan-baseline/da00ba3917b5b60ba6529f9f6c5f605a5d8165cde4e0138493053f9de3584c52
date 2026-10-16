# Card 14 — creel-sandbox-scenarios

**Prosa:**
- `//tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`

**Uses:**
- `//docs/research/creel-audit-2026-10-15/report.md`

**Intent:** Decision D9: the audit's proposed creel scenarios C1 to C13 become re-runnable scenarios of the shuttle sandbox suite, after S9.

Each of `### C1 -- …` to `### C13 -- …` goes after S9 in the suite's own shape: a `**Covers:** creel` line, a **Goal** naming the task rather than the commands, a **Watch** taken from the report's "Proposed creel sandbox suite" entry, and the **Verdict** the report gives (FAIL on …, or WARN for C3's timing drift and C11's unshown bounce), ids kept as `C1` to `C13`.
Where this task changed the behavior a scenario watches, the Watch says what the build now does:
C4's `read > /dev/full` exits non-zero and leaves the mail unread;
C7's interrupted prompt is cleared from the box, so the inject lands with no hand-clearing, and a box holding the operator's text is left alone;
C10's malformed middle line gives the verb an error naming the daemon log, which quotes the start's error line;
C11's reader sees `ended_unread_by` and `ended_unread_at` on the bounced message;
and C13's verdict follows D1 and D2: a plain `lyx reed add --cmd claude` strand receives its reminder, and a stacked 3-row strand's stall reason shows as a `stalled` event in `lyx creel log` and as a progress line on stderr of `lyx creel inject --wait`, FAIL on a silent stall.

A setup paragraph before C1 describes the fixtures the audit used, as steps the agent takes in the Hub: bash strands added with `lyx reed add --cmd bash`; a fake idle pane that swallows input (a script that draws a ready marker and an empty input box and reads lines without echoing them); a fake input box that ignores Enter; and an orch-role copy of the idle pane, added with `--role orch`.
It names the bound the audit kept: the agent stops the daemon by replacing its binary, never by killing a process it did not start.

A "Controlled creel exceptions" subsection under the Black-box rule names the non-black-box steps the scenarios need and nothing more: listing the tmux and lyxtemp socket directories read-only (C1); planting or editing a journal day file while the daemon is stopped (C10); and running a process detached from a strand's pane (C12).
C12's disagreeing-strand-name check is operator-assisted when the agent's harness refuses a command naming the strand variable: the agent asks the operator to run it in the pane and reports what the operator saw.

The suite's ref range in Capturing findings (`S1`-`S3`) becomes the S and C ids the suite holds, the session-log template gains one `C1` to `C13` line each, and the Notes say creel scenarios go here as `C` scenarios beside the shuttle `S` ones.
The suite's own tests read only its embedded body and the `**Covers:**` lines, so they stay green.
