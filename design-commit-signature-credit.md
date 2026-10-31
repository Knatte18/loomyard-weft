# Landing commits drop Co-Authored-By and Lym-Task, and carry an optional credit listing the engines used

`commit-signature-credit` · Task · enhancement, landing · running · Drift-Verify

## Seen (operator, 2026-10-30)

A landing commit ends with `Co-Authored-By: Claude <noreply@anthropic.com>`, from `engine_trailers.claude` in the landing config (`internal/landingshed/template.yaml`), and a Codex run would add `Co-Authored-By: Codex` the same way.
That makes each engine a co-author of the commit, which the operator does not want.
The commit's last line is `Lym-Task: <slug>`, added by `taskTrailer` in `internal/landingshed/finalize.go`; the operator sees no reason for it.

## Design (operator, 2026-10-30 and 2026-10-31)

- No engine is credited as a co-author: no `Co-Authored-By` line for Claude, Codex or any other engine.
- `Lym-Task: <slug>` is removed.
  A landing commit and its pull request carry no trace of LoomYard or of its use, bar the credit below.
- The commit ends with a credit block, written as plain lines, never as git trailers, by default:

  ```
  Created using
   - LoomYard https://github.com/Knatte18/loomyard
   - Claude <noreply@anthropic.com>
   - Codex <codex@openai.com>
  ```

- Each line is a config text of its own in the landing config: the heading, the LoomYard line, and one line per engine.
- The LoomYard line is always in the block.
- An engine's line appears only when that engine ran a session in the task, detected automatically as `engine_trailers` decides today; an engine that ran nothing in the task is not listed.
- The same block serves the pull request's body, so a commit and its pull request carry one signature.
