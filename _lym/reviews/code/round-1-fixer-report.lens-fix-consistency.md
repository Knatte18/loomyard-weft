Task: check the change set for agreement with its surroundings, through the fix-consistency lens.

No findings.

- **Instruction-file literals:** The only non-test spellings of `CLAUDE.local.md` and `AGENTS.override.md` left in production Go code are the two new constants at `internal/heddleengine/target.go:30` and `:34`. `localInstructionsFileName` is gone.
- **Codex `Omitted` readers:** Of the three, only the manifest one changed. The `setup.Omitted` loop in `codexengine/usersetup.go:39` is the other consumer, and the grep shows no empty-`Plugin` special case that disagrees with the new keying. I did not read its body.
- **Claude and Codex omitted keys:** The Codex empty-`Plugin` key is now `hook.Name`, the same key `claudeengine/manifest.go` uses for an omitted hook with no `Hook`. The two engines agree on the case F3 named.
- **Way-forward wording:** The Claude manifest-write error now names a way forward, as the Codex one already did, so both Prepare doc comments hold.

Calls made:
1. `grep` for `Omitted` in the codexengine and shuttleparity production files, and for the two file-name literals across production Go code: my context held only the files the fixer read, not the other consumers of `setup.Omitted` or the remaining literal sites.
