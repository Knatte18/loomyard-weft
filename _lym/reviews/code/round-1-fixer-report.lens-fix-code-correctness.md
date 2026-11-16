```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the fix's changed code with concrete inputs through the `fix-code-correctness` lens. I found one LOW and no wrong results.

**Findings**

- **LOW**, `tools/retro/retro.sh`, header line "The transcript layout is Claude Code's: a run on another agent engine has no such directory and is listed in missing.jsonl."
  - The text after `# ` is about 123 characters, over the 120-character limit that the prose skill sets for file comments.
  - Split it at the colon or into two sentences.

**Traces that held**

- **F6 cap tie (`Select`):**
  - Landings x08, y10, z10, w11 with a cap of 2 keep [x, y]. The first left out, z, landed at 10:00, which equals y, so y is dropped. The plan is [x] with until 08:00, and the next retro starts after 08:00 and covers y, z and w.
  - With the loop removed the plan ends at 10:00 and the new row fails.
  - A cap of 1 with a tie keeps one run and still skips the tied run. The `len(runs) > 1` guard allows this and the review asked for it.
- **F8 named `From`:** the plan starts at the oldest landing minus one second. The `TestSelect` row expects 07:59:59, and the CLI test expects the same value in the envelope's `from`. Both fail if the `Add` is removed.
- **F12 lookback message:** now 14:00 on 11-16 minus 60 days is 2026-09-17T14:00:00Z, which matches the row's expected strings.
- **F2 future `--until`:**
  - `parseWindow` runs before cwd resolution, so the `2999-01-01` row reaches it.
  - `--until` set to today parses to midnight, which is not after now, so today's date is accepted.
  - Named slugs with `--until` are refused earlier, so the new check never meets them.
- **F7:** the concatenated format string has one `%s` and no stray `%`.
- **F1 and F14 (`retro.sh`):** the variables are declared before use and `set -u` is satisfied. The `missing.jsonl` expectation matches the new `what` text.
- **F5, F10, F11, F13 fixture:**
  - The call at index 5 gets id `tf`, which matches the notification's `<tool-use-id>`. The regex captures exit code 1, and the call's result opens with "Command was moved…". Without the notification path `GateFails` would be 1, not 2, and the test fails.
  - The median trace holds: turns [1, 2, 6] give median 2 and a threshold of 4, and costs [0.5, 0.5, 1.5] give median 0.5 and a threshold of 1. Only s1 is an outlier, rendered as `run-a s1 (6 turns, $1.5)`.
  - The Runs row works out to 4 sessions, 1 fork, $2.5 and 1 gate-fixer session.
  - Session order in `sessions.jsonl` is s1, agent-f1, s2, s3, s4, as the test expects.
  - `usage.jsonl` and `facts.jsonl` rows follow the order and key order the scripts print.
  - The shared `edit` map is mutated by `transcriptLines` but marshalled inside each loop iteration, so no turn gets a wrong id.
- **F3 and F9 (`TestRetro_FilesAndReleasesTheTaskOfALandedRun`):**
  - Each step fails with its defect in place:
    - Without the board-name merge, the bare retro would find a run in the default 7-day window and file it.
    - Without the report-directory merge, the refusal would name the landing time instead of 2999.
    - If `--since` and `--until` were swapped, the refusal text would differ.
  - The test's tier-1 gate passed with the `research` type added to the Bolt `board.yaml`.
- **F4:** the expected title matches the format string in `TaskFields`.

**Tool calls:** none. My context already held every changed file and the gate result.
