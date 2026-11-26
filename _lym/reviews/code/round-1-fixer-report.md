---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed wording says the link stays until the beam lands, which the code does not hold: `lym beam pr close` drops the pull request from the landing record before the beam lands (internal/beamcli/pr.go, prClose). The legend and doc.go say only that the link is the one the landing record names and may already be closed or merged on GitHub."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

# Round 1 fixer report

## Changes

- F1 (`728b9da4b`): `readBeamMode` refuses a beam whose worktree is not on this machine, so `BeamModes` leaves it out.
  The dead not-exist branch is gone, `TestBeamModes` names a second beam with no worktree,
  and the `BeamModes` docs, `internal/hubgeom/doc.go` and the landing summary say so.
- F2 (`728b9da4b`), fixed differently: the README legend and `internal/boardengine/doc.go` say that the `Beam PR` link may already be closed or merged on GitHub.
  The claim that the link stays until the beam lands was left out, for the reason in the frontmatter.
- F3 (`728b9da4b`): `primeOfBoard` lists through `fabricengine.List` and takes the main worktree, which comes first; the `dotgit` import is gone.
- F4 (`75c1e98ba`): the `Config` doc comment names `LookupWithSource` among the readers.
- F5, F6, F7 (`f90f456b9`): `TestRenderToDisk_ShowsTheInstalledBeamModes` has rows for the beam-handle source, the code-span slug and a failed read.
  Its first row reads `project-root.md` and `project-beam-a.md`, and its fixture holds a done beam and a non-beam project.
- Self-check (`9f9a0ce8a`, `b675c339f`, `330b93712`): the legend, `internal/boardengine/doc.go`, `pattern/PATTERN-config-layers.md` and the log level of an absent beam worktree.
- Gate repair (`5ab11105b`): the gate found comment lines the round had wrapped mid-sentence in `internal/boardengine/beammodes.go`,
  `internal/boardengine/beammodes_test.go` and `internal/hubgeom/beammodes.go`.
  Those comments are rewritten with one sentence per line, with no change to code.
- `_lym/landing/summary.md` is edited for F1, F2, F3 and the self-check, and is not committed by this round.

Verification: `lym gate test --tier 0 --tier 1 ./internal/hubgeom ./internal/boardengine` passed after the gate repair, and `TestBeamModes` shows as passed in its tier1 log.
`lym gate test --tier 0 ./internal/dobbyengine` passed for F4.

## Self-check

- fix-consistency, MEDIUM, the legend's "every beam" no longer holds with an absent beam left out: fixed as proposed.
- fix-consistency, LOW, `internal/boardengine/doc.go` names no exception for an absent beam: fixed as proposed.
- fix-consistency, LOW, the `PATTERN-config-layers` sentence: fixed as proposed.
- fix-consistency, LOW, the summary's board-files section and Tests section: fixed as proposed.
- fix-code-correctness, LOW, the legend's "every beam": fixed as proposed, merged with the consistency finding.
- fix-affected-set, LOW, `internal/boardengine/doc.go` and the summary sentence: fixed as proposed, merged.
- fix-affected-set, LOW, the legend's "every beam": fixed as proposed, merged.
- fix-affected-set, LOW, the summary's Tests section: fixed as proposed, merged.
- fix-proposal, MEDIUM, a left-out beam can read as taking the hub-wide default: fixed as proposed, merged with the legend finding.
- fix-proposal, LOW, an absent worktree warns on every render: fixed as proposed; it is now an info line, and other failed reads still warn.

## Disputed

None.

## Deferred

None.

## Rulings

None.
