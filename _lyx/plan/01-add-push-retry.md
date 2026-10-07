# Card 1 — Retry a bare (failed) push in Add

**Create:**
- `internal/fabricengine/pushretry.go`
- `internal/fabricengine/pushretry_test.go`
- `internal/fabricengine/add_pushretry_integration_test.go`

**Edit:**
- `internal/fabricengine/topology.go`
- `internal/fabricengine/add.go`
- `internal/fabricengine/weftwiring.go`
- `internal/fabricengine/export_test.go`

**Intent:**
D1: `Add`'s warp push (step 11) and weft push (step 12, inside `pushWeftBranch`) go through one new fabricengine helper in `pushretry.go` that retries a transient server-side refusal.

The helper runs the plain, never-forced `git push -u origin <branch>` through `gitexec` in a given directory.
A classifier, a pure function over the push's stderr and the pushed branch, reports a push as transient only when stderr holds a `! [remote rejected] <ref> -> <ref> (failed)` line whose ref is the pushed branch and whose parenthesised reason is exactly `failed`.
Every other stderr line, `remote:` lines included, neither qualifies nor disqualifies: only the reason on the pushed ref's own line decides.
A transient refusal is retried up to 4 attempts in total, with exponential backoff and jitter of about 1s, 2s and 4s between them.
Every other failure (non-fast-forward, `pre-receive hook declined`, any other reason, an exec or auth error, a stderr with no `*gitexec.GitError`) returns on the first attempt, unwrapped as today.
Each retry is logged through `internal/logger` with the attempt number and the matching stderr line.
After the last attempt the last error is returned.

The helper's push runner and sleep (with its jitter) are injectable, so tests never wait out a real backoff (`PATTERN-test-speed`).
Hold them on an unexported field of `Topology`, nil in production meaning `gitexec.Run` and a real jittered sleep, and pass the same seam down to `pushWeftBranch`.
`export_test.go` gains a helper that sets that seam on a `Topology`, so a test injects per instance and stays parallel-safe.

What this lets through, and its bound: the retry repeats only a non-forced push of one commit to one ref, a no-op when an earlier attempt had already landed; it runs only inside `Add`, and at most 4 times, capping the delay at about 10 seconds.
A push retry adds no mutation entry of its own: the one accepted push records `branch_pushed` exactly as today (`PATTERN-mutation-record`).

Coverage:
- `pushretry_test.go` (untagged, `t.Parallel`): a table over the classifier: a bare `(failed)` line for the pushed ref is transient; the same line with `remote:` lines around it is transient; `(pre-receive hook declined)`, `(non-fast-forward)` and any other reason are not, with or without `remote:` lines; a `(failed)` line for a different ref is not.
  A second table drives the retry loop with a fake runner and a recording sleep: a refusal then success pushes twice and sleeps once; a persistent `(failed)` stops after 4 attempts and returns the last error; a hook-declined and a non-`failed` rejection are attempted once; the sleeps grow.
- `add_pushretry_integration_test.go` (`integration`, hubforge hub): an `Add` whose injected runner refuses the weft push once with a synthetic `(failed)` `*gitexec.GitError` and then delegates to `gitexec.Run` succeeds, with both worktrees, the warp branch and the weft branch present locally and on origin.

**ImpactSummary:** `Add`'s two push sites and its only weft-push helper change; no other verb pushes through them, and `NewTopology`'s callers see no signature change.
