```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The sibling sites for F4 and the post/warning docs already agree with the fix:
- The only other inject-only wording is `internal/creelengine/pool.go:412`, `internal/creelengine/doc.go:56` and `internal/creelengine/server.go:423`. Each describes stall notes, which are inject-specific, so none needed the F4 change.
- `internal/creelengine/doc.go:103` and `internal/creelengine/protocol.go:99` already say a waiting post sends the warning as progress.
- The only code sites that read or write `receiver_live` are `server.go`, `verbs.go` and `cli.go`. The tests cover the pending and final responses (F1) and the CLI `--wait-reply` path (F2).

Not covered, though the review did not ask for it: no CLI test checks that a post to a program receiver omits `receiver_live`. The review made that part of F2 conditional on the rig making it cheap.

Calls made:
1. grep for progress and inject wording in non-test creelengine and creelcli files, to find sibling sites for F4.
2. grep for `receiver_live` and `ReceiverLive` outside the files I already held, to find other sites the fixes should cover.
3. grep for "warning" in `creelengine/doc.go`, plus `protocol.go` lines 95–100, to confirm the docs already describe the post warning as progress.
