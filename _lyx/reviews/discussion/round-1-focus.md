---
round: 1
focus:
  - "decision-record.md D6, AskUserQuestion bullet: the notice says the steer already given by the deny reason applies, but a session sees that deny reason only after a refused call. Settle whether D6 specifies notice content the session can act on at spawn, before any refusal."
  - "decision-record.md D7: settle whether the rationale matches the path the notice actually travels. The existing forbidden-character check protects steer text inside a single-quoted echo of a JSON payload, while the notice goes through shell.Shell Quote as a command-line argument. Also settle whether the forbidden set is the right constraint for that path, including characters D6's sentences may contain, such as backticks under PowerShell."
  - "decision-record.md D6, fork-run notice and webster rationale: an appended system prompt reaches the Master and may also reach the forks it spawns. Settle whether 'only fork subagents may be spawned' is accurate for every session that receives it, and whether the stated reason for leaving out the webster guard holds, given which sessions that guard is installed for."
  - "decision-record.md D4/D5 and the byte-identical acceptance criteria: D4 says a preparation can amend the prompt, while D5 says the rotator appends to it. Settle whether the decision record pins the amendment contract (append-only, or arbitrary rewrite) tightly enough for the byte-identical and nil-preparation criteria to be testable as written."
  - "decision-record.md Notes for the plan writer, third bullet: the stale sentence in the internal/loomengine/plan.go file comment is a required doc fix, but it sits only in the optional notes. Apply the completeness-before-leanness test: settle whether it belongs in Scope and Acceptance criteria instead."
  - "decision-record.md Auto-mode assumptions, last bullet, against Open risks, third bullet: both describe the restarted run whose plan directory holds an unrelated plan. Settle whether one is redundant. Also settle whether Open risks bullets 1 and 2 describe real risks or are rationale that belongs under D6 and D2, or in support-log.md."
---

Seeded from a read of both artifacts, the discussion writer's stencil, and spot checks of `internal/shuttleengine/claudeengine/settings.go`, `internal/shedadapters/singlellm.go`, `internal/shedrecipe/entries_planwrite.go` and the `shell` quoting tests.

Checked and not promoted:
- The claim that attach matches on output files only holds: `AttachGated` collects candidates from `OutputFiles`.
- `Env.StencilsDir` already exists in `shedrecipe`, so D5's told-geometry constraint has a real route.
- The missing call-site enumeration and the absent rejected alternatives in the decision record are both on the rubric's do-not-flag list.
