```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to a goal point of the board entry, and each goal point to the changes that build it.

**Goal points (board entry under `## Goal`; the record has no `Added after Discussion` entries)**
- G1 (#193 Expected): a transient network failure on a git fetch, push or ls-remote is retried with backoff, for a bounded time, before the run stops.
- G2 (#193 Expected): only a failure that outlasts the retries reaches the parent, as a notice naming the failing remote operation, not as a question.
- G3 (#203 Expected): network calls to origin retry with backoff over a minute or more before a run halts.
- G4 (#203 Expected): a stop caused by a transient network error is retried by dobby, not handed to the parent as a question.

**Per change**
- C1, `internal/gitexec/transport.go` new `transportPhrases`, its doc comment, and the `transport_test.go` rows (the ls-remote column, plus 403 and 404 kept as refusals). Traces to G1 and G3: dropped connections and gateway errors are now classified as transport failures, so the existing retry paths and the new driver rows apply to them.
- C2, `internal/sheddriver/driverules.go` (`networkRestepWaits`, `networkRestepSpan`, the `networkFailure` restep row, `driveDecision.Wait`), `drive.go` (`Deps.Wait`, `goDriver.wait`, the wait in `settleStop`), the `doc.go` rule list, and the `driverules_test.go` and `drive_test.go` rows. Traces to G1, G3 and G4: re-steps after 15 s, 30 s and 1 min, 1m45s in total.
- C3, `driverules.go` hand-back row with `NetworkNotice`, and `drive.go` `handBackStop` and `networkNotice`, with their tests. Traces to G2.
- No change traces to nothing. No unrequested scope.

**Per goal point**
- G1: C1 and C2, but only for failures that surface as a shed step's error stop. The Finalize landing fetch and the pre-run and catch-up ls-remote are not covered (F1, F2).
- G2: C3, with the same limit: the landing halt and the catch-up refusal do not get the notice.
- G3: C2, same gaps as G1.
- G4: C2. The driver re-steps the run before any parent involvement.

**Findings**

1. MEDIUM · scope · `internal/sheddriver/driverules.go:271`
   The entry's Finalize case still escalates with no retry: "blocked at Finalize with a landing stop … the parent's upstream could not be fetched".
   - `landingshed/finalize_pr.go:524-525` halts with `LandingStopPrefix` when `ReadParentTips` fails.
   - The stop is a blocked hand-back stop. `decideDrive`'s landing-halt row at `driverules.go:271-272` matches before the new network rows at 289-292, so the stop hands back at once.
   - It gets no wait, no re-step and no network notice.
   - The diff does not touch this path, and `_lym/landing/summary.md` does not list it under "Not covered".

2. MEDIUM · scope · `internal/battencli/catchup.go:66`
   The entry's third case ("stopped at the start of its run on `git ls-remote origin`") and #203's refused hand pass ("origin is unreachable") are not retried.
   - The catch-up and pre-run reconciles pass zero `ReconcileOptions{}`: `battencli/resumereconcile.go:33`, `battencli/readopt.go:142`, `beamcli/dobbydeps.go:456`.
   - With zero options, `attempts()` at `fabricengine/originreconcile.go:83-87` returns 1, so ls-remote runs once.
   - The refusal at `catchup.go:66` reaches the parent with no backoff.
   - The summary's "Not covered" section admits this, but nothing in the record licenses dropping it, which G1 and G3 ask for.

3. LOW · scope · `internal/fabricengine/pushanchored.go:41`
   #203's Seen says weft pushes of the board and of a records branch "failed the same way, each with its own notice".
   - The diff changes no push path or push notice. Only C1's reclassification reaches it indirectly, through `IsPushOwed` and `pushclassified.go`'s `IsTransportFailure` cases.
   - So whether G1's "push … retried with backoff" holds for weft pushes rests on the existing owed-push retry, which this diff neither extends nor verifies.

**Tool calls**
- `sed`/`grep` on `battencli/catchup.go`, `ErrOwnerOriginUnreachable` and `IsPushOwed`: to ground the catch-up refusal text and the weft push paths, which were not in my inherited context.
