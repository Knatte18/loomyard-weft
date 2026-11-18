---
round: 2
focus:
  - "Both engines' python hook commands (claudeengine settings.go pythonAllowedCheck, codexengine guards.go pythonAllowedCheck): does each name the launching lym binary through selfexec.LymBinary or its injectable resolver rather than a bare lym from PATH, with the path shell-quoted, and do the tests render the command at that path?"
  - "internal/shuttlecli/hookpython.go shuttleConfigDirs: does the hub-versus-standalone choice match the launch's own mode test (preflight.BoardLymPresent), and do the standalone tests reach a real repository target, a subdirectory cwd, a missing state directory and a damaged hub without _lym?"
  - "internal/shuttlecli pythonAllowed: does a config that fails to load deny, and does a test reach that branch?"
  - "docs/overview.md Codex paragraph, the pythonAllowedCheck doc comments in both engines and _lym/landing/summary.md: do they match the diff as it now stands, including the standalone state-directory fallback?"
---
