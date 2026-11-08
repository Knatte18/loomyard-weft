## Goal

```markdown
## Seen (operator, 2026-11-07)

`lym slack login` in a terminal printed nothing and seemed to hang: `internal/slackcli/cli.go` reads the token with `io.ReadAll` on standard input, so it waits for end of input (Ctrl+D) with no prompt, and the pasted token shows on screen.

## Wanted

- On a terminal, `login` prints a prompt naming what to paste, reads one line with echo off, and goes on at Enter.
- Piped input keeps working as today, so a script can pass the token on standard input.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
