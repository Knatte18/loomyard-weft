```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced each change in the diff to the goal points of the decision record.

**Decisions.** The record has no `Added after Discussion` entries, so the goal points are the only decisions.
- **G1:** treat a catch-up fetch on an unreachable origin as a transient network stop, so the dobby retries the start on its own backoff.
- **G2:** report it only past the bound, as the pushes do.

**Per change:**
- **C1** — `firstPassExited` reads `PassExitOriginWait` as owing the catch-up, not as a refused start. Traces to G1. Files: `internal/dobbyengine/starts.go:343-363`, plus `doc.go:73-74`.
- **C2** — the `catchUpOwed` guard in `startReady`. Forced by C1: without it, an exit read earlier in the tick restarts the task in the same tick. File: `starts.go:92-96`.
- **C3** — `CatchUpWait.Since`, `waitingSince`, and `Since` carried over in `catchUpFailed`. Traces to G2: the push summary needs a wait duration measured against the bound. File: `catchupwait.go:36-46, 84-88`.
- **C4** — the push summary lists the dobby's catch-up waiting on origin as a network wait. Traces to G2. Files: `pushsummary.go:36, 95-99`, plus `doc.go:269-271, 278-279`.
- **C5** — the `doc.go` edits. Forced by C1, C3 and C4.
- **C6** — new subtests in `starts_test.go` and `pushsummary_test.go`. Forced by C1/C2 and C3/C4.

**Per decision:**
- **G1** is built by C1 and C2.
- **G2** is built by C3 and C4, only partly (F2 below).

**Findings:**

1. **F1 — MEDIUM, class `scope`.** Location: `internal/dobbyengine/starts.go:362`.
   - Summary: the new branch covers more than an unreachable origin. It also treats a first pass stopped by a refused registration, or by any other catch-up failure, as no refused start. Those starts lose their refused-start notice and their backoff, and the goal does not ask for that.
   - Evidence:
     - G1 names only "a catch-up fetch on an unreachable origin" (`_lym/discussion/decision-record.md:11`).
     - `passExitCode` maps every `catchUpRefusal` to `PassExitOriginWait` (`internal/battencli/drive.go:138-139`).
     - `catchUpFailure` builds that refusal for an unreachable origin, a `RegistrationError` and the default "the catch-up failed" case alike (`internal/battencli/catchup.go:64-80`).
     - The change description admits the widening as its own design decision (`_lym/landing/summary.md`, "Design decisions", first bullet). No parent entry licenses it.

2. **F2 — MEDIUM, class `scope`.** Location: `internal/dobbyengine/pushsummary.go:95`.
   - Summary: G2 is built only for the prime's dobby. A beam dobby's catch-up waiting on origin lives in its own state file, and no summary reads that file. So a beam's start waiting on origin is never reported past the bound unless the prime's own catch-up also happens to be owed.
   - Evidence:
     - The summary reads only `d.geom.StatePath` (`pushsummary.go:86`).
     - It runs only where `PushSummary.Ledger` is wired (`pushsummary.go:40`).
     - The prime's catch-up is owed only on a sleep, a dobby start, a hand-over, or a pass exit of its own (`loop.go:510, 712`; `catchupwait.go:173`; `owned.go:21`; `passes.go:468`; `starts.go:363`).

Tool calls: none.
