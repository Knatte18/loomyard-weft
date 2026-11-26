---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The proposal lets a dbus-send through when any argument word is --system. grep -E has no negation, so the pattern has to spell out every word that is not --system, and a version that parsed quotes and escapes let calls through that it could not read (escaped quote, tab). The pattern instead passes a dbus-send only when --system is reached through unquoted words, and refuses --session, --bus or --peer beside it. A quoted word, escape, continuation, tab, redirect or substitution before --system is refused although the call may reach the system bus."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- F1 (`internal/shellguard/rows.go`): `hub-tmux-server` matches `-L` and `-S` at the end of a cluster of tmux's value-less flags.
  The row comment names the quoted socket path with a spaced directory as passing.
  Rows added to `TestRows_PatternsDenyAndAllow`, including an allow row for that path.
- F2 (`internal/shellguard/rows_integration_test.go`): rows for both guards in a double-quoted `sh -c` script, and for a single-quoted `-L` name.
- F3, fixed differently (see the frontmatter): `session-bus` passes `dbus-send --system`.
  The steer, pattern and row comments, the `machine-sleep` comment, `doc.go`, the summary and the settings golden follow.
- F4 (`_lym/landing/summary.md`): the RC click's changed errors and the blast radius are stated.
  The file is a `_lym` record, so no commit holds it.
- F5 (`contracts/stencils/sampler/sampler-template-agent.md`): a trial server's name never opens with `lym-`.

Commits: `091e34046` (F1 to F4), `04c542ac2` (F5), `6a923f978` (self-check), `745cbe1e3` (gate repair). Nothing pushed.

The gate refused comment lines this round added to `internal/shellguard/rows.go`, for a wrapped sentence and for lines of 120 characters or more.
`745cbe1e3` rewrites them as shorter sentences, one per line, with no code change; `lym gate test ./internal/shellguard` passed in tier 0 afterwards.

The settings golden has no update flag, so a temporary test rendered it and was removed before each commit.
`lym gate test` passed on the touched packages in tier 0 and tier 1.

## Self-check

- fix-code-correctness, MEDIUM, `dbus-send` branch let through calls it could not parse (escaped quote, tab): fixed. Anything unreadable before `--system` is now refused.
- fix-code-correctness, LOW, pattern comment's list of false denies incomplete: fixed.
- fix-consistency, LOW, two summary lines over the line limit: fixed.
- fix-consistency, LOW, `Forms` still lists a bare `dbus-send`: fixed differently. The entry stays so the no-denied-recovery scan still covers it, and the row comment says what it stands for.
- fix-consistency, LOW, `--system` beside `--session` or `--bus` passed unstated: fixed. Such a call is now refused.
- fix-proposal, LOW, `--system --session` passed: fixed, with a test row.
- fix-proposal, LOW, tab-separated call passed: fixed, with a test row.
- fix-proposal, LOW, pattern comment's bound incomplete: fixed.
- fix-affected-set, MEDIUM, the steer lost the space after its colon: fixed.
- fix-affected-set, LOW, no row for a single-quoted `-S` path: fixed.

The fix-affected-set lens also noted, outside the change set, that the `tools/sandbox/SANDBOX-*-SUITE.md` files tell agents to run `tmux -L <socket>` against a hub server, which the new row would refuse in a lym session.
I did not verify or change this.

## Disputed

None.

## Deferred

None.

## Rulings

None.
