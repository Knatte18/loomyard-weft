```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked that every claim the fix added or rewrote holds against the code at HEAD.

no findings

- **Card 1 relaunch row:** the claim "relaunches its stored command `exit`" holds. `AddStrand` stores `ResumeCmd` from the spec (`internal/reedengine/strand.go:418`), and the card adds `pawl` with cmd `exit` and no resume command. The IfAbsent relaunch launches `ResumeCmd`, else `Cmd` (`strand.go:629-631`), so it runs `exit`.
- **Card 1 `isOverview` comment:** this changes only the comment, and the current comment sits at `internal/reedengine/ownwindow.go:170`.
- **Card 2 follow help:** `followCmd` is a method on `reedCLI` (`internal/reedcli/follow.go:31`), and its help says "else the overview" (`follow.go:38`). `reedcli/doc.go` does not mention the overview, so the claim that outside reedengine only `lym reed follow`'s help text changes holds.
- **Card 3:** the `runs.go` file header (`internal/orchcli/runs.go:1`) and `runsStrandAddSpec`'s doc comment (`runs.go:39`) both say "the session's overview window". The orch entry's clause in `docs/overview.md:424` matches the card's quote word for word.

Tool calls:
- grep `ResumeCmd` in `internal/reedengine/strand.go`: I needed to know whether AddStrand stores a resume command, which the span the fixer read did not show.
