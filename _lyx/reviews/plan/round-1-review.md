---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/05-run-bounded-lookup.md:67
    summary: The approve and reject unreadable-seed refusal names hand-editing seed.json as its way forward, which refusal-spec.md says never counts, and its named input is rejected upstream by loomCLI.resolveRecipe before the verb body runs
    fix: >-
      In card 5's Intent (lines 67-68), replace the way forward of the approveVerb and rejectVerb runStart refusal. The new message is `loom: approve: read the run's start: <err>; way forward: transient, re-run lyx loom approve`, and the same with `reject` in rejectVerb, matching the sibling read refusals at internal/loomcli/approve.go:64-72. Rewrite line 68 to say that `loomCLI.resolveRecipe` (internal/loomcli/arm.go:127-133) already refuses an unreadable or undecodable self seed in every approve and reject invocation's persistent pre-run (cli.go:230-252, arm.go:184), before the verb body runs. So the verbs' own branch is a propagated read failure, which refusal-spec.md exempts as transient, and D1's "the approve and reject verbs refuse with it" is carried by that upstream refusal. Keep the `SeedUnreadable` rows on line 79, but have them want "read the run's start" and "transient, re-run lyx loom approve" (and `reject`) in place of "fix the seed file".
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/05-run-bounded-lookup.md:63
    summary: wireLanding's seedRunStart decode-error branch, and the TestWireLanding_TellsLanguageSkills assertion over a `{not json` seed, guard a state resolveRecipe and the start bootstrap's ReadSeed reject first for the self run, and the propagated error names no way forward
    fix: In card 5's Intent line 63, state that for the self run the undecodable seed is refused before wireLanding by `loomCLI.resolveRecipe` (internal/loomcli/arm.go:127-133) on every armed verb, and by the seed read at internal/loomcli/sharedbootstrap.go:100 on the start path. wireLanding still returns seedRunStart's error unwrapped, as a propagated read failure. In the line 83 bullet, drop the second half that writes `{not json` and asserts wireLanding returns an error, keeping only the stamped-seed assertion that `c.env.Landing.RunStart` equals the stamp.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/05-run-bounded-lookup.md:56
    summary: Card 5 tells the implementer to amend internal/landingshed/doc.go's description of the lookup, but doc.go has no such description, so the forced doc edit has no anchor
    fix: In card 5's Intent line 56, replace "`internal/landingshed/doc.go`'s description of the lookup" with an instruction to add one sentence to `internal/landingshed/doc.go`'s Publish paragraph (doc.go:6). The sentence says every landing row's pull-request lookup takes the newest pull request created at or after the run's start, `Deps.RunStart`, whatever its state, and that a zero start keeps the unbounded lookup.
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/05-run-bounded-lookup.md:65
    summary: Card 5 adds findRunPullRequest, a GitHub lookup, to internal/loomcli/runstart.go, whose header card 4 writes to name only the seed read, so the header no longer says what the file holds
    fix: In card 5's Intent line 65, after placing `findRunPullRequest` in `internal/loomcli/runstart.go`, add that the file header gains the run-bounded pull-request lookup the approve and reject verbs use, beside the seed's start read.
---

### [BLOCKING] Approve/reject unreadable-seed refusal: invalid way forward, input rejected upstream

**Location:** `_lyx/plan/05-run-bounded-lookup.md:67`

**Issue:** Card 5's refusal says "way forward: fix the seed file the error names so it decodes, then re-run lyx loom approve".
`contracts/specs/refusal-spec.md` ("What a way forward is") says hand-editing a lyx-owned state file, `seed.json` named, never counts.
`internal/loomcli` is an `internal/loom*` package, so `PATTERN-refusal-way-forward` binds it.
Line 68 names the reaching input: a seed that exists and cannot be read or decoded.
`loomCLI.resolveRecipe` (`internal/loomcli/arm.go:127-133`) already rejects that input.
It reads the self seed through `shedrun.ReadSeed` and returns the same error from `armAt` (`arm.go:184`) in the persistent pre-run (`internal/loomcli/cli.go:230-252`).
That pre-run runs for approve and reject, since neither is in the `inReviewGroup` exemption (`cli.go:284-291`).
So the verb-body branch is reached only through the injected `runStart` fake, never from disk.
The guard-reach lens reported the same refusal as unreachable, merged here.

### [MEDIUM] wireLanding's seed decode error guards a state the arming already rejects

**Location:** `_lyx/plan/05-run-bounded-lookup.md:63`

**Issue:** Both `wireLanding` call sites, `internal/loomcli/arm.go:412` and `buildLoomShed` at `internal/loomcli/sharedbootstrap.go:358`, run after `armAt`.
`armAt`'s `resolveRecipe` refuses an undecodable self seed first, and on the start path so does `sharedbootstrap.go:100`.
The card's `TestWireLanding_TellsLanguageSkills` decode assertion (line 83) calls `wireLanding` directly (`internal/loomcli/wiring_publish_integration_test.go:138-143`).
That skips the arming that rejects the input, so it pins a branch no entry point reaches for the self run.
The way-forward lens also found that the propagated error names no way forward, merged here.

### [LOW] landingshed doc.go edit has no anchor

**Location:** `_lyx/plan/05-run-bounded-lookup.md:56`

**Issue:** A grep of `internal/landingshed/doc.go` for "whatever its state", "newest pull request" and `FindPullRequest` finds nothing.
The only pull-request lines are the per-row ones: line 6 for Publish, line 12 for PR-Align and line 116 for Finalize.
The card points at a passage that does not exist, so D1's Scope-In `doc.go` item has no definite edit.

### [LOW] runstart.go header goes stale in card 5

**Location:** `_lyx/plan/05-run-bounded-lookup.md:65`

**Issue:** Card 4 (`04-seed-run-start.md:8`) creates `internal/loomcli/runstart.go` "with a file header naming what it reads", meaning the seed.
Card 5 then adds `findRunPullRequest`, a GitHub client and list call, to the same file without updating that header.
The header no longer lets a reader tell from it alone what the file holds.

## Rejected

- way-forward lens, MEDIUM: rewording PR-Gate's nil-lookup reason "reopen the pull request or abandon the task" (`internal/landingshed/prgate.go:101`). The decision record's `## Open risks` already records this exact reason as unhelpful when the bound moves. After card 5, Publish creates the run's pull request before PR-Gate runs, so the nil branch is reached only in that recorded risk. Changing the reason is scope no Decision licenses.
- way-forward lens, MEDIUM (wireLanding names no way forward): merged into F2, the same branch.
- guard-reach lens, MEDIUM (SeedUnreadable unreachable): merged into F1, the same refusal.

## Focus directives settled

- Card 5 granularity: not a defect.
  The `FindPullRequest` re-sign forces every caller, the approve and reject closures included, into the same card to compile.
  `Deps.RunStart` must be filled by `landingDeps` in the same card, because `TestLandingDeps_EveryFieldPopulated`'s reflection walk fails on an unfilled field.
  The re-sign arrows are right: `FindPullRequest` and `landingDeps` carry them.
  `approveDeps` and `rejectDeps` are struct types whose fields change, and those take no arrow.
  `approveVerb`, `rejectVerb`, `approveCmd`, `rejectCmd`, `wireLanding` and `PRAlign.findPullRequest` keep their signatures.
- Card order: not a defect.
  Card 5 uses card 4's `seedRunStart`, card 6 uses card 5's `olderOpen`, card 3 uses card 2's `DecisionNoteTitle`, and card 2's rows use card 1's constant.
  Card 5's Publish discarding `olderOpen` (`pr, _, err :=`) compiles.
- Reachability: card 4's zero-bound warning is reachable from a legacy seed or a hand-edited stamp, and from no seed through approve and reject.
  Card 6's older-open stop is reachable through its new `TestPublish_OpenPullRequest` row.
  The unreadable-seed paths are F1 and F2.
- Attack surface: not a defect.
  Card 5 states what the zero bound lets through and what bounds it.
  Card 3 states that only an override lacking the marker newly fails.
- Fidelity: D1 to D4, the Scope-In items and the Constraints are each carried by a card, and no card adds unlicensed scope.
  The `internal/landingshed/doc.go` item is weakly anchored, which is F3.
- Test economy: not a defect.
  No existing test drives `seedRunStart`'s zero-bound warnings at tier 1.
  `TestFindPullRequest_QueryParametersAndEmptyList` covers the landingshed filter, not `findRunPullRequest`'s pass-through of `since`.
  The approve and reject success rows replace `findPR` with a fake, so they cannot cover it either.
- Verify coverage: not a defect.
  `## verify:` runs the untagged and `-tags integration` tests of `internal/alignment`, `internal/landingshed`, `internal/loomcli`, `internal/orchengine` and `contracts/stencils`, which are every package a card targets.
