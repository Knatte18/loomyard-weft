```yaml
lens: fix-proposal
outcome: reported
```

**Findings**

- LOW. `internal/summaryparser/description.go`, `isFenceLine` and `mergeDangerSection`, carrying out F1.
  - **Summary:** F1's proposal toggles the fence state on every line that starts with ``` or ~~~, whatever the marker's length or kind.
  - **Case where it is wrong:** a four-backtick fence that shows a three-backtick block inside it. The inner ``` line flips the state back to "outside", so the lines after it are read as live. A `## Merge danger` heading or `# comment` line there is then taken as real.
  - **Evidence:** the body "````\n```\n## Merge danger\n**Door:** two-way\n**Blast radius:** docs\n```\n````\n" would yield a section. A real section holding a ~~~ block that shows a ``` line has the same problem.
  - **Severity:** edge case; descriptions rarely nest fences.
  - **Not tested:** the new rows only cover a plain ``` block and a ~~~ block.

I found no other breakage:

- F2: the glossary edits drop only storage and runtime detail. They leave the line-6 path to `glossary.md`, which the review did not name.
- F3: the help text, doc and `way_forward` agree with `RecordBeamDescription`. No test asserts the old `way_forward` wording.
- F4, F5, F6: each new row fails under the mutant its finding names.
  - Labels directly under the heading: a section start shifted by one line drops the Door line.
  - A `# Other` heading: without the `# ` clause, the section would not end there.
  - The `One-way` door line with a final period: without the period strip, the door reads `One-way`.` and fails.
- F7: `beamDraft(...)` is exactly what `DraftBeamDescription` writes. The draft's placeholders ("one-way or two-way", "one word") produce both findings.
- F8: the assertion is sound, since the real stencils are seeded.
- F9: the `levelOnly` prefix `"## "` does not match a `### ` heading.
- F10: the layout now relies on the glossary value ending in a newline, so the `{{.glossary}}\n## Step 5` gap holds. The existing blank-line-before-Step-5 check at `discussiontable_test.go:243` guards that, and the gate passed.

Tool calls: none; my context already held every changed file and the gate result.
