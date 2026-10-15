# Card 1 — Lockless orch state reader

**Create:**
- `internal/orchengine#PeekState`
- `internal/orchengine#TestPeekState`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/orchengine#testPaths`

**Intent:**
Add `PeekState` to `internal/orchengine/state.go`, beside `LoadState`: it reads `p.StatePath` with `os.ReadFile` and decodes it with `encoding/json` into `State`, taking no lock and creating no directory or file, so the batten overview pane can read the orch state once per frame without creating `p.StateLockPath` or blocking the watcher (D2).
An absent file (`os.ErrNotExist`) returns a zero `State`, `false` and no error.
Any other read error, and a present file that does not decode, returns an error wrapped with an `orch: peek state:` prefix; the caller decides how to tolerate it.
Decoding through `json.Unmarshal` never refuses an unknown key (`PATTERN-tolerant-records`).
Unlike `LoadState`, an absent file is not turned into a zero state in phase idle: the overview draws no orch line for it, so the found flag carries the distinction.
`internal/state` writes the file atomically, so a lockless read never sees a torn file.
The no-lock half of the contract is structural: the body is `os.ReadFile` plus `json.Unmarshal` and calls nothing in `internal/state` or `internal/lock`; the test pins the no-create half.
Bound: the reader is read-only and changes no writer's locking; `LoadState`, `SaveState` and every watcher path keep their locks.
The new imports are stdlib only, inside `internal/orchengine`'s import allowlist.

`TestPeekState` in `internal/orchengine/state_test.go` (package `orchengine`), on its own `testPaths(t)`, runs these steps in order:
- the absent file returns found false and no error, and afterwards neither `p.Dir` nor `p.StateLockPath` exists;
- after `SaveState(p, want)` of a state with known tokens, a phase, a cycle count and `WatcherExit` set, remove `p.StateLockPath`; `PeekState` returns found true and a state equal to `want`, and `p.StateLockPath` still does not exist afterwards;
- the saved JSON with `"future_key":1` added, written with `os.WriteFile`, decodes with found true and the same state;
- `{not json` at `p.StatePath` returns a non-nil error;
- `p.StatePath` replaced by a directory makes `os.ReadFile` fail, and returns a non-nil error.
It is a new top-level test because `TestSaveLoadState_RoundTrip` pins the locked round trip, and `PeekState`'s failure contract is a behavior of its own; raise `internal/orchengine`'s row in `cmd/lyx/testdata/test-budget.yaml` from 143 to 144.

**ImpactSummary:** one new exported reader in `internal/orchengine` with no existing caller changed, its own new test, and one test-budget row raised by one.

**Commit:** 1: orchengine: lockless PeekState for the overview pane
