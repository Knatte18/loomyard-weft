---
round: 1
focus:
  - "internal/runprofile/template.yaml against the board entry's ladder table: do speed, light, solid and wide carry the Discussion, Plan, Code and Darn rounds the table gives, with Code 1 in every loom block, and is each perch's hard cap its rounds plus one?"
  - "internal/runprofile/template.yaml: is docs a darn-only profile (Sonnet writer at medium effort, one Code perch round), and are the defaults loom: solid and darn: light?"
  - "internal/runprofile/runprofile.go and internal/battenshed/seamchild.go: does Seed-Child write only the block of the recipe the task runs, so the shared code_* keys take the loom value in a loom run and the darn value in a darn run? Check the paths for a task naming no profile, a profile with no block for the task's recipe (docs in a loom run), and the reserved defaults entry parsed as if it were a profile."
  - "Speed's perch round count of 0: does profile settings alone turn the perch off, or does the diff silently rely on behavior the board entry lists as open?"
  - "No retired darn.review_* or other darn_*/review_* perch keys, and no merge_max_bounces, remain in the template, tests or docs; the removed budget and darn-wide profiles have no leftover references (internal/battencli, internal/boardcli, internal/configcli, plugins/ly/skills/board/SKILL.md, contracts/stencils/orch)."
  - "Docs landing in the same commit: internal/runprofile/doc.go, internal/battenshed/doc.go, internal/boardcli/doc.go, docs/overview.md and PATTERN.md against the changed behavior; do they state the new profile shape and not the old one?"
  - "Test economy in internal/runprofile/runprofile_test.go, internal/battencli/runsettings_test.go and internal/battenshed/seamchild_test.go: does each new test cover behavior no existing test covers, and does the diff add behavior with no coverage?"
  - "The commit message's claims (reserved defaults entry, hard cap rounds plus one, hub profiles rewritten to the ladder with no retired keys) against the diff."
---

Seeded from the board entry's table, its operator decisions and the one-block-per-recipe decision, plus the diff stat of the range.
