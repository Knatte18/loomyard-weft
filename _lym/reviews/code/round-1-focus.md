---
round: 1
focus:
  - "internal/fabricengine/hubpull.go: do sides run in the decided order (Bolt, prime, local beams in `git worktree list` order), and does a stop on one side never skip the next?"
  - "internal/fabricengine/hubpull.go and bolt.go: does `Bolt.Pull` take the board write lock before the pull-first path that takes the push lock, and does it return the same skip and way-forward detail a pull-first write reports?"
  - "internal/fabricengine/hubpull.go: is each pair pulled through `Fabric.Pull` under the parent lock of its own code branch, released after that one pair and never held across sides, with a run id naming the hub pull?"
  - "internal/fabricengine/hubpull.go: is a beam whose code branch has no upstream reported `up_to_date` and not `stopped`, and does the beam list come from origin records via fabricengine (not the acting worktree), with the beam mark honored whatever machine owns it?"
  - "internal/fabriccli/hubpull.go, weft_verbs.go, anchordeps.go: does the shared lock seam wait with no time limit, print the holder, and end on Ctrl-C; does `--hub` keep the role guard and leave `lym fabric pull` without the flag unchanged; does any stop yield an error envelope that still carries every side's entry with kind, worktree, outcome and detail?"
  - "Commit messages: does the commit adding the pair pull carry `Pattern-Departure: PATTERN-parent-lock`, and do docs (`doc.go` of both packages) match the diff, with PATTERN untouched unless a new invariant arrived?"
  - "Tests in both hubpull_integration_test.go files: is each acceptance criterion covered once (moved, level hub, dirty prime stop with others moving and non-zero exit, no-upstream beam, parent-lock wait, Bolt under the board write lock), with no duplicate coverage and no uncovered added behavior?"
---

Seed focus from the board entry and decision record; no prior round exists.
