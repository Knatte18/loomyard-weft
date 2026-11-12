```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content to move out of or cut from `decision-record.md`, applying the rubric's relocation and completeness-before-leanness tests.

**Findings**

1. **MEDIUM**, D1 rationale, Auto-mode assumptions bullet 1 and Open risks bullet 1. The argument that the extra branch served a merge the direction drops appears in all three places.
   - Evidence: D1's last sentence ("The extra branch and junction the board's Idea names served a merge…"), the first Auto-mode assumption ("supersedes the extra-branch Idea, since that branch existed for a merge…"), the first Open risk, and the support log's first Rejected alternative.
   - Fix: keep the requirement "adds no new branch, worktree or junction" in D1. Keep the parent question in Open risks. Cut the deliberation from D1 and the duplicate assumption; the support log already holds the reasoning.

2. **MEDIUM**, D2 and Open risks bullet 3. D2's last two lines ("Git cost of the fork… Disk cost… about 41 MB…") describe consequences and decide nothing, and the disk cost is said again in Open risks with a second figure.
   - Evidence: D2's "Disk cost: … about 41 MB in the compact form of D6" and Open risks' "about 41 MB of maps (D2); twenty pairs hold about 0.8 GB".
   - Fix: state the cost once, in Open risks, and drop it from D2.

3. **MEDIUM**, Notes for the plan writer bullet 2. It carries a requirement the plan writer must meet, but the section is optional hints only: the dobby must read the exit record from its new place under the prime's `.lym/testcover/`.
   - Evidence: "`internal/dobbyengine/testmap.go` reads the exit record from the bolt today; it moves with D4…". D4 says only "under the worktree's `.lym/`" and never says the dobby reads it there.
   - Fix (completeness before leanness): add "the dobby reads the prime's exit record from its new place" to D4, then drop the note or shorten it to the file pointer.

4. **LOW**, D6 rationale. The measurement sentence pins perishable numbers in the plan writer's input.
   - Evidence: "211 maps, about 14 MB compressed" and "Measured on the 2026-11-12 set: 56 MB indented, 41 MB compact; gzip 14.7 MB and 14.2 MB".
   - The support log's Turn 1 already records these numbers.
   - Fix: keep a one-clause rationale ("so a landing commits only the maps it changed; compaction cuts disk per worktree by about a quarter") and move the full figures to the support log.

5. **LOW**, Constraints, the "Fact" bullets. Each fact is stated twice:
   - The fork fact (`forkrecords.go` drops only run records) also appears in D1 and D2.
   - The `PlanUpkeep` `Changed` fact also appears in D6's rationale.
   - Fix: keep each fact in Constraints with its evidence, and have D1, D2 and D6 refer to it without restating it.

6. **LOW**, Constraints, the `PATTERN-owner-machine` bullet. It duplicates `## PATTERN changes`, which already lists the change.
   - Fix: cut the bullet, or shorten it to "see PATTERN changes".

7. **LOW**, Scope "In". Its bullets restate D1 to D9 one for one, so the reader meets the same list twice.
   - Fix: shorten "In" to a single line pointing at the Decisions. Keep "Out", which carries information found nowhere else.

**Other (emphasis)**

8. **LOW**, D5 and D6 headings. Each heading bundles two independent decisions under one id: D5 joins the owner-machine refusal with keeping the stamp, and D6 joins skipping unchanged maps with compact JSON.
   - Acceptance criteria for D5 cover only the refusal. Nothing covers "stamp unchanged", which is fine since nothing changes, but the bundling hides that.
   - Fix: split D6 into two decisions, or accept as is.

Tool calls: none. My context already held both files, the rubric and the exploration results.
