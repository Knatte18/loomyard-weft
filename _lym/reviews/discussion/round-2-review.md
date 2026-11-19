---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: D5's first bullet now resolves the open verb's cwd with lymcwd.Resolve, as the other board verbs do, and states that the VS Code task's cwd is the anchored directory.
  - id: F2
    status: resolved
    answer: D9 writes the state file and the launcher 0600 with an owner-only DACL on Windows, pinned to internal/githubclient/doc.go and cache_windows.go, and the D9 criterion covers the Windows DACL.
  - id: F3
    status: resolved
    answer: D11 removes a leftover state file right after taking the lock, writes its own only after the bind, and stops taking requests before releasing the lock. D5 waits until the state answers a token-carrying request. Criteria cover an open during a build and during a drain.
  - id: F4
    status: resolved
    answer: D5 and D11 start the server in the prime's anchored directory through lymcwd.NamedWorktree, whatever worktree open ran from. D11's bound states the effect, and a D5 criterion covers an open from a task pair.
  - id: F5
    status: resolved
    answer: D9 sends the CSP as a response header only, D2 picks the mode by location.protocol before reading window.BOARD, and D4 limits the rerender line to file://.
  - id: F6
    status: resolved
    answer: D3's bound now states that an older binary replays every newer commit as a plain cherry-pick, because each touches board-data.js, and that its writes hit the diverged-Bolt refusal until it upgrades.
  - id: F7
    status: resolved
    answer: D10's release refuses a task in a run status, naming lym board set-status, and a D10 criterion covers the refusal.
  - id: F8
    status: resolved
    answer: D2 cites the Fetch Standard and the browsers' file-scheme behaviour, D4 the WHATWG URL Standard's basic URL parser, D5 xdg-utils 1.2.1's open_generic, D6 VS Code's Multi-root Workspaces documentation and _runTaskCommand, and D9 CSP Level 3, which also adds the no-inline-handler rule.
  - id: F9
    status: resolved
    answer: Settled by the added entries 'board.html and board-data.js stay committed render outputs' and 'open's refusal naming lym board rerender is dropped; the rerender line lives in the file:// page'.
  - id: F10
    status: resolved
    answer: D5's bound caps the opener wait at the keep-warm resolver's cap for a caller with a strand name, and a D5 criterion covers success at the cap.
  - id: F11
    status: resolved
    answer: Criteria now cover the file route's t token, the hook members named lym.generated, folders or settings, the token-free envelope address and the D10 Review check of forms, actions and the stale display.
  - id: F12
    status: resolved
    answer: D8's first bullet states that a write's answer computes the revision over the saved entry as a fresh load and migration reads it, and a D8 criterion covers an edit with the returned revision.
  - id: F13
    status: resolved
    answer: D1 now states that each revision is taken over the stored entry before the undiscussed copy, and that the decision-file name follows the copy.
  - id: F14
    status: resolved
    answer: D11 registers the server with runproc.SelfRecord under runproc.HubOwner in a new role kept out of subjectPassSpared, and D13 lists internal/runproc.
  - id: F15
    status: resolved
    answer: D8's create bullet answers a slug refused for any other reason with the board's own refusal message and no entry.
  - id: F16
    status: resolved
    answer: Q7's rationale now says the token still reaches a same-user process, and the Rejected alternative no longer claims the refusal closes that path.
  - id: F17
    status: resolved
    answer: Notes bullet 3 is cut to its first sentence, and bullet 4 reads only that D8's check goes in boardCriticalSection.
  - id: F18
    status: resolved
    answer: D3's last bound sentence now says every board write's sync commit carries board-data.js, because the file carries the render time.
---

Every round-1 finding is resolved in the record as it now stands.
F9, held by the parent, is settled by the two added entries of 2026-11-19 quoted in its answer.
This is an incremental round, so a new finding must be `BLOCKING` and sit on a changed hunk.
Changed hunks were taken from `git -C _lym diff 79d266752e HEAD` over the two files the changes file lists.
No lens finding and no handler finding meets both conditions, so the frontmatter lists none.

## Not raised this round

These items are real defects or gaps, but the incremental-round rule keeps them out of `findings`.
They are recorded here so the parent can act on them.

- **Stale Open risks bullet and support-log citations** (lens:interactions F-a; handler, found on its own).
  `decision-record.md:452-457` still asks the parent to confirm the dropped refusal and the kept render outputs.
  Both 2026-11-19 added entries settle those questions and tell the writer to remove the bullet.
  Q8 and Q9 at `support-log.md:106-107`, the Rejected alternative at `support-log.md:75` and Turn 4 at `support-log.md:62` still cite Open risks.
  The rubric treats such a line as a `consistency` finding.
  It is not on a changed hunk and is not `BLOCKING`.
  Left in place, it sends a settled question back to the parent at the Discussion-Review perch.
- **Revision held by an open edit form across a live refresh** (lens:walkthrough 1, rated `BLOCKING` by the lens).
  The record does not say that an open form keeps the revision from when it opened.
  The Goal at `decision-record.md:8` ("the revision of the entry it edited") and D10's stale display beside the unsent edit at `decision-record.md:298` imply it.
  So this is a clarification of `MEDIUM` weight at most, not a `BLOCKING` gap.
  D10's page bullet at `decision-record.md:297-299` is unchanged since round 1.
- **Projection and board revision read separately** (lens:interactions F-c, lens:bound-breaker 1).
  `GET /api/board` can pair an old projection with a newer revision, so the page stays stale until the next write.
  This is `MEDIUM`, and `decision-record.md:282-284` is unchanged.
- **Server log's pid line under racing spawns** (lens:interactions F-e, lens:bound-breaker 2).
  `decision-record.md:149-150` is a changed hunk.
  The weight is `MEDIUM`: the refusal can name a wrong or missing pid when two servers' stderr share one log.
- **`lymcwd.NamedWorktree` named as the failing call when the prime is missing** (lens:walkthrough 2, lens:bound-breaker 3).
  `NamedWorktree` needs no worktree to exist, so the failure would come from `fabricengine.PrimeName` or the spawn (`internal/lymcwd/lymcwd.go:97-112`).
  It sits on a changed hunk at `decision-record.md:146-148` and is `LOW`.
- **An inherited strand name in a VS Code window an orch launched** (lens:bound-breaker 4).
  The press would be judged as the orch's call.
  It sits on a changed hunk at `decision-record.md:162-165` and is `LOW`.
- **File link over http without the `t` token** (lens:interactions F-d) and **a `design_prefix` holding a separator** (lens:walkthrough 3).
  Both are `LOW`, and `decision-record.md:112` and `:285` are unchanged.

## Rejected

- No lens finding was judged a false positive.
  Every item a lens raised is either kept above as a real but unraisable gap, or a duplicate of another lens's item.
