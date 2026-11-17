```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out: F1–F9 and F11, with F5 done as the review's second option.

**F11 — `MEDIUM`.** Changed text: card 8 Intent sets `CLAUDE_PROJECT_DIR` to `req.RepoRoot`; the `TestUserSetup_PluginSessionStartContexts` row asserts it "set to the request's `RepoRoot`".
- Summary: the value differs from what Claude Code sets in an anchored worktree, so a hook does not run "the way Claude Code runs it" (D8).
- Evidence: Claude Code sets `CLAUDE_PROJECT_DIR` to the directory the session started in.
  Card 10 says the session's working directory is "the engine's anchor, where the pane starts".
  An anchor can be a subdirectory: `hubforge.Shape{Anchor: "backend"}` in `render_integration_test.go:30-35`, and card 10's `src/app` fixture.
  The same card-8 stdin already uses `req.WorkDir` for `cwd`.
  With anchor `backend`, a hook under Claude sees `<worktree>/backend` but sees `<worktree>` under Codex.
  `req.WorkDir` matches Claude Code. The value comes from the parent's ruling, so it would need re-confirming with the parent.

**F6 — `MEDIUM`.** Changed text: card 7 says an unreadable CLAUDE.md above `RepoRoot` "is left out of `InstructionFiles` with a `Warnings` entry naming it".
- Summary: the card's own Create glyph `func instructionFiles(configDir, repoRoot string) ([]shuttleengine.InstructionItem, error)` has no way to return a warning.
- Evidence: the function either surfaces the unreadable file as an error, which the card now forbids, or drops it silently.
  So the new behaviour cannot be built as the function is signed. The glyph needs a warnings result, for example `([]shuttleengine.InstructionItem, []string, error)`.

**F7 — `LOW`.** Changed text: card 7 says that when `claudeConfigDir` fails, `UserSetup` "returns the zero `UserSetup` with one `Warnings` entry".
- Summary: the zero result also drops parts of the setup that never need the config directory.
- Evidence: the card resolves project skills from `.claude/skills` of `WorkDir` up to `RepoRoot`, and CLAUDE.md files above `RepoRoot` from the walk.
  Both resolve without `HOME`. With `HOME` unset, a Codex launch loses them although Claude itself would load them.
- This is the parent's ruling as written. I'm noting the case, not proposing that it be changed in this round.

**Proposals checked with no case found:**
- **F1:** `claudeengine` already holds tier1 `_integration_test.go` files.
- **F2:** the boundary holds; `MAX_ARG_STRLEN` counts the NUL.
- **F3:** `TestPrepare_UserSetup` and `TestPrepare_AppendsStandingPassages` assert content, not whole lines.
- **F4:** there is no import cycle. `heddleengine`'s dependencies hold neither `shuttleengine` nor `codexengine`.
- **F5:** every `WorktreeEvent` builder sets `Root`. `EventFor` sets it at `worktreeseam.go:64`, and `newSeamFixture` sets `Root: root`. Dropping the empty-Root branch therefore leaves no builder reaching an empty `Root`.
- **F8 and F9:** no case found.

Tool calls:
1. Grepped `fabricengine`'s `WorktreeEvent` `Root` and `newSeamFixture`'s event literal. My context showed neither, and I needed them to test whether dropping the F5 branch is safe.
2. Ran `go list -deps ./internal/heddleengine`. My context lacked `heddleengine`'s transitive imports, and I needed them to check for a cycle from F4's new import in `codexengine`.
