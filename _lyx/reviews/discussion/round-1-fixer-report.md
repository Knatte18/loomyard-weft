# Discussion round 1 — fixer report

Every finding was checked against `internal/reedengine` and against tmux 3.6 on a private socket, and every finding held.
All seven are fixed.

## Validation

- F1: `ensureServerAndSessionLocked` in `internal/reedengine/lifecycle.go` runs `serverSpawnArgv` and `pinBootOptionsLocked` whenever the worktree's session is missing, joins included, and `reed.lock` lives in the worktree's own state dir (`lock.go`).
- F2: a `source-file` of `set -g destroy-unattached on` after `new-session -d` exits 0 and the server exits.
  A session-local `destroy-unattached off` set first survives `set -g destroy-unattached on`, but not an untargeted `set destroy-unattached on`.
- F3: an unclosed `if-shell true {` gives `syntax error`, exit 1, and the preceding `set -g @m1 yes` is unset.
  An unknown option between two valid lines gives exit 1 with both valid lines applied.
- F4: the clause sat in D5 as quoted.
- F5: the teardown paths in `lifecycle.go` and `overlay.go` kill the server only when `list-sessions` is empty.
  Reed's session names come from the worktree root (`server.go`'s `SessionName`), so a file-created session can't be told apart by its name.
- F6: `source-file` of existing `a[1].conf` and `c\x.conf` fails with "No such file or directory".
  `b{1}.conf` and a literal `d*.conf` load.
- F7: the bullet sat under Auto-mode assumptions and repeats Interview item 6.

## What changed

`decision-record.md`:

- F1: D3 identifies the starter in two steps: a boot that found no live server before its spawn is a candidate, and one in-server check-and-set of `@lyx_tmux_conf_claimed` picks exactly one candidate.
  It also covers concurrent boots and servers started before this change.
  An acceptance criterion now covers two concurrent boots sourcing exactly once.
  Scope gains the item.
- F2: D5 is rewritten.
  It sets a session-local `destroy-unattached off` before the claim, pins `-g destroy-unattached off` and `exit-unattached off` after it, and runs a post-claim session check that fails the boot, naming key, variable and path.
  The third acceptance criterion now names the `-g` forms and covers a later worktree's session, and a new criterion covers the untargeted form failing the boot.
  D4's bound and an open risk name D5's check, and the open risk covers a concurrent boot's session.
- F3: D4's rationale separates a parse error, which applies nothing, from a run-time error, which applies the rest.
  The syntax-error criterion is split into those two cases.
- F4: D5's open clause is dropped, and D5 states that the pin set is closed.
  The first open risk now names the closed set as the bound.
- F5: new D8.
  Reed tags its sessions with `@lyx_reed_session`, and on a server whose claim is `sourced`, teardown, discovery, reaping and status count only tagged sessions.
  Servers without a sourced file keep today's rule.
  The Q11 open risk is removed, an acceptance criterion is added, and D1's bound and Scope name D8.
- F6: D2 refuses `*`, `?`, `[` and `\`, with the rationale.
  The path-refusal criterion gains a `[` case, and the glob open risk is removed.
- F7: the Auto-mode bullet is deleted.

`support-log.md`:

- Q7, Q8 and Q11 are updated, and Q12 (starter identification) and Q13 (glob characters) are added.
- Four rejected alternatives are added: a liveness probe alone, a pin only after sourcing, escaping glob characters, and killing file-created sessions after sourcing.

`lyx loom validate-discussion` exits 0 on the edited pair.

## Disputed

None.

## Deferred

None.
