```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff's changed functions, branches and error paths.

List:
- `internal/boardcli/intake.go` `closeCmd`, `Short` string (help text only, no logic): correct. It ran nothing because it is a constant.
- `internal/boardcli/intake.go` `closeCmd`, `Long` new example `{"issue":15,"reason":"Already fixed by 6fa7f1740 on main: …","completed":true}`: I checked the example's keys against `intakeClosePayload` (`required: issue, reason`, `optional: completed`, intake.go:33). Every key is declared, so `decodeIntakePayload` (intake.go:366) accepts the example and the help shows a form that runs. Correct.
- `closeCmd.RunE` with an empty `args`, a `hubRepo` failure, or a dispatch to `intakeClose`: the diff leaves this unchanged. No changed branch.
- `contracts/orchrole/role-intake.md` and `plugins/ly/skills/board/SKILL.md` are prose with no executable path. They are outside this lens.

Findings: none. The diff changes no function body, branch or error path. Its only Go change is help-text constants, and the new example payload uses only declared keys.

Tool calls:
- `grep` for `intakeClosePayload`, `Completed` and `intakeClose` in `internal/boardcli/intake*.go`. I needed the declared payload keys to check the new help example, and my context did not have the payload definition.
