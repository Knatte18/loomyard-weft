---
round: 1
focus:
  - "contracts/orchrole/orch-role.md, new 'Making a beam' section: does `lym batten cancel <slug> --remove` exist as written? The board entry names `lym batten cancel <slug>` with no flag; check the flag against the batten cancel command and what it does to the worktree."
  - "Same section: check each claim the board entry does not state against the code. 'dobby starts no project entry, so a release never makes a beam'; the task-run inner producers 'tapestry, arras, darn or sampler'; and 'filtered to its current producer' for `lym batten status`. Does each hold, and does the section name only commands and flags that exist?"
  - "Does the section cover all six design points of the board entry (project upsert then create from prime or parent beam only, Worktree-Create/Seed-Child/Beam-Up rows and the late orch, --anchor, `beam start` refusing a task pair, a task run being no beam with status then cancel then create, never fabric add or hand deletion)? Is anything the entry does not license added, such as the operator go-ahead requirement for creating a beam?"
  - "Is the pair of beam-vs-task rows consistent with the batten row names the code defines (Worktree-Create, Seed-Child, Beam-Up)? Is the new section placed and worded consistently with the rest of the orch role, and is the command list earlier in the file still accurate for the commands the section now relies on?"
  - "Does anything else pin the orch role text (a stencil test, a golden, a doc that mirrors it) that the diff should have updated? Check the stencil-wording rule: does the diff drop, weaken or contradict an existing rule of the role?"
  - "The diff touches one file and adds no test; judge under test economy whether any behavior is added that needs coverage, treating the role's static prose as exempt."
---

Single-commit, single-file docs change to a role stencil.
The riskiest content is the set of factual claims about lym commands and rows, which the reviewer should check against the code rather than the writer's summary.
