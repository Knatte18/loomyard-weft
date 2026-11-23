```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F2, F3 and F6, all fixed as proposed).

**LOW — `contracts/stencils/webster/webster-body-verify-fix.md`, new first bullet under `## The rules`**
- **Summary:** the new bullet duplicates the cold frame's rule in the cold path and words it differently.
- **Evidence:**
  - The file's header says `webster-body-verify-fix.md` is joined behind `gatefix-frame-cold` for the cold fixer strand.
  - That frame already has `## External tools and processes`, saying "from a test in the repository that the gate runs".
  - The cold fixer strand therefore reads the rule twice, with and without the `--tier <that tier>` single-test form.
  - F3's `fix` key named burler, webster card and landing fixers plus "any other" stencil a grep turns up.
  - This stencil's cold path is covered by the frame, so only its fork path (`webster-prefix-verify-fix-fork`) needed the bullet.
  - The reviewer's own wording ("worded as in `gatefix-frame-cold.md`") would have made the two copies match.
- **Suggestion:** reword the bullet to the frame's text, or move it into `webster-prefix-verify-fix-fork.md`, which only the fork path reads.

**No finding — F1 steer text.** The new text holds no quote or backslash, since "the package that holds the test" replaced the apostrophe form. "--tier" and "--" in a steer are not a hazard: `steerPublishDeny` already carries "--input". The golden edit replaced both the steer and notice copies, because the notice contains the steer.

**No finding — F2, F6, and the F3 stencils for burler, landing and webster implementer.** I found no case where the proposed text is wrong or contradicts a sibling. The webster implementer's "one route to a test run" section already allows `--tier <n>`, so the new section agrees with it.

**Not covered:** `webster-fragment-card-fixer.md`, named by F3, was left unchanged. Its agent is the implementer, which now carries the rule, so I count that as covered.

**Tool calls:** none. My context already held every changed file and the diffs.
