# Landing commits credit LoomYard and list the engines used, with no co-author

`commit-signature-credit` · Note · enhancement, landing

## Seen (operator, 2026-10-30)

A landing commit ends with `Co-Authored-By: Claude <noreply@anthropic.com>`, from `engine_trailers.claude` in the landing config (`internal/landingshed/template.yaml`), and a Codex run would add `Co-Authored-By: Codex` the same way.
That makes each engine a co-author of the commit, which the operator does not want.
The commit's last line is `Lym-Task: <slug>`, added by `taskTrailer` in `internal/landingshed/finalize.go`.

## Design (operator, 2026-10-30)

- No engine is credited as a co-author: no `Co-Authored-By` line for Claude, Codex or any other engine.
- The commit ends with a credit line to LoomYard followed by the LLM services the task actually used, for instance `Created using LoomYard (<domain>), running Claude (<domain>), Codex (<domain>)`.
- An engine is listed only when it ran a session in the task, as `engine_trailers` already decides.
- The wording and the domains are configurable in the landing config; the default carries the credit to LoomYard.
- The same line serves the pull request's body, so a commit and its pull request carry one signature.

## Open

- The domain written after LoomYard and after each engine.
- Whether `Lym-Task: <slug>` stays as the last line, and whether the signature is a plain line or a git trailer: a plain line is never read as a co-author.
