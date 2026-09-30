# Card 5 — shedtransient: translate leaf classifications into the mark

**Create:**
- `plan:internal/shedtransient#Class` -> `func Class(err error) shedengine.TransientClass`
- `plan:internal/shedtransient#Mark` -> `func Mark(err error) error`
- `internal/shedtransient/doc.go`
- `internal/shedtransient/shedtransient.go`
- `internal/shedtransient/shedtransient_test.go`
- `internal/shedtransient/seam_enforcement_test.go`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `plan:internal/shedengine#TransientClass`
- `plan:internal/shedengine#TransientGitTransport`
- `plan:internal/shedengine#TransientGitHubAPI`
- `plan:internal/shedengine#TransientAgentStart`
- `plan:internal/shedengine#MarkTransient`
- `plan:internal/shedengine#TransientOf`
- `plan:internal/gitexec#IsTransportFailure`
- `plan:internal/githubclient#IsTransient`
- `internal/shuttleengine#ErrNotStarted`

**Intent:**
A new leaf package, `internal/shedtransient`, is the one place that translates a lower-level package's own classification into `internal/shedengine`'s transient mark (decision record D1, D2).
It exists so `shedengine` stays stdlib-only and the leaf packages gain no `shedengine` import, while the producer boundary (the classifier `shedbuild.NewShed` tells a `Shed`) and the bootstrap boundary (the `PreStep` hooks) share a single mapping.
The landingshed producers (card 8) call `Class` too, to decide which of their failures to return as an error rather than a verdict.

`Class` returns, in this order:
- the class already on `err`, via `shedengine.TransientOf`, so a mark set closer to the failure wins;
- `TransientAgentStart` when `errors.Is(err, shuttleengine.ErrNotStarted)`;
- `TransientGitTransport` when `gitexec.IsTransportFailure(err)`;
- `TransientGitHubAPI` when `githubclient.IsTransient(err)`;
- otherwise the empty class.
A nil error gives the empty class.

`Mark` returns `shedengine.MarkTransient(Class(err), err)`: the error unchanged when nothing matches, and never double-wrapped when the error already carries a mark.

`doc.go` states the package's role and its import allowlist: stdlib, `shedengine`, `shuttleengine`, `gitexec`, `githubclient`, and nothing else.
`seam_enforcement_test.go` enforces that allowlist over the package's non-test files, in the same membership-list shape as `internal/shedverbs/seam_enforcement_test.go`.

`docs/overview.md`'s package tree gains an `internal/shedtransient/` line beside `internal/shedengine/`: the one translation from lower-level failure classifications into the shed engine's transient mark.

`shedtransient_test.go` is untagged and spawns nothing; a table drives `Class` and `Mark` over:
- an `ErrNotStarted` wrapped twice with `%w` → `agent-start`;
- a `*gitexec.GitError` for `-c push.autoSetupRemote=true push` whose stderr carries `Could not resolve host`, wrapped as `gitrepo` wraps it → `git-transport`;
- a rejected push `*gitexec.GitError` → empty;
- a 503 `*github.ErrorResponse` → `github-api`;
- a 404 `*github.ErrorResponse` and a `*github.RateLimitError` → empty;
- an error already marked `github-api` that also wraps `ErrNotStarted` → `github-api`, and `Mark` leaves it with exactly one mark;
- a plain error and nil → empty, with `Mark` returning the same error value.

**Commit:** 5: shedtransient: one translation from leaf classifications into the transient mark
