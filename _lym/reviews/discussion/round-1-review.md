---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:63-76 and :89-90 (D2 build steps and Bound, D3 Bound)
    summary: A build step that commits through fabric spawns a detached push that outlives the build and writes the template after the pack and fingerprint; neither D2 nor D3 bounds it, and the walk that would catch it is gone.
    fix: "In D2, add a bullet: a build spawns no process that outlives it; a step that runs a fabric verb spawning the detached push (`fabricengine.SpawnDetachedPush`, which honours only `FABRIC_SKIP_GIT` and `FABRIC_SKIP_PUSH`, not `gitkit.FixtureGitEnv`) passes the verb's skip-push option or stays in the test after the copy. Add that step to D7's list of steps that stay in the test. Extend D3's Bound to name this second writer and say the D2 rule is what bounds it. Add a D2 acceptance criterion: a build whose step would spawn the detached push leaves no child running when the template is packed."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:72-76 (D2 testing.TB bullet and Bound)
    summary: D2's Bound claims refusing Cleanup, Setenv and TempDir keeps every build inside the template root, but a build can still call os.MkdirTemp, os.Setenv, git config --global, or the Go 1.26 TB methods Chdir and ArtifactDir, and a path outside the root survives relocation into every copy.
    fix: "In D2's testing.TB bullet, add `Chdir` and `ArtifactDir` to the refused methods. Rewrite D2's Bound to state what the refusing TB does not catch, namely process-wide calls outside the TB (`os.Setenv`, `os.MkdirTemp`, `git config --global` into the binary's one hermetic global config) and a retained `Context()`, and what bounds them: name the bound, such as review of each registration in its D7 task, or a check after the build that refuses a template file or link naming a path outside the template root."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:35, :47, :211 (D1 and its reftable criterion)
    summary: The reftable fallback is stated for fabricengine.List alone, so the websterengine sibling read, which D1 also moves onto the dotgit reader, fails on a reftable repository where git worktree list answers today.
    fix: "In D1's Bound, state that the `internal/websterengine/gitwrap.go` worktree read falls back to `git worktree list --porcelain` on the reader's named reftable error, as `List` does, and that the porcelain path stays for that case in both callers. Extend the D1 reftable acceptance criterion to cover the websterengine sibling read."
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:201 (Open risks, git floor line)
    summary: The Open risks line leaves whether git 2.43.0, the floor, orders and marks worktree entries as D1 assumes to a later parity test, a deferred research question the writer's Step 2 says to settle now.
    fix: "Research `git-worktree` at 2.43.0 (its manual and the release notes back to the versions that introduced each behaviour) for: linked worktrees sorted by path, the `prunable <reason>` and `locked` lines in porcelain output, an entry with no `gitdir` file left out, and an entry with no `HEAD` file listed detached with the all-zero SHA. Record each answer in D1 with its source and version, and state any behaviour that differs at 2.43.0 and how the reader handles it. Then drop the Open risks line."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:141 and :148 (D7 battencli bullet, D8)
    summary: D8 moves TestNoticeSite in the battencli task, but TestNoticeSite and its fixture noticeHub sit in notices_integration_test.go, which the task's named fixture beamHub (beamguard_integration_test.go) never touches, so D8's own no-file-opened-only-to-move-a-test rule forbids the move.
    fix: "Either name `noticeHub` in D7's `internal/battencli` bullet beside `beamHub`, as a second fixture on the same beam, member and plain-pair start state with the shortname written in the build, so the task edits `internal/battencli/notices_integration_test.go`; or drop `TestNoticeSite` from D8 and from D9 task 11. Keep D8's sentence about which tests move consistent with whichever is chosen."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:97-98 and :220 (D4 and its criterion)
    summary: D4 and its criterion say every gitkit and hubforge initialiser writes five keys at today's values, but the bare initialisers write only core.fsmonitor, maintenance.auto and gc.auto, so the criterion either fails or forces new keys against keys and values stay as today.
    fix: "In D4, say the non-bare initialisers write the five keys and the bare initialisers write `core.fsmonitor`, `maintenance.auto` and `gc.auto`, each as today. Reword the D4 criterion to: a repository from each initialiser reads the keys that initialiser writes today, at today's values, through `git config --get`."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:74 and :132 (D2 board rule, D7 steps that stay in the test)
    summary: D2 fails a build that leaves the board's HEAD unequal to its upstream, but D7 keeps in the test only a step that leaves the board behind its upstream, so a step that leaves it ahead, such as hubforge.SeedFabricConfig's unpushed Bolt commit, is neither kept out of the build nor admitted to it.
    fix: "In D7, change the stay-in-the-test case from a step that leaves the board behind its upstream to a step that leaves the board's HEAD unequal to its upstream tip, ahead or behind, matching D2's rule."
---

### [MEDIUM] A build's detached push outlives the build

**Location:** `_lym/discussion/decision-record.md` D2 (lines 63-76) and D3's Bound (lines 89-90).

**Issue:** D7 moves builder steps over as they are.
A step that runs a fabric verb landing a commit spawns the detached push (`internal/fabricengine/commit.go`, `SpawnDetachedPush` in `internal/fabricengine/spawn.go:64`).
That push skips only under `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH`; the build-time mark `gitkit.FixtureGitEnv` does not stop it.
The child can write the template's bares, tracking refs and reflogs after the build function returns, past D3's pack and the fingerprint.
D2's HEAD-equals-upstream check then races it, and copies taken meanwhile get a torn tree that, with the per-copy walk removed, nothing catches.
D3's Bound names only a stray `SharedHub` write, so the focus question "is that the whole of what the walk no longer catches" is answered no.

### [MEDIUM] D2's refusing TB does not bound what it claims

**Location:** `_lym/discussion/decision-record.md` D2, the `testing.TB` bullet and the Bound (lines 72-76).

**Issue:** The record says refusing `Cleanup`, `Setenv` and `TempDir` means no build places a path outside the template root.
A build can still call `os.MkdirTemp`, `os.Setenv`, `git config --global` against the binary's one hermetic global config, and, under `go 1.26` in `go.mod`, `TB.Chdir` and `TB.ArtifactDir`, which the record does not refuse.
`findRootReference` in `internal/hubforge/template.go` checks only for the template root, so a path outside it reaches every copy unflagged.
This is an attack-surface gap in a new registration verb, which the rubric makes a finding.

### [MEDIUM] Reftable fallback covers List alone

**Location:** `_lym/discussion/decision-record.md` D1 (lines 35, 47) and the reftable criterion (line 211).

**Issue:** D1 moves both `fabricengine.List` and `otherWorktrees` in `internal/websterengine/gitwrap.go` onto the dotgit reader, but the Bound gives only `List` the porcelain fallback.
On a reftable repository the reader refuses with its named error, and the webster read, which needs paths alone, now fails where it works today.

### [MEDIUM] The git floor's worktree behaviour is left to a later test

**Location:** `_lym/discussion/decision-record.md` Open risks (line 201).

**Issue:** D1's parity rests on how git orders and marks worktree entries.
The record settles that for git 2.53.0 by experiment and leaves the floor version, 2.43.0 (`internal/toolfloor`), to "the D1 parity test against the installed git".
That is a fact the design rests on, handed downstream; the writer's Step 2 and the rubric's deferred-research rule require it researched from the manual and release notes now.

### [MEDIUM] TestNoticeSite's move has no file the battencli task edits

**Location:** `_lym/discussion/decision-record.md` D7's `internal/battencli` bullet (line 141) and D8 (line 148).

**Issue:** `beamHub` lives in `internal/battencli/beamguard_integration_test.go`.
`TestNoticeSite` and its own near-identical fixture `noticeHub` (beam `b`, member `m`, plain pair `p`, plus a shortname) live in `internal/battencli/notices_integration_test.go`.
D8 says no task opens a file only to move a test, so as written the battencli task cannot move `TestNoticeSite`.

### [MEDIUM] D4's five keys do not hold for the bare initialisers

**Location:** `_lym/discussion/decision-record.md` D4 (lines 97-98) and its criterion (line 220).

**Issue:** `initBareRemote` in `internal/gitkit/gitkit.go` and `initBareRepo` in `internal/hubforge/hub.go` write three keys, not `user.name` and `user.email`.
The criterion then either fails for bares or makes the plan add keys, contradicting "the keys and values stay as today".
Three lenses raised this; it is kept once.

### [LOW] D7's exemption names only a board behind its upstream

**Location:** `_lym/discussion/decision-record.md` D2 (line 74) and D7 (line 132).

**Issue:** `hubforge.SeedFabricConfig` commits through `Bolt.Commit`, which does not push, leaving the board ahead of its upstream.
D2 fails such a build, and D7 keeps in the test only a step that leaves the board behind, so a mover following D7 writes a build D2 fails.

## Focus verdicts that found no defect

- D1 and D5's porcelain fallback and old hook: the fallback is bounded for `List` (the webster gap is F3); the old hook in installed hubs is stated and still works, so not a defect.
- D2's duplicate-key and failed-build lookups: bounded as stated, not a defect.
- D3, a test writing to a copy's pack files: a copy's packs are copied files, not links into the template, so the write stays in the copy; not a defect.
- D2's "pays from its third use" and the 0.18 s and 17 ms figures: measured in the report (section 2), not left to an implementer. The Open risks line on concurrent builds is a load expectation D10 re-measures, not a fact the design rests on; not a defect.
- Board entry and task filing: no Decision edits this task's own entry; D10 files its re-measure as a note; D9's darn tasks are the split the entry asks for. Not a defect.
- Parent-decision class: D3 keeping the amend and push departs from the entry's lever wording, but for a recorded technical reason (the reflog `internal/fabricengine/reflog.go` reads) and through the Auto-mode assumptions channel; no other entry, run or landed commit contradicts it, so the writer's pick stands. D5's `internal/adoptcli` sites come from the report's item 8 row the entry cites; not a defect.
- Relocation: D1's evidence bullets are the evidence the writer's spec requires a fact to carry, and each Rationale keeps one headline figure; nothing found to move to `support-log.md`.
- Consistency of D9 against D5, D6 and D8 for `internal/hubreconcile` and `replay_integration_test.go`: held; the battencli break is F5.
