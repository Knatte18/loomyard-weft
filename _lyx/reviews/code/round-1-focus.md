---
round: 1
focus:
  - "internal/vscode/keybindings.go keybindingsBlock: does every rendered entry stay valid JSONC, and do the four Alt-arrow lines render byte-identical to before (args, when, quoting of the escape sequences)?"
  - "internal/vscode/keybindings.go and config.go: does the ctrl+alt+a entry's args equal the task label through the one attachTaskLabel constant, and does strconv.Quote yield the JSON string the record's acceptance criteria require?"
  - "Docs: do the package doc, the keybindings.go file header and ideengine.Spawn's doc all stop calling the block terminal-only, and does any other doc or comment in the repo still say the block is terminal-only or tasks.json runs only from Tasks: Run Task?"
  - "internal/vscode/keybindings_test.go: does the new TestKeybindingsBlock duplicate coverage of an existing merge or seed test, and is the acceptance criterion about an older lyx block replaced in place pinned by an existing test?"
  - "Scope: does the diff leave the task itself (no runOptions, no dependsOn) and the SeedKeybindings merge and write mechanics untouched, as the record's scope requires?"
---

Small diff over four files; the focus covers rendering fidelity, label coupling, doc wording and test economy.
