```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the fix introduced can be carried out from the state it names and leads to progress.

Ways forward the fix introduced:
1. Open risks: the operator runs `lym heddle set` to add `codex-file` to the targets of `nicknames`.
2. D13: a logout removes the `auth.json` link in the lym home, and Prepare creates it again (D5).
3. D1 bound: a lym process that dies before it writes the marker leaves the chair waiting until its own review timeout or until a resume's probe writes the marker.
4. D6 bound: Prepare logs a Warn naming `.codex/config.toml` when the worktree holds one.
5. D6: a dropped positional prompt ends the run at its timeout, with the marker reading `timeout`.
6. D2: the kept copy is written before the done marker, so a reboot that clears `/tmp` loses no done report.

Findings:

- **LOW** — Open risks, "Parity of repository instructions needs one operator step: `lym heddle set` adding `codex-file` to the targets of the repository entry `nicknames`"; D12, "A repository-layer entry reaches a Codex seat once the operator adds `codex-file` to its targets with `lym heddle set`".
  Summary: the step names only the verb and leaves out the render that an existing worktree needs before it sees the change, so calling it "one operator step" overstates it.
  Evidence: `lym heddle --help` says "A worktree takes a change only when lym makes the worktree, at a take-in render or at a hand render". `lym heddle set --help` shows the concrete form: `lym heddle set nicknames --targets claude-file,codex-file`. That form is reachable, but a worktree that already exists gets the change only from `lym heddle render` or a take-in.

Ways forward 2 to 6 are reachable as stated: D5's Prepare "creates any missing link"; D1 already describes the resume probe; the timeout ending matches D7's bound.

Tool calls:
- `lym heddle set --help`: to check that the operator step's verb takes a `--targets` flag; my context had only `lym heddle --help`.
