```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over a diff that changes no Go code (CLAUDE.md, one orch stencil line, two board skill lines).

**List: each function or branch, the case run, correct or not**

- No function, branch or error path changes in the diff. All three files are prose (`CLAUDE.md`, `contracts/stencils/orch/orch-template-role.md`, `plugins/ly/skills/board/SKILL.md`), so there is nothing to run step by step.
- The one behavioural claim I could check against code is the board skill's added line "`lym board prune` removes it" (the done entry). `internal/boardengine/board.go:375-381` shows `Prune(apply bool)` removing every done entry under the write lock and returning the removed slugs. I did not run it. This agrees with the claim, so it is correct.
- The added orch-stencil sentence ("Never keep a durable note in an engine's session memory store...") describes no code path. It is not a code-correctness matter.

**Findings:** none.

**Tool calls:** none. The manifest already held the diff, `internal/boardengine/board.go:375-381` and the stencil greps.
