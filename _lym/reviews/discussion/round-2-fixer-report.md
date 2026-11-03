# Round 2 fixer-report — discussion

## Changes

- F1 (BLOCKING), fixed as proposed.
  D2 writes the report directly under the anchor's `.lym` at a path from the review path and attempt token, with no `/tmp` directory, 0700 mode, hash name or kept copy, and writes the done marker once the seat gate has passed.
  D1's wait directive reads the report itself.
  D6's launch line carries `-c 'sandbox_workspace_write.writable_roots=["<weft checkout>"]'`, marked to be proven in the build.
  The Constraints bullet says `.lym` and `_lym` resolve into the weft checkout, which D6 adds as a writable root.
  The macOS `/tmp` Open risk is gone, the D2 acceptance criteria name the `.lym` path and the gate-then-marker order, and Q5 and the support log's rejected alternative on writable roots follow the parent's entry.
- F2 (MEDIUM), fixed as proposed.
  D6's Bound states that the session can write anywhere in the worktree and the weft checkout, names the weft content that reaches, and states that only the prompt bounds it, the seat gate checking its own report alone.
- F3 (MEDIUM), fixed as proposed.
  D2 records the attempt token before attempt 1 and rewrites the record with `<round>b` before attempt 2 spawns, and `probeLiveReview` builds the whole table, chair included, from the recorded token; the acceptance criterion covers the chair.
- F4 (MEDIUM), fixed as proposed, narrowed by the self-check.
  A new Constraint states that Codex 0.164.0 loads `rules/*.rules` from the home and each trusted project layer's `.codex/`, with no TUI switch.
  D6's Bound states that an allow rule lifts the sandbox.
  D5 has Prepare refuse the launch under a `.codex/rules/` in the worktree, with a Warn and the refusal's text in the `not started` marker, and the D5/D6 criterion covers it.
  The main checkout is not covered: the fix-proposal fork found that its `.codex/` is a different directory from the worktree's, so refusing on it would stop an assistant over rules its session never loads.
- F5 (MEDIUM), fixed with own choice among the proposed options.
  D12 makes the `codex-file` render carry every repository-layer entry aimed at `claude-file`, so the assistant gets the repository's instructions by default, citing the operator rule of 2026-10-27; shipped entries stay out, with the reason and a bound.
  The `nicknames` Open risk is gone, and Q11 and the D12 criterion follow.
  The stencil pointer to `CLAUDE.local.md` was not chosen: it carries the shipped entries' directions for Claude tools the assistant lacks.
- F6 (LOW), fixed as proposed: the fan Open risk is deleted and Q4 is Resolved, citing the parent's entry.
- F7 (LOW), fixed as proposed: D5 has Prepare warn on an `auth.json` or `sessions` that is a regular file or a link elsewhere, naming the path and the way forward; D13 and the D5 criterion follow.
- F8 (LOW), fixed differently.
  Instead of the config refusal it proposed first, the table builder caps the assistant's timeout so that its start with retries, its timeout and one await cap end before the chair's timeout, `review_timeout_min` or shuttle's `run_timeout_min` when that key is 0, with a Warn when it caps.
  The refusal was made first and the self-check rejected it: `internal/loomengine/config.go` accepts `review_timeout_min: 0` on purpose as deferring to shuttle, which the refusal would break, and "below" with a one-minute margin still lets the chair time out inside its wait.
  The review's fix key names the cap as its alternative.
  D1's Bound, D3 and the D3 criterion state it.
- F9 (LOW), fixed as proposed: D1 adds the `gate failed` marker text, never copied beside the review, and "let the run through" is dropped from D1, D2 and the criterion.
- F10 (LOW), fixed as proposed: D12 states that `fileText` renders nothing while no entry names `codex-file`, so Codex loads `AGENTS.md` or `CLAUDE.md` itself.
- F11 (NIT), fixed as proposed: Turn 4 reads "which shuttle does not retry".

The support log gained a Round 2 entry and rejected alternatives for the choices above.

## Self-check

- fix-consistency, LOW, D13 left stale by F7: fixed as proposed; D13 states that a login before the next Prepare leaves a regular file Prepare warns about.
- fix-claims, MEDIUM, and fix-proposal, MEDIUM, merged: "below" does not keep the chair's wait inside its timeout: fixed with own change, the table builder's cap (F8 above).
- fix-claims, LOW, D12's render sentence reads as current code: fixed as proposed; D12 states it as a decision that the render adds `codex-file` to a target list naming `claude-file`, and the criterion covers it.
- fix-proposal, LOW, and fix-way-forward, LOW, merged: the untested writable root has no fallback: fixed as proposed; D6 sends a build that disproves it back to the parent, and an Open risk states the failure.
- fix-proposal, LOW, main checkout's `.codex/rules/` is not loaded by the worktree session: fixed as proposed; the refusal covers the worktree alone.
- fix-proposal, LOW, D12's layer rule rests on a content reason: fixed as proposed; the reason is now the layer, and a bound states that a repository entry naming Claude tools reaches the assistant.
- fix-way-forward, BLOCKING, the timeout refusal rejects a legitimate `review_timeout_min: 0`: fixed with own change, the cap (F8 above); config load no longer relates the keys.
- fix-way-forward, MEDIUM, the `.codex/rules/` refusal never reaches the operator: fixed as proposed; the refusal logs a Warn and puts its text in the marker, as D11 does.
- fix-way-forward, MEDIUM, clearing the assist key cannot help the refused round: fixed as proposed; the way forward says it applies to later rounds.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
