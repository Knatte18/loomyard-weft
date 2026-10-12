# Decision record — run-robustness

## Goal

A run record written by a newer `lyx` binary never halts or loses data in a long-lived process started on an older one from this binary on: every Go-written JSON run record is read with unknown fields accepted, and every rewrite keeps the fields its binary does not know.
A verify step that hung or was killed by a timeout or signal goes to the fixer as a failure and is never re-run unfixed, and a Go test binary that keeps a CPU core busy far below go test's 10-minute timeout is stopped with SIGQUIT, so its goroutine stacks reach the log early.

## Scope

In:

- `internal/state`: tolerant reads and unknown-key keeping on every write (D1–D3).
  This reaches every `state` caller, records outside a run included, such as fabric's indexes and reed's and orch's state.
- The two hand-rolled strict JSON decoders of run records: `internal/shedrun`'s seed decode and `internal/battencli`'s cleanup status judge (D4).
- `internal/websterengine`'s verify gate: no flake rerun after a hang (D5, D6).
- `internal/loomshed`'s verify gate: a halt reason after a watchdog quit names `busy_quit_sec` (D6).
- `internal/landingshed`'s verify gate: a halt reason after a watchdog quit names `busy_quit_sec` (D6).
- `lyx gate test`: a CPU-busy watchdog that SIGQUITs a Go test binary (D7, D8), with its CPU sampling in `internal/proc` and its budget in hub-wide `gate.yaml`.
- A PATTERN entry for D1–D4 (D9), the package docs of `state`, `gatecli`, `websterengine`, `loomshed`, `landingshed` and `proc`, the gate config template, and the comments in `shedrun` and `battencli` that describe the strict decodes.

Out:

- Agent-written contract files (webster's `outcome.yaml` and `report.yaml`, the arbiter's `verdict.yaml`, plan and recipe files): their strict decoding catches agent errors, and no binary rewrites them.
- Go-written YAML records: `yaml.v3` already decodes them tolerantly, and they are written once per attempt (see Auto-mode assumptions).
- `board.json`'s keeping: task `board-field-drop` owns it, and this task leaves its encoding unchanged.
- A process that is deadlocked without CPU use: go test's own `-timeout` still bounds it.
- A busy non-test process, such as a `lyx` child a test spawns, and a verify command that does not run through `lyx gate test`: SIGQUIT yields stacks only from a Go test binary, and only `lyx gate test` knows its child is go test.
  In a non-Go hub, such as a C# one (`dotnet-scoped-verify`), D5 sees only the exit-code markers and D7 never fires.
- Windows, which has no SIGQUIT (D8); `verify-watchdog` keeps the Windows process-tree kill.

## Decisions

### D1 — Run records are read with unknown fields accepted

Decision: no reader of a Go-written JSON run record refuses an unknown field.
`state.ReadJSONStrict` drops `DisallowUnknownFields` and is renamed for what still sets it apart from `ReadJSON`: it creates no missing parent directory.
A malformed file is still `ErrDecode`.
Rationale: the batten halt (`decode failed: json: unknown field "routed_to"`) came from `ReadJSONStrict[shedengine.Status]`, which every standalone shed `status.json` reader in batten, loom and shed uses.

### D2 — Every `state` write keeps the keys its type does not know

Decision: `WriteJSON` and `UpdateJSON` read the file they replace under the write lock they already hold, and add to the new encoding each key of the old file that the record's Go type does not declare, with its raw JSON value.
Known keys come from the Go type: json tag names and field names, embedded structs' promoted fields, matched case-insensitively as `encoding/json` matches them.
The merge recurses through struct-typed fields, pointers, maps by key, and slices and arrays by index under D3's rule.
A type implementing `json.Marshaler` or `json.Unmarshaler`, a `json.RawMessage`, an interface value or a non-struct value is opaque: its new encoding is written whole.
A key present in the new encoding is never overwritten by the old file.
An old file that is absent or does not parse adds nothing; `UpdateJSON` still aborts on a decode failure before `mutate` runs, as today.
Rationale: one generic mechanism covers every record written through `internal/state` at once, and the incident's field sits in a history entry, so the merge must reach nested objects.
Bound: the keys that pass through are only those already in the file being replaced; no caller can add or change a key through this path.
Each write costs one extra read of that file under the lock it already holds.
A kept value is carried unchanged while known fields beside it change, so it can go stale (Open risks).
`board.json` is unaffected: on `main` since `board-field-drop` landed (`78e31aa06`), its write type `storeFile` implements `json.Marshaler`, so it is opaque and written whole, and its own keeping and retired-key consumption stand.

### D3 — An array element keeps its unknown keys only while its known fields are unchanged

Decision: an element of an array or slice at index i keeps the old element i's unknown keys only when the old element's known fields, decoded and re-encoded by this binary, encode identically to the new element i.
An element updated in place, removed, or moved to another index loses them.
Rationale: an appended or untouched entry, such as a shed history entry, keeps its keys, and a key is never carried onto a different element.
A map value needs no such check: its key is its identity, so it is merged like a struct field.
Bound: an in-place update of an array element by an older binary drops that element's unknown keys, the same loss as before this change, for that element only.
A map value replaced under the same map key, deleted and re-created or overwritten wholesale, keeps the old value's unknown keys.

### D4 — The two hand-rolled strict decoders become tolerant

Decision: `shedrun`'s seed decode and `battencli`'s cleanup status judge stop calling `DisallowUnknownFields`.
`WriteSeed` keeps its agreement check on recipe, driver and params, and an agreeing seed is not rewritten, so an unknown key in an existing seed survives.
`seedWire` and its `LegacyParent` field, which exist only so the strict decode accepts the retired `parent` key, are removed.
Rationale: both decode run records another binary may have written.

### D5 — The webster verify gate does not re-run a hung step

Decision: the verify gate's flake rerun, which runs the verify command a second time inside the same gate evaluation after any failure that was not the 60-minute `verifytree` timeout, is skipped when the first run's result is a hang.
A hang is any of: `res.TimedOut`, as today; an exit code of 128 or above; an exit code of -1 with an empty `Detail`, which `verifyrun` returns for a shell killed by a signal; or a log carrying a hang marker: go test's `panic: test timed out after`, the Go runtime's `SIGQUIT: quit` dump header, go test reporting a test binary ended by a signal, or the watchdog line of D7.
The hung result fails the gate at once and goes to the fixer through the normal re-prompt, spending one attempt; it is never recorded as flaky.
A rerun that itself hangs is reported the same way.
Rationale: `board-field-drop`'s verify-2 was this rerun, which counts only `res.TimedOut` as a hang, so a test binary SIGQUIT by the orch read as an ordinary failure.
Bound: the markers are matched as text anywhere in the log, so a test that prints one, or a test binary the OOM killer ends, also reads as a hang; all a match can skip is the one rerun, and a flaky failure in the same run reaches the fixer without its retry.
Any exit code of 128 or above reads as a hang whether or not a signal caused it, such as git's fatal 128 or a tool's 255 in a hub's verify command; all it skips is the one rerun.

### D6 — The gate report names the hang

Decision: the verify gate report records that the run hung, with the marker or exit code it matched, the failing identities and the log path; the fixer's findings and the stuck reason say the step was not re-run.
When a timeout or SIGQUIT marker matched, they say the step hung and the stacks are in the log; when only an exit code of 128 or above matched, they say the step was killed or exited with that code, and name it.
When the matched marker is D7's watchdog line, the report, the findings and the stuck reason also name the window and say that a package that is CPU-bound but healthy needs `busy_quit_sec` in the hub's `gate.yaml` raised through `lyx config gate` from the prime.
The stuck reason's row in `contracts/specs/refusal-spec.md` ("verify gate exhausted") gains the hang case in the same commit, per `PATTERN-refusal-way-forward`, with raising `busy_quit_sec` through `lyx config gate` from the prime as the way forward for a watchdog quit.
The watchdog fires in every verify command run through `lyx gate test`, not only webster's, so every other halt that reports a failed verify run through `verifytree.Verify` does the same: the loom verify gate (`internal/loomshed`) and the landing verify gate (`internal/landingshed`), when the failed run's log carries D7's watchdog line, name the window in their halt reason and raising `busy_quit_sec` through `lyx config gate` from the prime as its way forward.
Their refusal-spec rows (`Webster-Burler-Fix verify gate exhausted`, `Darn-Write verify gate exhausted`, and the landing rows for a failed verify) gain that way forward in the same commit.
These gates run no flake rerun, so D5 changes nothing in them.
Rationale: the fixer needs to look for a livelock or deadlock, not a flake;
and a watchdog quit of a healthy package has no code fix, so a halt that names only fixing the tests names a way forward that does not fit.

### D7 — `lyx gate test` quits a Go test binary that keeps a core busy

Decision: while go test runs, `lyx gate test` samples the CPU time of every process in its child tree whose executable path, read from `/proc/<pid>/exe` or `cmdline` and never from the 15-character `comm` in `stat`, ends in `.test`, every 5 seconds.
A test binary that used at least 90 percent of one core over every sample period for a continuous window of `gate.yaml`'s new key `busy_quit_sec` gets one SIGQUIT, and the verb prints one line to its stderr naming the binary, its pid and the window.
A sample under that threshold restarts that binary's window.
The threshold sits a fixed 10 percent below one core because a single spinning goroutine uses at most one core, and clock-tick granularity in `/proc/<pid>/stat` puts its samples just under one core.
An absent key reads as the default, 120 seconds, so a hub not yet reconciled still has the watchdog; an explicit 0 turns it off, and a negative value is refused like the file's other invalid values.
The window counts whole samples, so a budget shorter than the sample period quits after the first busy sample.
A run whose go test flags carry `-bench` or `-fuzz` runs no watchdog, since those keep a core busy by intent.
`lyx gate test` sets a new environment variable, carrying its own pid, only in the environment of the go test it spawns.
A `lyx gate test` that finds that variable naming a live ancestor of its own process, walked through the parent pids `internal/proc` already reads from `/proc/<pid>/stat`, runs no watchdog of its own, since it runs under an enclosing `lyx gate test` whose tree already holds its test binaries; so no binary gets two SIGQUITs.
An inherited slot does not turn the watchdog off: the slot holder is not always an outer `lyx gate test`, since `verifytree.Verify` holds a slot and passes it to the verify command, so a `lyx gate test` run as a verify command runs its watchdog.
The key is hub-wide only, not in the gate module's `LocalKeys`.
The sample period is a field a test injects; production uses 5 seconds.
The SIGQUIT and the sampler's start and failure are logged through `internal/logger`.
The Linux sampling reads `/proc` in `internal/proc`, beside its existing stat reader; the loop lives in `internal/gatecli`.
Rationale: a SIGQUIT makes the Go runtime print every goroutine's stack and exit, so go test reports that package as failed with the stacks, minutes before its own timeout and without holding the CPU for them.
The 10 percent margin covers tick granularity only;
a spinning goroutine starved below it by CPU contention falls to go test's `-timeout`, the case Open risks names for a livelock below 90 percent.
Bound: the watchdog signals only `.test` processes in the verb's own child tree, once each, and covers only verify commands that run through `lyx gate test`; within those, it runs in the outermost `lyx gate test`, whether that one acquired its slot or inherited it.
A variable naming no live ancestor, such as one a tmux server a test started carried into its panes after daemonizing out of the outer run's tree, or one set by hand, is ignored, so the skip holds only where an outer watchdog sees the process.
A CPU-bound test that is not hung but runs over the window is quit and fails, and the operator raises the key.
A run carrying `-bench` or `-fuzz` is bounded only by go test's own `-timeout`, as before this change.

### D8 — The watchdog is Linux-only

Decision: on other platforms the sampler is a no-op that logs once that no watchdog applies.
Rationale: Windows has no SIGQUIT; the operator's machines run Linux or WSL.

### D9 — A PATTERN entry pins tolerant run records

Decision: `PATTERN.md` gains a one-line entry, with its detail in a new `pattern/` background file: no reader of a Go-written JSON record refuses an unknown key, and a rewrite of one goes through `internal/state`, which keeps them, or through a type that keeps them itself as `board.json`'s does.
A read-only decode outside `state`, such as batten cleanup's judge of committed bytes, is allowed.
A `go test` scan fails on any production `DisallowUnknownFields` call.
The scan covers only `encoding/json`'s `DisallowUnknownFields`; `yaml.v3`'s `KnownFields`, which the strict agent-written contract decoders use, is outside it, and a future agent-written JSON contract that needs strictness validates its keys some other way.
The line fits under the overview's headroom line that `internal/patternlint` checks.
Rationale: the failure class recurs with every landing that adds a field, so the rule must survive future readers.

### Added after Discussion (parent, 2026-10-12): Plan against main with board-field-drop

Decision: The parent merged main (now at 113593990, which contains 78e31aa06) into run-robustness during Plan-Write. The plan pins board.json's encoding against the merged tree directly.

Rationale: D2 and the acceptance criterion rely on 78e31aa06's storeFile Marshaler; merging now lets cards name glyphs that exist.

## Constraints

- `PATTERN-shed-run-directory`: `shedrun` stays the sole declarer of run-directory paths and `Seed`; D4 changes only its decode.
- `PATTERN-daemon-wakeups`: D7's sampler lives as long as go test, so it counts as an unbounded wait; it wakes every 5 seconds, spawns nothing, and its file joins the scan's declared list.
- `PATTERN-hub-wide-config` and `PATTERN-config-layers`: `busy_quit_sec` is added to the hub-wide gate template and read through the existing gate config loader; an existing hub gets it through the binary-change reconcile.
- `PATTERN-spawn-observability`: the SIGQUIT is logged.
- `PATTERN-test-speed` and `PATTERN-test-economy`: unit tests inject the sampler and clock; one tagged integration test drives a real spinning test binary with a 1-second budget and an injected 1-second sample period.
- `PATTERN-gate-slot`: the watchdog runs inside the held slot and does not change slot handling.
- A record written by this binary stays readable by older binaries that read tolerantly; kept keys are ordinary JSON object members.

## Auto-mode assumptions

- Go-written YAML records (verifytree markers and records, webster's verify-gate report, shed adapters' records) are written once per attempt and never rewritten whole by a different process, so they need no keeping.
- At least 90 percent of one core sustained for 120 seconds separates a hung test binary from a healthy one in this repository's suites.

## Open risks

- The guarantee starts with this binary: a batten or loom process started on an older binary still refuses or drops new fields until it is restarted after this lands.
- A kept key goes stale when it depends on known fields an older binary changes, such as `lyx-halt`'s `paused_since` kept across an older binary's resume.
- A `WriteJSON` that means a fresh record over a file an earlier session left at the same path carries that file's unknown keys, and so does a map value replaced under the same map key (D3).
- A healthy package that keeps a core busy past the window is quit, D5 reads that as a hang, and the fixer cannot fix it in code, so the gate can run out of attempts and stick the run; the operator raises `busy_quit_sec`.
- A test binary that livelocks below 90 percent of one core, spins on a goroutine that CPU contention starves below it, or deadlocks without CPU, still runs to go test's `-timeout`.
- go test's wording for a test binary ended by a signal is matched by text, so a Go release that rewords it loses that marker; the exit code and the other markers still apply.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d6-watchdog-false-hang-no-way-forward (class design, severity BLOCKING, open in rounds 1, 2)
  - d7-busy-threshold-single-core-ceiling (class design, severity MEDIUM, open in rounds 1, 2)
  - d7-enclosing-variable-trusted-unchecked (class design, severity MEDIUM, open in rounds 2)
  - d7-integration-criterion-satisfied-by-outer-watchdog (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - finalize-verify-failed-no-refusal-row (class scope, severity BLOCKING, open in rounds 2)
  - loomshed-round-record-rewrite-drops-unknown-keys (class scope, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- A shed `status.json` carrying an unknown top-level key and an unknown key in one history entry reads through every reader D1 renames, and round-trips through `UpdateJSON` and `WriteJSON` with both keys and their JSON values kept.
- An `UpdateJSON` that appends a history entry keeps the existing entries' unknown keys; one that changes a known field of an entry drops that entry's unknown keys and keeps the others'.
- A map-valued field keeps a value's unknown keys under the same map key.
- A key known only by case variant is decoded into its field and not kept, so the next write carries it once.
- A type owning its own JSON encoding and a `json.RawMessage` field are written exactly as the new value encodes them.
- A write over an absent file, and a `WriteJSON` over a malformed one, write the new value alone; `UpdateJSON` over a malformed file still aborts without writing.
- A seed with an unknown key decodes, and an agreeing `WriteSeed` over it leaves the file unchanged.
- Batten cleanup judges a done status that carries an unknown key as a candidate.
- A `board.json` write encodes exactly as it does on `main` at `78e31aa06`.
- The webster verify gate, after a first run whose log carries each hang marker in turn, or whose exit code is 128 or above, runs the verify command once, fails, records the hang and no flaky identities; an ordinary failure still runs it twice, and a rerun pass still records flaky identities.
- An ordinary first failure whose rerun hangs fails the gate with the hang recorded.
- A run that ends stuck after a hung verify carries a stuck reason naming the hang, matching its refusal-spec row; after a watchdog quit it also names the window and `busy_quit_sec` raised through `lyx config gate` from the prime, and after a bare exit code of 128 or above it names the code and claims no stacks.
- A failed verify whose log carries D7's watchdog line yields, in the loom verify gate and in the landing verify gate, a halt reason naming the window and `busy_quit_sec` raised through `lyx config gate` from the prime, matching its refusal-spec row.
- The watchdog, with an injected sampler and clock, quits a `.test` process busy for the whole window exactly once, quits one sampled at 92 percent of one core for the whole window, prints its line, sends no second SIGQUIT while it stays busy, restarts the window after an idle sample, finds a `.test` binary whose name is longer than 15 characters, ignores processes not ending in `.test`, and does nothing with the key at 0, with `-bench` or `-fuzz` flags, or under an enclosing `lyx gate test`, and runs it when the variable names a pid that is not its ancestor.
- A `lyx gate test` started in an inherited slot with no enclosing `lyx gate test` runs the watchdog, and one started under another `lyx gate test` does not.
- On a platform other than Linux the watchdog does nothing and logs once.
- Under `lyx gate test` on Linux, a test that spins several goroutines, at least two, with `busy_quit_sec: 1` ends with goroutine stacks and the watchdog line in the output and a failing exit code, well before go test's timeout.
  The test removes D7's enclosing-gate-test variable from the inner `lyx gate test`'s environment and asserts the watchdog line carries the inner run's 1-second window, so the outer run's watchdog cannot satisfy it.
- The PATTERN scan fails on a production `DisallowUnknownFields` call.
- A `gate.yaml` with `busy_quit_sec: 0` loads with the watchdog off, one without the key loads with 120 seconds, and one with a negative value is refused.
- The package docs of `state`, `gatecli` and `websterengine` and the gate template describe D1–D3, D7–D8 and D5–D6, and those of `loomshed` and `landingshed` describe D6's watchdog halt reason.

## Notes for the plan writer

- The flake rerun is the second `s.verify(site, command)` in `internal/websterengine/verifygate.go`; `failTimedOut` there is the existing hang path.
- `internal/proc/proc_linux.go` already reads `/proc/<pid>/stat` through `statFieldsAfterName`, which gives CPU ticks and parent pids but only the truncated `comm` name (D7).
- `internal/parentreview/store.go` reads one record with the standard library while it holds the lock; confirm it decodes tolerantly and writes through `state`.
- `gate.yaml` is decoded with plain `yaml.Unmarshal`, so an older binary tolerates the new key.
- `cmd/lyx/testdata/test-budget.yaml` may need rows for `internal/state` and `internal/gatecli`.
