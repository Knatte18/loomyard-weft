---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/10-tmuxkit-socket-check.md
    summary: checkSockets runs after Main's sweep, which kills every server and removes every socket file, so D6's runtime check can never fire
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/06-tmuxkit-package-server.md
    summary: the tmux-tier classification covers only reedengine and reedcli; hub- and standalone-keyed reed servers in eight other packages stay unregistered, so D4's hermetic non-login server never reaches them and D6's tagged check fails them
  - id: F3
    severity: BLOCKING
    class: decision
    location: _lyx/plan/00-overview.md
    summary: Shared Decision reed-exit-empty-probe changes production reedengine against D4's explicit Reed is unchanged and Scope Out; the plan licenses its own deviation instead of getting the record amended
  - id: F4
    severity: BLOCKING
    class: scope
    location: _lyx/plan/23-dotgit-leaf.md
    summary: the PATTERN overview cap amendment is carried by no card; cards 23, 25 and 32 each grow PATTERN.md, now 16380 of 16384 bytes, with no offset and no 16200-byte bound
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-tmuxkit-hermetic-server.md
    summary: the exit-empty early return's stated bound is false, since production reed servers read the operator's ~/.tmux.conf, which may set exit-empty off and so disable the stale-holder reap
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/19-reclassify-fabricengine-reconcile-to-worktree.md
    summary: warplayout_test.go is classified read-only as two in-process reads, but lyxcwd.ResolveWorktree spawns git rev-parse until card 24, and a spawned git is a write under D11 and the plan's own classification
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/plan/13-gatecli-prebuild.md
    summary: a tagged gate run whose root has no cmd/lyx neither sets nor strips the prebuilt variable, breaking D7's bound that lyx gate test always sets or strips it, and the harness's untagged strip is unstated
  - id: F8
    severity: MEDIUM
    class: scope
    location: _lyx/plan/12-lyxbin-run.md
    summary: the package comments of lyxbin and tmuxkit state exemption bounds that cards 12 and 7 make false, and no card targets them, against the record's docs-in-the-same-commit constraint
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/00-overview.md
    summary: Shared Decision rev-parse-survivors names verifytree and impactset as out-of-scope survivors but omits gitrepo's MergeHeadPresent rev-parse --verify --quiet MERGE_HEAD ref read
---

### [BLOCKING] D6's runtime socket check runs after the sweep has removed every socket

**Location:** `_lyx/plan/10-tmuxkit-socket-check.md` (with `_lyx/plan/07-tmuxkit-private-tmpdir-leftovers.md`, which keeps the sweep ahead of `afterRun`)

**Issue:** Card 7 has `Main` keep "its socket sweep and then returns `afterRun(...)`", and card 10 has `afterRun` run `checkSockets` "after the sweep, before the leftover scan".
`sweep` in `internal/testkit/tmuxkit/tmuxkit.go` runs `kill-server` on every socket under the kit's per-user directory and then calls `removeDeadSocket`, which removes each socket once it refuses connections.
A socket whose server had already exited is removed the same way.
By the time `checkSockets` lists the directory, it is empty in every normal run.
So the acceptance criteria "A package built without `tmux` or `llm` whose test starts a tmux server fails after `m.Run`, naming the socket; a `tmux` build with an unregistered socket fails the same way" can never be met.
`TestCheckSockets` calls `checkSockets` in isolation over a hand-made socket, so it cannot catch the ordering.

**Fix:** In card 10, run `checkSockets` over the socket directory before the sweep and keep its result, then sweep, then fail the package on a non-empty result.
Pin the ordering through `afterRun` or a sweep-aware helper, not `checkSockets` alone.
For example, `TestCheckSockets` or a row beside it drives the post-run sequence over a listening socket and asserts the failure is reported although the sweep removed the file.

### [BLOCKING] Most tmux-tier reed servers are never registered with tmuxkit, so D4 and D6 do not reach them

**Location:** `_lyx/plan/06-tmuxkit-package-server.md` (the classification), with `_lyx/plan/22-reclassify-reed-orch-shuttle.md` and `_lyx/plan/10-tmuxkit-socket-check.md`

**Issue:** D4 makes `Socket` and `KillOnCleanup` the only route to a pre-started hermetic non-login server.
Its bound says a test that starts a server without registering its key runs the operator's config and login shells, and D6's check fails the package.
D5 says "The plan classifies each tmux-tier test by that criterion."
Card 6 classifies only reedengine's contract tests and says reedcli's smoke tests keep their key.
I grepped every package with a `tmux` or `llm` test file for `tmuxkit.KillOnCleanup` or `tmuxkit.Socket`.
These packages start reed servers but have no such call: `internal/loomcli`, `internal/shuttlecli`, `internal/seatengine` (`smoke_seats_test.go` builds a hub and wires real `reedengine`), `internal/treadleengine`, `internal/burlerengine`, `internal/orchcli`, `internal/webstercli` (`cli_tmux_test.go` boots a standalone reed session) and `internal/pairteardown` (`teardown_integration_test.go`, `tmux && !windows`, drives real reed sessions keyed by `reedengine.ServerName(h.Path)`).
Card 22's own register-free statement reflects the same gap: "A hub copied for a reed-driven test keeps the hub's own `reedengine.ServerName` key".
No card registers these keys.
Two consequences follow.
First, the goal's "Tier 3 panes run a non-login shell on a hermetic tmux server" is unmet for most of tier 3: those servers still read `~/.tmux.conf` and start login shells.
Second, once F1 is fixed, card 10's tagged branch fails each of these packages for an unregistered socket, first at Publish, because card 10's `Verify:` runs only the integration tier.

**Fix:** Extend card 6's classification, or add a card before card 10, so that every tmux- and llm-tier test that drives reed registers its key before reed boots.
For hub tests, that is `tmuxkit.KillOnCleanup(t, tmux, reedengine.ServerName(<hub path>))`.
For a standalone session, it is the standalone key.
List the test files per package.
Give card 10 a compile-only tmux `Verify:` line, `lyx gate test --tags tmux <packages> -- -run '^$'`, per the overview's own tagged-verification decision, and a run line for at least one reed-driving package other than reedcli.

### [BLOCKING] reed-exit-empty-probe adds production reed scope the record excludes

**Location:** `_lyx/plan/00-overview.md` (Shared Decision `reed-exit-empty-probe`), `_lyx/plan/05-tmuxkit-hermetic-server.md`

**Issue:** D4 says "Reed is unchanged: production strand panes stay tmux's default login shell."
The record's Scope In names no reedengine change, and Scope Out excludes "Pane start mode in production reed".
The premise is real.
`Engine.sessionlessSocketHolderPersists` in `internal/reedengine/lifecycle.go` returns true for a holder with no sessions after `staleSocketGrace` (5s).
`ensureServerAndSessionLocked` then reaps it, so a pre-started session-less server would be killed and replaced on every cold boot.
But the plan resolves the conflict by itself: it re-reads "Reed is unchanged" as "reed's pane start mode is unchanged" and calls this "the one deviation from the record's letter".
Under the rubric's fidelity rule, no card introduces scope the answer key does not license.
A production change to reed's stale-holder guard is exactly the kind of choice the record's "Added after Discussion" amendments exist for.
It is not one the plan writer may settle.

**Fix:** Take the conflict to the parent for a decision-record amendment that licenses the reed probe, with its bound (see F5).
Alternatively, rework card 5 to an approach inside the record's scope.
For example, a pre-start that leaves the server holding a placeholder session would keep reed's probe untouched, though it must not break tests that assert the session set.
Until one of these happens, the Shared Decision and card 5's reedengine targets are unlicensed.

### [BLOCKING] No card carries the PATTERN overview cap amendment

**Location:** `_lyx/plan/23-dotgit-leaf.md`, `_lyx/plan/25-gitrepo-geometry.md`, `_lyx/plan/32-removestrand-scan.md`

**Issue:** The record's amendment "PATTERN overview stays under its cap" requires that cards 23, 25 and 32 keep `PATTERN.md`'s overview at most 16200 bytes on the task branch.
An added name is offset by shorter wording, with detail moved to the background file.
`PATTERN.md` is 16380 bytes today, against `internal/pattern/overview_test.go`'s 16384-byte cap.
Card 23 adds `dotgit` to the `PATTERN-leaf-packages` line.
Card 25 rewrites the `PATTERN-gitrepo-client-boundary` line to name the `dotgit` geometry read.
Card 32 adds three package names, their roles and "(test)" to `PATTERN-shuttle-stop`.
A grep of the plan for `16200`, `16384`, the cap or a shortening finds nothing.
Card 23 alone pushes the file past 16384 and fails `TestOverview_RealFilePassesCheck`.
None of the cards says what wording it shortens.

**Fix:** In each of cards 23, 25 and 32, state that the card keeps `PATTERN.md` at most 16200 bytes, and name the overview line or lines it shortens and the background file that receives the moved detail.
The first of the three must also absorb today's 180-byte excess over 16200.
Card 23 should also say that the leaf entry keeps room for `fswatch` from `daemon-wakeups`, so a merge keeps both.

### [MEDIUM] The exit-empty early return rests on a false bound

**Location:** `_lyx/plan/05-tmuxkit-hermetic-server.md`

**Issue:** Card 5 bounds the new early return with "Production servers never set `exit-empty`, so no production boot takes the new branch."
Reed starts its production server with `new-session` and no `-f`.
That is why D4 needs tmuxkit's own config at all.
So a production server reads the operator's `~/.tmux.conf`, and an operator config with `set -g exit-empty off` makes every production holder that answers the query skip the stale-holder reap.
That reap is the guard against a holder that wedges a fresh boot: the psmux helper, a dying server.
The card makes the guard bypassable, and the bound it states for that is false.

**Fix:** Restate the bound in card 5's Intent: the branch is taken by any server whose config sets `exit-empty off`, including an operator's.
Narrow the probe to servers tmuxkit started, for example by checking a test-only marker option that tmuxkit's config sets, or say why skipping the reap for such operator servers is safe.
This rides on F3's amendment.

### [MEDIUM] warplayout_test.go is not read-only when card 19 classifies it

**Location:** `_lyx/plan/19-reclassify-fabricengine-reconcile-to-worktree.md`

**Issue:** Card 19 classifies `TestWarpLayoutFor_FastPathMatchesResolveWorktree` as the one read-only test because it compares `WarpLayoutForForTest` with `lyxcwd.ResolveWorktree` on the prime, "both in-process reads".
At card 19, `lyxcwd`'s `gitWorktreeRoot` still runs `gitexec.Run(rev-parse --show-toplevel)` (`internal/lyxcwd/lyxcwd.go:149`).
Card 24 removes that spawn later.
The test file's own header calls it "the spawning lyxcwd.ResolveWorktree fallback".
D11 and the overview's `hub-fixture-classification` decision count any spawned git subcommand as a write.
So under the plan's own rule, the test copies at card 19, or it is shared only after card 24.

**Fix:** Pick one of two options.
Classify `warplayout_test.go` as `CopyHub` of the `.` no-pair shape in card 19.
Or move its `SharedHub` move to a card after card 24, citing card 24's removal of the spawn.
Then update the overview decision's "Shared:" list to match.

### [MEDIUM] Card 13's skip branch lets an inherited prebuilt value into a gated run

**Location:** `_lyx/plan/13-gatecli-prebuild.md`

**Issue:** D7's bound rests on "A forged value never reaches a gated run, because `lyx gate test` always sets or strips the variable itself".
Card 13 adds an edge: "when `<root>/cmd/lyx` does not exist the build is skipped and nothing is exported".
On that tagged path the card neither sets nor strips, because only the untagged path calls `stripPrebuilt`.
An inherited value therefore reaches `go test`.
The Intent states no bound for this edge.
Separately, D7 says `cmd/testtiming` "does the same", and card 13 gives the harness only the tagged build, with no strip on its untagged runs.

**Fix:** Call `stripPrebuilt` on every path that does not export a freshly built value: untagged, and tagged with the build skipped.
Give the harness's `prebuildLyx` the same strip for an untagged run.
Add a `TestGateTest_Scenario` row for a tagged run in a root with no `cmd/lyx` that asserts the stand-in sees no variable.

### [MEDIUM] Package docs of lyxbin and tmuxkit are left stating false bounds

**Location:** `_lyx/plan/12-lyxbin-run.md`, `_lyx/plan/07-tmuxkit-private-tmpdir-leftovers.md`

**Issue:** The record's constraint is that docs land in the same commit, including the package docs of every changed package.
`internal/testkit/lyxbin/lyxbin.go`'s package comment says the kit "runs nothing but that `go build`".
Card 12 adds `Run`, which runs the built binary under group kill.
`internal/testkit/tmuxkit/tmuxkit.go`'s package comment bounds the exemption to "running the `tmux` binary against sockets under the kit's own directory or its own fixture keys".
Card 7 adds the `/proc` scan, killing leftovers, and the private `TMPDIR`.
Card 5 adds the config pre-start, and card 6 adds `PackageServer`.
Both cards update `pattern/PATTERN-testkit.md`, but no card names either package comment, so both stay false.
Card 14 likewise changes `rawGoCommandPattern` without saying that its doc comment's "the slot-inheritance variable or the strand-name variable" line gains the third variable.

**Fix:** In card 12, state that `lyxbin`'s package comment widens to "go build of ./cmd/lyx and running that binary under group kill".
In card 7, state that `tmuxkit`'s package comment records the private `TMPDIR`, the `/proc` scan and leftover kill, and the exported probes.
Have cards 5 and 6 add the pre-start and `PackageServer` sentences.
In card 14, say that the pattern's doc comment names the third variable.

### [LOW] The rev-parse survivors list misses one gitrepo ref read

**Location:** `_lyx/plan/00-overview.md` (Shared Decision `rev-parse-survivors`)

**Issue:** The decision lists the surviving `rev-parse` spawns and names `verifytree` and `impactset` as out of scope and untouched.
It does not mention `internal/gitrepo/merge.go`'s `MergeHeadPresent`, which still spawns `rev-parse --verify --quiet MERGE_HEAD`.
That is a ref-existence read in the very package D12 moves onto go-git and `dotgit`, and card 25 edits its sibling `MergeHeads`.
A reader cannot tell whether it was judged out of scope or missed.

**Fix:** Add `MergeHeadPresent` to the decision.
Either list it as untouched, with the reason that D12's ref-read move targets fabricengine's spawns, or move it onto a `readGoGit` read in card 26 with a parity row.

## Focus departures

None.
The focus items were settled as follows.
Fidelity: F2 through F4 and F7 are the gaps; D1–D3 sit in cards 1–4 and 33, D9 in card 15, D10 in card 12, D11 in cards 16–22, D12 in cards 23–30 and D13 in card 32.
The hooks and D8 amendments are carried by card 30's survivors and card 8.
The commondir amendment is carried by cards 23 and 24.
Attack surface: F5 and F7.
Cards 11 and 14 state their bounds.
Card order: the one dependency on a later card is F6 (card 19 on card 24).
The kits, lyxbin, hubforge and reclassify cards otherwise use only symbols created earlier.
Verify coverage: the plan-level section runs `./...` untagged and integration, vets `tmux` and `llm`, and follows the stencil's order.
The only nested module, `internal/planglyph/testdata/glyphchain`, is targeted by no card.
Granularity and Edit glyphs: the thirds in cards 17–19 and the spawn-site split in cards 28–30 are each fixture-only or behind-signature refactors that can be reviewed alone.
The Edit cards that name symbols list member glyphs, and no member is re-signed.
Test economy: each new test function (cards 1, 5, 6, 7, 10, 12, 16 and 32) covers a surface no existing test in its package covers.
The parity and tier-purity additions extend `TestParity`, `TestParity_UnbornHEAD`, `TestTierPurity_UntaggedTestsSpawnNothing` and `TestIsTierTagged_RecognizesKnownTagsList`.
