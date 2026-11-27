```yaml
lens: code-affected-set
outcome: reported
```

**Per change — set found, covered or not**

1. Guard admits `add`/`remove` for a docs-run darn writer (`internal/fabriccli/roleguard.go` `benchPairVerb`, `sandboxedRoleRefusal`)
   - `internal/fabriccli/doc.go` package doc — updated. Covered.
   - roleguard file header — updated. Covered.
   - `TestSandboxedRoleGuard` — new subtest. Covered.
   - `contracts/stencils/webster/webster-body-implementer.md:107-113` — describes the webster implementer, not a darn writer; still true. Covered.
   - `contracts/specs/refusal-spec.md:21` (task-session refusal's "the orch runs it" trailing clause) — `taskSessionRefusal` keeps that clause and the bench clause is appended after it; a conditional additional way forward does not contradict the spec. Covered.
   - Shell-guard table / settings denies — none mention `lym fabric add`/`remove`, so the new way forward names no denied step (PATTERN-refusal-way-forward). Covered.
   - PATTERN-fabric-boundary (no board/IDE/shuttle/heddle knowledge) — new imports are `shedrun` and `runprofile`, neither of those modules; the board read was dropped in 53987e2. Covered.
   - Hard-coded verb/flag literals (`"add"`, `"remove"`, `"parent"`, `"adopt-local"`) — same form as existing `mergeTaskVerb`/`readerVerb` code at `roleguard.go:245-264`. Covered.
   - `docs/research/**` reports saying a task session cannot run `lym fabric add` — measurement reports, exempt from stale-specifics. Covered.
2. Detached push of a bench pair passes (`injectedPathsAreOwn`, `isBenchPairPath`) — `fabriccli/doc.go` and roleguard header updated; push rows in the subtest. Covered.
3. `RemoveBenchPairs` and Worktree-Teardown wiring (`internal/pairteardown/leftover.go`, `internal/battencli/wire.go`)
   - `internal/pairteardown/doc.go` "Leftover pairs" — updated. Covered.
   - `internal/battenshed/doc.go` — updated. Covered.
   - `pattern/PATTERN-pair-remote.md` "Pair teardown" — not updated; see F1.
   - Callers — only wire.go (grep of `pairteardown` callers). Covered.
4. `requireStillOrphan` refactored into `runNamingReason` — error wording changed ("read the board before removing the pair %q" and the reason text); the only test asserting it (`leftover_integration_test.go:118`) expects "a batten run directory names it", which the new reason string still produces. No other Go test asserts the old wording. Covered.
5. `darnengine.IsWriter`, `IsMergeTaskWriter` built on it — behaviour unchanged; doc updated; reached from production through roleguard. Covered.
6. `runprofile.DocsProfile`, `retroengine.TaskProfile` uses it — value unchanged; no other readers to update. Covered.
7. `shedrun.ReadRunUsageRecord` — parses through the existing `ParseRunUsage`; PATTERN-sole-parsers lists no usage-directory parser (grep found no match); file header updated. Covered.
8. Commit messages — c04363a claims the guard checks "a slug … that no board entry names" and the "task entry" profile; 53987e2 supersedes both. No `Pattern-Departure:` trailers; no declared PATTERN changes. Covered.

**Findings**

- F1 · LOW · consistency · `internal/pairteardown/leftover.go` (`RemoveBenchPairs`, the `td.Run` call)
  Summary: `pattern/PATTERN-pair-remote.md` "Pair teardown" states the driver checks are skipped only for a pair that is paused, ended, owned by another machine, or done locally with batten no longer running, and lists the dobby's teardown routes; the new route passes `Verdict: fabricengine.EntryEnded` for a bench pair that has no entry and may hold a live driver, skipping the driver checks and ending its live processes, and the background file is not updated to name this route or this skip. Precedent: `RemoveOrphanPair` passes the same verdict, but only after `requireStillOrphan` confirms no live process.
  Evidence: `pattern/PATTERN-pair-remote.md` "## Pair teardown" opening paragraph ("Its driver checks are skipped for a pair that is paused, ended or owned by another machine…") and the dobby bullet; diff `leftover.go` new line 549; `internal/pairteardown/teardown.go:70-72` (`Verdict` doc).
  Fix: add the bench-pair route to that section of the background, or not forge the verdict.

Tool calls:
1. Grep of shell-guard table, old orphan error strings, PATTERN-sole-parsers usage entries, pairteardown doc and roleguard verb literals — context lacked the guard rows, the test assertions on the changed error text, and the sole-parser list.
2. Grep of settings/docs for `lym fabric` denies and task-session refusal statements — context lacked deny tables and rule docs describing the guard.
3. Read `contracts/specs/refusal-spec.md` lines 10-40 — the prior grep pointed to its role-guarded fabric clause.
4. Read `taskSessionRefusal` and grep production uses of the orch clause — to check how the appended bench clause fits the spec's required wording.
