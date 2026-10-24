```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff c9c4e7ddf..HEAD.

**List (function/branch, case run, verdict)**

The diff changes no executable code. It touches only `PATTERN.md` and doc comments in six Go files, so there is no function, branch or error path to run with a concrete input.

- `PATTERN.md`, six new entry lines: I measured the file at 8052 bytes and its longest line at 317 characters. That is under `MaxOverviewBytes` (16384), `HeadroomBytes` (15564) and `MaxEntryLineChars` (360). Correct.
- `PATTERN.md`, `friction-capture` entry: it keeps `(test)`, which its removed line carried. Test files under `internal/frictionengine` and `internal/webstercli` exist, so the marking is plausible. I did not check whether a test enforces it. Correct as far as I read.
- `internal/friction/doc.go` new sentence: "`lyx loom start` vouches for the resume it spawns, so a deliberate resume never leaves a crash-resume note." Run against `internal/loomcli/start.go:244-261,273-278`:
  - Normal case: `vouchForSpawnedResume` writes a voucher only when `frictionDir` is non-empty, which the code comment ties to Tier 2 being on.
  - Failure case: if the status file can't be read or doesn't exist, it only warns and writes no voucher.
  - The voucher suppresses at most one entry observation, and only when the history length and state match what start recorded.
  - So "never leaves a crash-resume note" is an overstatement. The sentence sits in a paragraph that begins "With Tier 2 on", which covers the Tier 2 condition. But a failed status read, a mismatched history length or state, or a second observation would still write a note. This is a doc-precision LOW at most, and I am not raising it as a finding.
- Doc-comment ID citations:
  - `internal/hubreconcile/doc.go`, `internal/parentlock/doc.go`, `internal/shedengine/doc.go`, `internal/reedengine/doc.go` and `internal/agentname/agentname.go` each cite an ID that matches its `PATTERN.md` entry exactly.
  - `internal/friction/doc.go` cites `PATTERN-friction-capture`, which also matches exactly.
  - Correct.

**Findings:** none.

**Tool calls:** none. The manifest already held the diff, `internal/loomcli/start.go` and the `internal/pattern` caps.
