```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points.

**Decisions.** The answer key is the decision record alone. Its `## Goal` has four points, and the record has no `Added after Discussion` entries.
- **G1:** `role-intake.md` gets a fourth outcome, close as already fixed. Its reason names the fixing commit on `main` and how the fork saw it.
- **G2:** the fork judges from `main`'s code and history: the code path the issue names, and the commits since the issue's build that touch it.
- **G3:** when the fork cannot tell, the issue goes on the board as now.
- **G4:** `lym board intake close` may need a reason form that names the fixing commit; the run decides.

**Changes mapped to decisions**
- **C1** `contracts/orchrole/role-intake.md` hunk @23 (diff 001:9-23): the outcome count goes from three to four, and a new "close as already fixed" bullet is added. It traces to G1 (commit named in the reason, what the fork saw on the path), G2 (judged from the main branch and its history, the named path, commits since the issue was filed or since its build) and G3 ("cannot tell … becomes a note or a fold like any other").
- **C2** `internal/boardcli/intake.go` hunks @114 and @125 (diff 002:10, 20-24): the `close` Short and Long help now name the already-fixed case. It traces to G4: the run decided to keep the free-text `reason` with `"completed":true`, and the help shows that form. The commit message and `_lym/landing/summary.md` "Design decisions" record the choice.
- **C3** `plugins/ly/skills/board/SKILL.md` hunks @228 and @243 (diff 003:9, 17): an example line and a bullet sentence. It traces to G4, and C1 forces it, because the fork "follows `ly:board` for the commands" (role-intake.md:24).

**Decisions mapped to changes**
- G1: C1
- G2: C1
- G3: C1
- G4: C2, C3

**Findings:** none. Every change traces to a goal point or is forced by one, and every goal point has a change that builds it. The contract adds "or cannot name the fixing commit", which is stricter than G3. G1 requires a reason that names the commit, so this follows from it and is not new scope. "Since the issue was filed or since the build it reports" extends G2's "since the issue's build" to cover issues that report no build, so it stays within G2.

Tool calls: none.
