```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. I traced every member of the set each change forces outside its own lines.

**Per change: the affected set, and whether it is covered**

1. **`IsTransportFailure` now counts a missing local repository path** (`internal/gitexec/transport.go:124`).
   - Production callers: `add.go:96` (`RetryOriginReadInTransit`, used at `add.go:296,306,530,603,630`), `pull.go:639`, `executor.go:182`, `pushanchored.go:459`, `originreconcile.go:236,260,262,625`, `pushclassified.go:214,367,647,760,807,813` and `shedtransient.go:34`, which feeds `sheddriver/driverules.go:312`, `landingshed/finalize.go:711` and `loomcli/halt.go:121`. Each now treats a missing path as transient or owed. No caller's code needs to change.
   - Covered by the diff's tests: push classification (`pushclasses`) and reconcile (`originreconcile`).
   - Fixtures the diff moved to an existing non-repository directory: `cleanupcli`, `add_remoteleftover`, `originreconcile`, `pushanchored` and `pushclasses`. Each still checks what its name says.
   - Fixtures the diff left on a missing path:
     - `add_rollback_adopt_test.go:271-273`: wrong, see F1.
     - `clone_singlerepo_integration_test.go:775/878`: its comment no longer fits, see F2.
     - `battencli/lifecycle_integration_test.go:869`, `stepTeardown_UnreachableRemote…`: covered. It means "unreachable", and the new class agrees with that.
     - `hubreconcile/ensure_integration_test.go:776`, `TestEnsure_UnreachableBoardRemote…`: covered, for the same reason.
2. **`gitexec` `doc.go` claim** (`doc.go:23-24`): it matches the code in the English locale. The locale bound belongs to the correctness lens.
3. **`IsTransportFailure`'s doc comment** (`transport.go:92-105`): it matches the code. The `Bound:` line is in the doc and the summary.
4. **Rule documents:**
   - `pattern/PATTERN-pair-remote.md:29` ("An owed push, on an unreachable origin … is retried in the pusher") stays consistent with the new class.
   - No spec under `_bolt/_lym/specs`, `docs/specs` or `pattern/` names `IsTransportFailure` or defines a transport failure, so none goes stale. No stencil changed.
5. **Commit messages and departures:**
   - `6870ad49f` carries a `Root-Cause:` trailer. It names the matcher's gap, which the diff removes. Covered.
   - No `Pattern-Departure:` or `Loosened-Check:` trailer is needed:
     - The reconcile row "an origin that is no repository" keeps `wantError: true` and gains an `Unreachable` assertion.
     - The `cleanupcli` assertions are unchanged.
     - The `other`-class push test keeps its assertions.
     None is loosened.
6. **Declared PATTERN changes:** the directive declares none, so nothing applies.
7. **The `//testtiming:keep` annotation** (`transport_integration_test.go:15`) does not name the test's new case, see F3.

**Findings**

- **F1 — MEDIUM — consistency** — `internal/fabricengine/add_rollback_adopt_test.go:269-273`
  `TestAdd_GitFailureCarriesGitsOwnReason` still points origin at a missing path (`no-such-remote.git`) and runs `Add` with `AddOptions{}`, which sets no `BackoffSleep`.
  - That failure is now a transport failure, so `RetryOriginReadInTransit` (`add.go:89-101`) retries it after real `time.Sleep` delays of 1 s, 2 s and 4 s (`add.go:81`), about 7 s or more per probe.
  - Real-time waiting breaks `pattern/PATTERN-test-speed.md:207`: a production interval a test would otherwise wait out is injected.
  - The test's comment says the path makes the probe fail "for a reason only git can state", but the test now exercises the retried transport path instead of an immediate refusal.
  - The diff moved every sibling fixture of this kind to an existing non-repository directory and missed this one.
  - Evidence: `add_rollback_adopt_test.go:269-273`, `add.go:81,89-101`.
- **F2 — LOW — consistency** — `internal/fabricengine/clone_singlerepo_integration_test.go:775,878`
  The `originUnreadable` field's comment says it points origin "at a path that holds no repository", but line 878 sets a missing path (`no-such-repository`).
  - Until now the two classed alike. Now a missing path is a transport failure and an existing directory is a refusal.
  - So the `OriginCannotBeRead` row tests a different class from the one its comment states.
  - Its "no failed-push record" assertion now checks the transport class.
- **F3 — NIT — consistency** — `internal/gitexec/transport_integration_test.go:15`
  The `//testtiming:keep` reason lists an unreachable remote, a removed local origin and a hook-rejected push. It omits the directory holding no repository, which the test now pins and the file comment on line 2 names.

Tool calls:
1. A grep of the specs, `pattern/PATTERN-pair-remote.md` and `pattern/PATTERN-test-speed.md`, for docs and rules that describe transport failures and test clocks; the manifest had read no `pattern/` background file.
2. A grep of the `RetryOriginReadInTransit` call sites in `add.go`, the battencli teardown step and the Bolt files, to see which `Add` paths now retry; the manifest held only `add.go:85-105`.
