---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The proposed doc sentence that an unconfigured remote is a refusal is false for a remote name holding a path separator, such as team/origin, which the code still stats; the doc states it only for a name with no separator. The code and tests are as proposed."
  - id: F2
    disposition: fixed-differently
    reason: "The proposed blast radius of fabric and shed names less than the change reaches: internal/battencli/wire.go:1180 calls RetryOriginReadInTransit, and internal/battencli/land.go:107, internal/beamcli/pr.go:492, internal/loomshed/samplersession.go:449 and internal/landingshed/ctx.go:45 ask shedtransient.Class. The radius is fabric, shed, landing, batten, beam, loom, and the caller list also names rejectionRetryKind."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The greedy pattern is as proposed. The proposed row, an apostrophe in the directory's own name, would pass with the old pattern once F1 applies, since the captured fragment s.git holds no separator; the row puts the apostrophe in a parent directory instead, so the old pattern captures s/plain.git and the row fails."
  - id: F6
    disposition: fixed-differently
    reason: "The file URL row is as proposed. The proposed bare base name in the relative row returns false under F1 before the join is reached, so the row names ./<base> instead. The existing missing relative row names ./removed.git for the same reason."
  - id: F7
    disposition: fixed-differently
    reason: "The proposed Bound line says the read matches git only at the worktree top, which is false for clone, since git resolves its relative source against the current directory; the line limits the statement to push, fetch, pull and ls-remote. The claim about how every call site runs git is left out, as a list of callers that goes stale unnoticed."
---

## Changes

- F1: `namesMissingLocalPath` never reads a name with no path separator as a path.
  Added a hand-built unconfigured-remote row and a real-git `ls-remote` of an unconfigured remote.
  Fixed differently in the doc wording, see the frontmatter.
- F2: `_lym/landing/summary.md` names the callers that change and the wider blast radius, and agrees with the code fixes.
  It is not committed.
  Fixed differently, see the frontmatter.
- F3: `os.Stat` replaces `os.Lstat`, with a dangling-link row.
- F4: the package doc and the `IsTransportFailure` doc say which unmounted paths read as unreachable and bound the empty mount point.
- F5: the path is matched greedily within one line, with an apostrophe row. Fixed differently in the row.
- F6: two rows that fail without the prefix trim or the join. Fixed differently in the relative row.
- F7: a Bound line on relative remote paths. Fixed differently in its wording.

Commits:

- `825759f94` holds F1, F3, F4, F5, F6 and F7, with a `Loosened-Check` line for the changed relative row.
- `d92dc790d` holds the doc wording the self-check corrected.
- `96b0c5fe2` breaks the new comments at sentence boundaries, as the gate findings asked: in the package doc,
  the `IsTransportFailure` doc and the integration test's header. It changes comments only.

Tiers 0 and 1 of `internal/gitexec`, `internal/fabricengine` and `internal/shedtransient` passed after `d92dc790d`.
Tiers 0 and 1 of `internal/gitexec` passed again with the comment repair.

## Self-check

All lenses returned a report.

- fix-code-correctness: no findings.
- fix-consistency, LOW (`rejectionRetryKind` missing from the caller list): fixed.
- fix-consistency, LOW (batten's direct call and the other users of the class not named): fixed, merged with the two blast-radius findings below.
- fix-proposal, MEDIUM (blast radius still too narrow): fixed.
- fix-proposal, LOW (doc claim false for an unconfigured name with a separator): fixed.
- fix-proposal, LOW (relative-path Bound line false for clone): fixed.
- fix-affected-set, MEDIUM (batten call and blast radius): fixed.
- fix-affected-set, LOW (`rejectionRetryKind` omitted): fixed.

## Disputed

## Deferred

## Rulings
