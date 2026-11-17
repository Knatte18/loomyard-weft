# Plan round 1 — fixer report

## Changes

- **F1** (fixed as proposed): card 8's tests no longer sit under one "Tests, tier0" heading.
  `TestUserSetup_PluginSessionStartContexts` is tier0.
  `TestRunPluginHook` is tier1 and sits in its own test file, `pluginhook_integration_test.go`, whose tests all name tier1, since its rows spawn `sh` and `sleep`.
- **F2** (fixed as proposed): card 5 now refuses a setting whose length is at least `maxArgumentBytes`, since `MAX_ARG_STRLEN` counts the terminating NUL.
  A setting of `maxArgumentBytes`-1 bytes goes ahead.
  `AtTheArgumentCap` sizes the setting at `maxArgumentBytes`-1, and `OverTheArgumentCap` at exactly `maxArgumentBytes`.
- **F3** (fixed as proposed): card 10 lists `plan:internal/shuttleengine/codexengine#TestPrepare_SubagentTerms` under Edit.
  Its test note says that test's want lines also add `projectDocMaxBytes: defaultProjectDocMaxBytes`.
- **F4** (fixed as proposed): card 10's `unreadClaudeFiles` skips the anchor's CLAUDE.md only when the anchor's `AGENTS.override.md` opens with `heddleengine.Marker`.
  An unmarked override leaves the CLAUDE.md named.
  `internal/heddleengine#Marker` is added under Uses, and `TestPrepare_InstructionFiles` gains the unmarked-override row.
- **F5** (fixed as proposed, the review's second option): card 4 drops the empty-`Root` branch and the `NoRoot` row.
  Every `RenderRequest` builder sets `Root`.
  The production builders are `requestOf` and `renderInto`.
  The test builders are `f.request()`, set to the fixture's anchor, and `TestRender_WritesAtTheAnchorAndLeavesNoUntrackedFile`, set to `event.Root`.
  Both test builders move to Edit, and the ImpactSummary counts all four builders.
  The first option, keeping a branch reached only by test builders, would keep a code path for a state production never reaches.
  `render_integration_test.go:48` is a second builder that the review did not name.
- **F6** (fixed as proposed): in card 7, an unreadable CLAUDE.md above `RepoRoot` becomes a `Warnings` entry and is left out of `InstructionFiles`.
  Only the global file's read error is still an error.
  `TestUserSetup_InstructionFiles` gains the matching row.
  The self-check found that the `instructionFiles` glyph had no warnings result, so it now returns `(items, warnings, err)`.
- **F8** (fixed as proposed): card 2 lists `internal/heddleengine#Target` under Edit.
  Its Intent now says the doc comment's `RendersInto` names the targets, of either delivery, that a shared target's entries reach.
- **F9** (fixed as proposed): card 3 moves `pattern/PATTERN-heddle-instructions.md` from Extends to Edit.
- **F10** (fixed differently): card 8 now states the bound and holds it as review-held.
  The bound: the timeout and the group kill hold only while the lym process running `Prepare` lives, and a hook left by a killed `Prepare` runs to its own end, owned by no record.
  The overview's "Plugin hook processes" Shared Decision carries the bound too.
  - Not made: registering each run with `runproc.ChildRecord` under the run's owner.
  - Why: `runproc.Register(dir, owner, rec)` needs a runproc directory and a run owner, and no shuttle engine holds either.
    No non-test file of `internal/shuttleengine` names `Owner`, `RunProcDir` or `HubPath`.
    `Codex.Prepare` gets only `runDir`, `spec` and `cfg`.
  - So registering would need a new owner path through shuttle that no decision asks for.
    `PATTERN-child-processes` asks for a record only for a run's detached or long-lived processes.
  - The review's issue was that the bound was unstated, and the card now states it.

`lym loom validate-plan` reports `ok: true` with only informational findings.

## Self-check

- fix-consistency, LOW: the overview's hook-process Shared Decision did not carry card 8's new bound. Fixed as proposed.
- fix-claims, MEDIUM: card 4 claimed the integration test's git-status assertion covers the import record, but the record lives in the weft worktree's `.lym`. Fixed as proposed: the claim is dropped.
- fix-claims, LOW: card 10 said `Render` reads the file's head, but it reads the whole file and compares a prefix. Fixed as proposed: the wording now says it compares a prefix.
- fix-proposal, MEDIUM: on F11, `CLAUDE_PROJECT_DIR` should be `req.WorkDir`, not `req.RepoRoot`. I flagged it to the parent, who ruled `req.WorkDir`; it is carried out under Rulings.
- fix-proposal, MEDIUM: on F6, the `instructionFiles` glyph could not return a warning. Fixed as proposed: the glyph gains a `warnings` result.
- fix-proposal, LOW: on F7, the zero `UserSetup` dropped the parts that do not depend on `HOME`. I flagged it to the parent, who replaced the ruling; it is carried out under Rulings.
- fix-affected-set: no findings. Its F3, F5 and F8 sets are all covered.

## Disputed

None.

## Deferred

None.

## Rulings

- **F7**, from the parent's decision records.
  - First ruling: a `claudeConfigDir` failure returns the zero `UserSetup` with one warning naming `CLAUDE_CONFIG_DIR` or `HOME`, never an error.
    I carried it out in card 7, then flagged to the parent (creel #1556) that it dropped the parts that do not depend on `HOME`.
  - Replacing ruling: leave out only the parts that need the config directory, keep the rest, add one warning and return no error.
    Card 7 now leaves out the global file, the user and plugin skills, and card 8's plugin contexts.
    It keeps the CLAUDE.md files above `RepoRoot` and the project skills.
- **F11**, from the parent's decision records.
  - First ruling: set `CLAUDE_PROJECT_DIR` to `req.RepoRoot` beside `CLAUDE_PLUGIN_ROOT`.
    I carried it out in card 8, then flagged to the parent (creel #1556) that in an anchored worktree Claude Code would set the anchor instead.
  - Replacing ruling: `CLAUDE_PROJECT_DIR` is `req.WorkDir`.
    Card 8's Env sets it, and the `TestUserSetup_PluginSessionStartContexts` row asserts that value together with `CLAUDE_PLUGIN_ROOT`.
