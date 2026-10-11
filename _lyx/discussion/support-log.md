# Support log — gate-test-hygiene

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick recorded in the question ledger.

1. Read the board entry (four merged items) and the running `small-fixes-batch` entry, which may raise the same test-budget rows.
2. Read `internal/pattern` (Check, caps, doc), the overview test, the test-budget guard and its background in `PATTERN-test-speed`, Publish's failure record and its Webster-Burler way forward, the comment lint's verb and both gate sites, and the `/proc` readers in test files.
3. Found the overview at 16217 bytes against a 16384 cap, `gofmt -l .` listing six files (the board's `bootstrap_test.go` is already fixed), and a third `/proc` copy in `reedcli` the board did not name.
4. Overview: reasoned that any cap the same `go test` enforces also runs on the merged tree, so headroom needs a check bound to a branch's own growth; settled on a diff-scoped lint beside the comment lint (D1–D3), a one-time trim (D4), and a byte count in the hard-cap finding (D5).
5. Test budget: kept the format; the failure message names the value to set (D6).
6. gofmt: an in-process Tier 1 guard using `go/format`, with a message that leads to an Edit rather than `gofmt -w` (D7).
7. Probes: kit gains parent-pid and comm probes; local matchers stay local (D8).

## Rejected alternatives

- Lower the overview cap to 95% in `go test`: the same test runs on the merged tree, so the collision moves to the lower cap.
- A warning-only threshold in `go test`: `t.Log` output is invisible in a passing gate, so it is a target in prose again, which the board says does not stick.
- Make Publish's recorded verify failure bounce to Webster-Burler automatically: Publish would need the Webster-Review segment, whose Burler Done resets the episode count, so an unfixable failure could loop unbounded; it also changes routing for every post-merge failure, far beyond a hygiene task.
- A per-branch delta format for the test budget (per-slug files or delta lines): extra files that need periodic consolidation, and adjacent appended lines still conflict; the post-merge fix is already a one-line edit once the message names the value.
- Branch-unique markers on budget rows so concurrent raises conflict: the conflict session might pick one side anyway, and markers accumulate.
- Slack in budget rows: defeats the guard's purpose of a visible decision per raise.
- `gofmt -l` as a plan verify step: depends on each plan's verify text and a `gofmt` binary on `PATH`.
- Telling the reader to run `gofmt -w`: agents change files only with Edit or Write.
- A PATTERN entry for gofmt: toolchain convention, not a structural invariant, and it spends overview bytes.
- A guard banning `/proc` reads outside the kit: not asked for; one grep in acceptance covers this task.
- Leaving `reedcli`'s copies: they are the same duplicated parent-pid parser as `loomcli`'s, which the kit rule covers.
- Folding the headroom lint into `lint-comments`: the verb's name would no longer say what it checks.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): how to give the overview headroom that sticks? A headroom line at 95% of the cap, enforced on growth only by a diff-scoped lint at both comment-lint sites; hard cap unchanged in `go test`.
- Q2 (auto-pick, resolved): should Publish route a merge-only verify failure automatically? No; keep the existing manual way forward, make the fixes mechanical.
- Q3 (auto-pick, resolved): trim target for the overview? At most 15000 bytes, moving detail to background files, dropping no rule.
- Q4 (auto-pick, resolved): test budget fix? Message names the value to set; format unchanged.
- Q5 (auto-pick, resolved): where does the gofmt check live? Tier 1 guard in `cmd/lyx` via `go/format`, skipping `testdata`, failure naming file and line.
- Q6 (auto-pick, resolved): PATTERN entry for gofmt? No.
- Q7 (auto-pick, resolved): include `reedcli`'s `/proc` readers? Yes, same copy class.
- Q8 (auto-pick, resolved): name of the new verb? `lyx loom lint-pattern`, mirroring `lint-comments`'s base flag and envelope.
- Q9 (resolved, fact): the board's gofmt example `internal/loomcli/bootstrap_test.go` is already clean; six other files are flagged today, recomputed at implementation.
