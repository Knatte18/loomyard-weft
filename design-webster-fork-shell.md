# Webster fork used sed -i and an rm on a computed /tmp path, stalling on a permission prompt

`webster-fork-shell` · Note · bug, webster, stencil

## Seen (2026-10-11, `shuttle-hook-first`, Webster batch 01)

The batch 01 implementer fork ran, from Merriam's session:

```
tmux -L askexp kill-server; rm -rf "$(cat /tmp/claude-1000/askdir)" /tmp/claude-1000/askdir; cd internal/shuttlecli && sed -i '604s|Write(|Edit(|' smoke_turns_test.go && sed -n 603,605p smoke_turns_test.go
```

- Claude Code stopped it with a permission prompt ("Dangerous rm operation on statically-unresolvable target"), which fires in bypass mode too, so the run waited on the operator.
- The fork edited a test file with `sed -i`, against `contracts/stencils/edit/edit-directive.md` ("never `sed`").
- It kept experiment state in `/tmp` and ran its own tmux server (`-L askexp`); no Webster stencil names a scratch place or forbids a system temp directory.

## Open

- Whether the edit directive reaches Webster implementer forks at all, and if so why the fork ignored it.
- Whether the implementer stencil should name a scratch place inside the pair and forbid `rm` on a computed path.
- Whether a pending permission prompt in a fork should reach the operator as a notice instead of waiting unseen.
