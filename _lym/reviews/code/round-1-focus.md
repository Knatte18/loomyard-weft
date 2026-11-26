---
round: 1
focus:
  - "internal/lymcwd/anchor.go, checkCwdAnchorGate and its callers: does the new cause and way-forward text hold on every path that surfaces this error, including `lym creel post`? Check cwds other than a subdirectory (parent, sibling, another worktree), an anchor below the worktree root, and whether any caller replaces or swallows the message."
  - "The way forward in that refusal and in the new shipped entry: is each suggested command usable as written, for example `cd <path>` with a path holding spaces or Windows separators, and `env -C` on the platforms the repository supports?"
  - "internal/heddleengine/shipped.json, entry lym-shell-directory: do its order, targets and wording fit the other shipped entries and the guard rules agents already receive, does it reach the sessions the board entry's agent guidance means, and is any claim in it broader than the gate's behaviour (such as verbs that accept a subdirectory)?"
  - "Docs and constraints: does PATTERN.md, pattern/ or any doc they name (cwd resolution, heddle shipped entries, error message conventions) require an update in this diff, or quote the old refusal text or list the shipped entries?"
  - "_lym/landing/summary.md against the diff: check the `## Merge danger` door and blast radius, given that the diff changes more than internal/lymcwd, and check the claims about which verbs get the new text and about the test failing on the merge base."
  - "Commit trailers and tests: does the Root-Cause trailer name the defect the diff removes, and do the changed tests in internal/lymcwd and internal/heddleengine cover the behaviour the diff adds without asserting static prose beyond what the rubric allows?"
---

The decision record holds only the board entry under `## Goal`, with no added entries, so that entry is the measuring stick.
It offers two remedies for the refusal and also asks for agent guidance about `cd`.
