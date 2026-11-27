---
round: 1
focus:
  - "internal/landingshed/fixerrun.go, the ancestry test on a moved remote tip: settle whether 'the tip is startHead or an ancestor of it' matches what the entry asks for in every movement case, including a ref rewound or force-moved to an older commit, a tip that predates the session without being in startHead, and a tip that diverges from startHead, and whether the halt's startHead..remoteTip range and wording hold in each."
  - "internal/landingshed/fixerrun.go and internal/loomcli/landingdeps.go, the new TaskHead read and IsAncestor seam: settle whether argument order, the repository of the pair that is queried, the timing of the start-head read and the handling of an ancestry error are right, including what the attempts record and the row are left with when the call fails."
  - "internal/landingshed/fixerrun.go sessionRef: settle from the shuttle and seat engines whether a real run populates Engine, SessionID and RunDir on each outcome (done, timeout, died, other), and whether every halt reason that points a reader at session evidence, inside landingshed and any caller that renders these outcomes, now names something that survives cleanup."
  - "internal/gitrepo/push.go: settle whether every path that moves or deletes a remote ref of a task code branch reaches the new trace line, whether the logged ref and tip are accurate on each path (retry, rebase, delete, already up to date), and whether the line reaches a log that identifies the writing process, as the change description claims."
  - "Repository documents and package docs describing the pushed end, the fixer or fix review halt text, or the TaskRemoteTip behaviour: settle whether any is left stating the old behaviour, and whether the repository's constraints document requires a doc change this diff lacks."
  - "Tests in the range: settle whether the changed halt-reason assertions are no looser than before, and which added behaviours have no coverage, such as the real IsAncestor wiring, the ancestry-error path, the trace line on a delete and sessionRef with no session id."
---

Seed pass over `d51900fb264f4423409439bc74805c8a92b0ab77..HEAD`.
The answer key is a decision record alone, whose entry covers two imported issues:
blame for a moved remote ref, and halt reasons that name a removed run directory.
No redundancy, reach or mutation report is current.
