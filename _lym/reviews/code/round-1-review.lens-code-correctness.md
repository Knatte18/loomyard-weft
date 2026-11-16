```yaml
lens: code-correctness
outcome: reported
```

Lens code-correctness: I traced each function and branch the merge range changes.

**Cases run:**

- **`launchOrch` resume branch, `st.SessionID="s1"`, `remoteControlName=""`** (`internal/orchcli/start.go:733-743`): `resumeOrch("s1","")` builds a spec with an empty `RemoteControlName`, so the session runs without remote control. This matches `main`'s inline version. Correct.
- **Same branch, `remoteControlName="neve::ly:orch"`:** the spec carries the name, both on the first `StartSession` and on the retry after a held session, because the same `spec` is reused (`start.go:766,776`). Correct.
- **Same branch, `StartSession` fails with `ErrNoTranscript`:** `resumeFailure` wraps it with `%w` (`start.go:806`), so `errors.Is` at `start.go:738` still matches. It falls through to a fresh launch with `abandoned="s1"`, and the fresh launch also passes `remoteControlName` (`start.go:752`). Correct.
- **Same branch, `RenderReloadPrompt` fails:** the error comes back unwrapped, and `launchOrch` returns it because it is not `ErrNoTranscript`. Same as `main`'s side. Correct.
- **`resumeOrch` with a `SessionHeldError` and `orchRegistered == nil`:** `registered` is false, so it refuses with the holder's pid and launches nothing (`start.go:782-792`). Correct.
- **`resumeOrch` with a held session whose holder is registered but `endOrch` fails:** it returns a wrapped error with a way forward and does not retry. Correct.
- **`resumeOrch` retry after the holder ended, with the second `StartSession` returning nil:** `resumeFailure` returns nil, so `started` is returned. Correct.
- **`wireCLI`, beam and prime, after `endTestMap` was dropped** (`internal/orchcli/cli.go:292-312`): the remaining seams are wired for every owner. The only reader of `endTestMap` was `beamDown`, and `main` removed it from there (`stop.go` diff). There is no nil path left. Correct.
- **`runproc.subjectPassSpared` without `RoleTestMap`** (`end.go:100`): `RoleTestMap` no longer exists in the tree (grep found nothing), so no live record can carry it. The spared set for the hub owner's pass still leaves push, Slack relay and teardown alone. Correct.
- **`TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast` fixture after removing pid 81** (`end_test.go:479-485`): the `HubOwner` row still expects only `term pid 84` then `term pid 85`. The ordering and the spared roles are still pinned. Correct.
- **`runproc/doc.go` conflict text** (`doc.go:9-10,61-62`): the hub roles list is `RoleSync`, `RoleSlackRelay` and `RoleAdopt`, which matches `subjectPassSpared` and the absence of `RoleTestMap`. Correct.

**Findings:** none in this lens.

Tool calls:
- I read `orchSpec` (`start.go:205-245`) and `resumeFailure` (`start.go:802-807`). My context held only `orchSpec`'s signature line and not `resumeFailure`'s body, and I needed both to trace the `ErrNoTranscript` wrap and the empty-name case.
