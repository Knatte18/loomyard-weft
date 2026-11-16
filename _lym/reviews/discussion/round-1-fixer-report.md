# Fixer report — discussion round 1

F8 is a `decision` finding held by the parent; no ruling reached this round, so it is left out.

## Changes

- F1, fixed as proposed: D12 hands the nudge over as `additionalContext` with a `systemMessage`, never a block, joining the one `hooks.PostToolUse` setting beside the activity and creel mail entries.
  The Rationale cites `codexengine/doc.go` on a block reading as a failed call, and the Facts cite both creel reports.
  The D12 criterion says the nudge arrives without the call being marked failed.
- F2, fixed as proposed: D12 maps each Claude breach to Codex's tools.
  A lookup is a shell call passing Claude's read-only lookup check, which moves to a shared package under `internal/shuttleengine`.
  A sliced read is a shell line-range read judged by `slice_lines` and the read-again rule.
  An edit is an `apply_patch` call.
  The matcher is `*`.
  The criterion names those rules and their thresholds, and Constraints name the shared package.
- F3, fixed differently: the review proposed settling the question tool's payload name from the binary or a spike run.
  The binary does not settle it: its strings hold `request_user_input`, but not the name a `PreToolUse` payload uses.
  A spike run means a live Codex session in a pane, which this fixer round cannot start.
  Instead, the deny takes the `*` matcher and denies any payload tool name ending in `request_user_input`, so a prefixed name like the spike's `collaborationspawn_agent` still matches.
  The Facts record `codex features list` (0.164.0) showing `default_mode_request_user_input` under development and off.
  They also say that no run has recorded the payload name.
- F4, fixed as proposed: D1's test covers every seat in `seatengine`'s tables and every other producer of a launch spec, through a producer table.
  A scan fails on a production launch-spec literal in a package the table lacks.
  The Goal stays as written, and the first criterion and the support log's Turn 3 line and Q1 match.
- F5, fixed as proposed: D4 marks both `lym shuttle run` forms hand-written, and its scan also fails on a hand-written mark outside `internal/shuttlecli`.
  The criterion covers both.
- F6, fixed as proposed: D10 runs `Prepare` into a temporary run directory it removes, with production-code stand-ins for the hook runner and sandbox probe, and its bound is narrowed to match.
- F7, fixed as proposed, by its second option: D6 and D9 are dropped from the first criterion, and their own criteria hold their settings.
- F9, fixed as proposed: D8's bound gives the timeout's source and the hook's stdin input (`source`, `cwd`, the run id as `session_id`, `CLAUDE_PLUGIN_ROOT`).
  It also says stdout is parsed as Claude Code parses it.
  The self-check changed the timeout's source, as recorded below.
- F10, fixed as proposed: D5 bounds the `developer_instructions` argument at Linux's 128 KiB cap.
  Past that cap, `Prepare` refuses the launch, naming the size, the largest item and the way forward.
  The support log's Turn 3 line and its leanness entry no longer call the risk settled.
- F11, fixed as proposed: D3's bound says an edited engine copy is kept as it is, without the shared directive's added rules.
- F12, fixed as proposed: D5's order names D8's items one by one: the instruction files, then D9's pointers, then the skill listing, then the plugin contexts.
- F13, fixed as proposed: the D9 criterion covers a CLAUDE.md beside an `AGENTS.md`.
- F14, fixed as proposed: D9's Rationale says the codex-cli 0.164.0 binary holds `project_doc_max_bytes` and `AGENTS.override.md`, and I confirmed that with `strings`.
- F15, fixed as proposed: D2's bound says the allowlist reason is held by review, not by the scan.

## Self-check

- fix-consistency, MEDIUM, D1's producer scan has no carve-out for the orch, `lym shuttle run` or `lym shuttle open`: fixed. The scan skips the orch places, the shuttle CLI and `internal/testkit`. Merged with the matching fix-way-forward and fix-proposal findings.
- fix-consistency, MEDIUM, the D12 nudge criterion claims more than the body: fixed. The criterion names `lookup_run`, `slice_lines` with the read-again rule, and `edit_run`.
- fix-consistency, LOW, the new D5 criterion was out of order: fixed. It is merged into the D5 criterion line.
- fix-consistency, LOW, the support log's Turn 3 board-scope line and Q1 still said seats only: fixed.
- fix-consistency, LOW, Constraints did not name the shared lookup-check package: fixed in the `PATTERN-shuttle-provider-seam` line.
- fix-consistency, LOW, the plan writer's note covered seats only: fixed with a note on rendering for producers outside `seatengine`.
- fix-claims, MEDIUM, D10's bound ignored the Codex home writes and the sandbox probe: fixed.
  D10 uses a sandbox probe that reports the configured mode, and the bound names the Codex home's links, trust record and `config.toml`.
  The criterion matches.
- fix-claims, LOW, nothing at HEAD shows that the launch line is sourced from a script: fixed. The support log no longer claims that tmux's limit does not apply.
- fix-proposal, MEDIUM, the D1 scan breaks on specs the record keeps out: fixed, merged with the fix-consistency finding.
- fix-proposal, MEDIUM, D10's verdict did not say how an unrun plugin context counts: fixed. The verdict compares plugin contexts by key alone, so an unrun one counts as present, and the criterion says the verb passes with contexts listed unrun.
- fix-proposal, LOW, D8 cited Claude Code's 60-second hook timeout with no version, and it may be stale: fixed. The fallback is now a new codex.yaml key, `plugin_hook_timeout`, whose template default is 10 seconds.
- fix-way-forward, MEDIUM, the D1 scan's way forward cannot be reached for orch, shuttle CLI and testkit specs: fixed, merged with the fix-consistency finding.
- fix-way-forward, LOW, D5's size refusal named no way forward: fixed. It says to shorten that file or turn its plugin off in the session scope.
- fix-way-forward, LOW, D10 pointed at a test-only hook runner: fixed. The runner and probe are production code that the parity test shares.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
