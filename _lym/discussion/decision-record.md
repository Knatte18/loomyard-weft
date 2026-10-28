# Decision record: codex-hook-spike

## Goal

Find whether Codex can take creel posts through hooks the way Claude does, so a later task can move Codex sessions off the pane path.
The deliverable is one research report, `docs/research/codex-hook-delivery-<date>/report.md`; no lym code changes.

## Scope

- Six Codex CLI runs on `gpt-6-luna` at `low` effort, one per case, no repeats: idle wake, `Stop` block, `post_tool_use` block with a reason, trust without the sentence, folder trust for a linked worktree, `Tab` queue and `codex exec`.
- Each case uses a hand-written hook script standing in for creel.
- Each run is a reed strand in this worktree; its turn end is read from a hook writing a file, never from a pane glyph.
- The report gives per case the hook config and script, whether the model took the message at its next step, later or not at all with its words quoted, a `tmux capture-pane -p` copy after delivery, and what the operator could see; it ends with a table of case by outcome and operator visibility.
- Out of scope: any change to Go code, stencils, PATTERN or the operator's `~/.codex`.

## Decisions

- Decision: the spike uses its own `CODEX_HOME` under the gitignored scratch area, seeded with a copy of the operator's `auth.json` and nothing else, removed when the spike ends.
  Rationale: case 5 needs a trust list the spike controls, and the operator's `~/.codex/config.toml` must stay untouched.
  Bound: the copy lives only in the scratch area, never in a commit, and holds credentials only for the spike's duration.
- Decision: hooks are given with `-c hooks.<Event>=[…]` plus `--dangerously-bypass-hook-trust`, except where a case tests trust itself.
  Rationale: the 2026-10-24 spike showed this is the launch path an engine would use.
- Decision: a case a run cannot answer because Codex lacks the feature is reported as "none exists" with the evidence, not worked around with the pane path.
  Rationale: case 1 asks for "a finding that none exists" as a valid outcome.
- Decision: the report is dated with the run date and placed beside the other `docs/research` reports; the change description carries the findings summary.
  Rationale: matches the board entry's report path.

## Constraints

- `PATTERN-lym-temp-root`: spike scratch lives under `.scratch/` or `.lym`, never a system temp directory.
- `PATTERN-pane-text-injection`: the spike drives panes by hand through reed strands for observation only; nothing in Go changes.
- `CLAUDE.md`: interactive tmux only, never `claude -p`; Markdown with one sentence per line; no push, no production deploy.

## Auto-mode assumptions

- The operator is absent; the entry's cases and method are followed as written, and ambiguity is settled by the narrowest reading.
- Folder trust entries Codex writes go only into the spike's own `CODEX_HOME`.

## Open risks

- One run per case on one model, so a model's next step is one sample.
- `codex exec` may not fire the same hooks or may not be drivable as a reed strand; the report records what happened either way.
- The copied `auth.json` may be refreshed by Codex during the spike; the operator's own token could then be stale on the next use, which a re-login fixes.

## Acceptance criteria

- The report exists at `docs/research/codex-hook-delivery-<date>/report.md` with a section per case holding hook config, script, outcome with quotes, pane capture and operator visibility.
- The report ends with a case by outcome table including operator visibility.
- Each case's strand is removed and the spike `CODEX_HOME` and its credentials copy are deleted.
- The operator's `~/.codex/config.toml` is unchanged.
- `lym loom validate-description` is clean.
