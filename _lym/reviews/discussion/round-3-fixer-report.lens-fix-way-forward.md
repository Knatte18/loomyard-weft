```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every refusal, error, next step, row or test the fix introduced can be carried out from the state it names and leads to progress.

No findings.

What I checked:
- **Main-checkout Warn (D5, D6, Prepare acceptance bullet):** Prepare can test this from the state it holds. It reads the main checkout root through `internal/dotgit` (D5), and it can check whether the worktree root has a `.codex/` directory and whether the main checkout has a `.codex/config.toml`. The Warn refuses nothing. The worktree Warn before the fix offered no way forward either, so the fix added no dead end.
- **Stop-and-return on a blocked sandbox write (D6, Open risks):** a build can still carry this out. The fix narrowed the condition from "Codex ignores the key" to "the sandbox blocks the write through the links", which a single write test shows, and the next step is still returning to the parent before D2.
- **Record without the assistant fields (D2, acceptance bullet):** this reading always leads somewhere. The round runs with no assistant and the chair under the bare round, which matches how the chair ran before the assistant fields existed. A round recorded with an empty assistant choice ends the same way, so the reading never strands a probe or resume.
- **Marker and outputs under `.lym` (D1):** this adds no new step. The chair's `lym shuttle await <marker>` and the Go copy on a done marker still reach the paths D1 derives.

Tool calls: none.
