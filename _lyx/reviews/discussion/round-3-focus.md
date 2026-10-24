---
round: 3
focus:
  - "Beam landing path: `lyx beam land` runs `lyx batten run <slug>` on a project whose pair already exists and has no batten status file (made by `lyx beam create`, or left after `lyx beam promote` clears batten's records), and batten's start guard (`internal/battencli/startguard.go`) refuses a run with no status file over a present task worktree, records worktree or records branch, naming `cancel --remove`. Settle whether D11 states an exemption or other route so the landing reaches Worktree-Create's skip without that way forward destroying the beam, and whether D11's acceptance criterion covers a beam from `create` and one from `promote`."
  - "Pair creation under a dirty prime: D5 and D7 pass the prime as the acting location when the board picks a beam member's parent, and `internal/fabricengine/add.go` refuses tracked dirt in the acting location first. Settle whether the record states which worktree's dirt Add checks when `Parent` is named, and whether a member can start while the prime holds tracked dirt."
  - "Owner and parent reading from an origin record: D4 falls to `ParentBeam` only when no record is found. Settle whether the record states how a found record becomes a parent slug (today `fabricengine.ParentSlug` matches `ParentBranch` against live worktrees and answers the prime when none holds it, while `hubgeom` `ResolveParent` reads `ParentWorktree`), so a member whose beam pair is absent on this machine still reads the beam as parent and the beam's owner as its own; and whether D13's ready-notice walk states where it goes for a beam absent here."
  - "Attack surface of D8's widened guard: it admits every session in a beam worktree, and the beam's landing runs Finalize agents (merge fixer, conflict session) in that worktree. Settle whether D8 states what those agents can reach across the hub and what bounds it, and whether D11's dropped large-merge reroute states what lands unreviewed and what bounds it."
  - "Board layering: `ComputeLayers` in `internal/boardengine/layer.go` gives every open entry with no open dependency the Ready layer with no kind test. Settle whether D1 or D15 states the layer a project lists under, given that starting a beam project is its landing."
---

Targets the round after round 2.
Entries name the sites and questions only; they cap no severity and presume no verdict.
