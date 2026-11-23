```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out, through the fix-proposal lens.

Proposals checked: F1, F2, F3, F4, F5, F6, F7, F8, F9, F10, F11, F12, F13 and F14, plus the fixer's own handling of each.

**BLOCKING — `_lym/landing/summary.md`, `## Merge danger` (F6 proposal)**
- **Summary:** The widened blast radius breaks the final-summary spec, which allows one word.
- **Evidence:**
  - The text now reads `**Blast radius:** dobby, the beam's change description and its pull request on GitHub`.
  - `contracts/specs/loom-final-summary-spec.md:42` says "Blast radius is one word naming what breaks…".
  - Line 48 of the same spec lists "a blast radius other than one word" among the refused values.
- **Consequence:** The description would be refused by validation when recorded or landed.
- **A sound change:** Use one word, such as `dobby` or `beam`, and put the rest in the body prose below it. The body already carries the sentence about the pull request and the commits.
- **Not checked:** I did not read `summaryparser`'s code, only the spec.

**MEDIUM — `describe.go`, `endFailedSpawn` (F2 proposal)**
- **Summary:** A failed spawn drops the landings and posts through the best-effort `d.post`, so a failed post loses the only record of the failure.
- **Evidence:**
  - `endFailedSpawn` calls `endDescribe` first, then `d.post`.
  - `post` (`loop.go:440-447`) only logs a failed Notice.
  - The `NearestOrchUnreadable` row uses "creel is down", the same channel `Notice` posts through.
- **Case:** Under a creel outage, `NearestOrch` fails, the landings are dropped, and the post fails too. The description stays stale and nobody is told, which is the loss F8 fixed for `settleDescribe`.
- **Fix:** Route the spawn failure through the same owed-line mechanism, or post before dropping the landings and keep them when the post fails.

**LOW — `describe.go`, `concludeDescribe` (F8 proposal)**
- **Summary:** If the post succeeds but `endDescribe` fails, the next tick posts the owed line again.
- **Evidence:** `OwedLine` is cleared only by `endDescribe`.
- **Case:** A state write that fails once duplicates one notice.

**LOW — `contracts/stencils/beam/beam-template-describe.md` (F1/F9 proposal)**
- **Summary:** F9 is fixed by a prompt rule only, so the intermediate-draft hole stays open for an agent that ignores it.
- **Evidence:** `settleDescribe` still records the first draft that passes `Findings` and stops the strand at once. The reviewer allowed this alternative, so it is a residual risk, not a defect.

No findings for F3, F4, F5, F7, F10, F11, F12, F13 and F14. For F10, `writeBytesAtomic` writes the file at mode 0600 instead of 0644; git ignores that.

Tool calls (1 of 6 used):
1. I ran `grep` over the stencils, `landingshed` and `contracts/specs/loom-final-summary-spec.md`. My context did not hold the spec's blast-radius rule or how other stencils point at the spec.
