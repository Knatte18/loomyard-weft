# Card 8 — push-auth-class

**Create:**
- `plan:internal/gitrepo#ErrPushAuthRefused` -> `var ErrPushAuthRefused error`
- `plan:internal/gitrepo#authRefusalTriggers` -> `var authRefusalTriggers []string`

**Edit:**
- `internal/gitrepo#Repo.PushRebaseFree`
- `internal/gitrepo_test#TestPush`

**Intent:** D10's auth or permission class needs a push error distinguishable from a rejection and a transport failure.
`Repo.PushRebaseFree` checks git's stderr for an auth or permission refusal (`authRefusalTriggers`: GitHub's `Permission to`, `denied to`, `returned error: 403`, `Authentication failed`, `could not read Username`) before its rejection check, and returns `ErrPushAuthRefused` bare, as `ErrPushRejected` is returned bare, with git's last error line kept on a wrapping error that unwraps to it; the bounded `(failed)` retry never retries it.
The open risk the record names holds: a wording change moves a 403 into the transport class, recorded only after the backoff.

`TestPush`, the existing step test of `PushRebaseFree`'s error classes (integration, package `gitrepo_test`), gains a step over a local bare whose `pre-receive` hook prints GitHub's 403 wording and exits non-zero, which asserts `errors.Is(err, ErrPushAuthRefused)`, not `ErrPushRejected` and not `ErrPushRetriesExhausted`, and that the hook ran once.

**ImpactSummary:** `PushRebaseFree`'s callers all live in fabricengine; today they treat the new error as an unclassified push error, and card 9 gives it its class.

**Commit:** 8: gitrepo: an auth refusal of a push is its own class
