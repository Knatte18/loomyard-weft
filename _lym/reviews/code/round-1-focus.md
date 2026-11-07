---
round: 1
focus:
  - "internal/lymcwd/lymcwd.go and gate_test.go, docs/shared-libs/lymcwd.md: confirm ResolveWithAnchor has no remaining reference anywhere in the repo (code, docs, stencils, specs), and that the gate_test.go edit drops no coverage of a surviving function."
  - "docs/shared-libs/configengine.md: check each rewritten heading and the layout section against the actual exported signatures and files in internal/configengine; any signature or file the doc still names wrongly is a finding."
  - "internal/websterengine/render.go: check each corrected comment claim against what the code does, since the record asks for claims to be confirmed or corrected."
  - "cmd/lym/stencilseed.go and docs/overview.md: the goal lists stencil paths under {{.specs_dir}}/loom/ naming no existing folder; settle whether every such path in stencils and docs is now fixed, or whether some remain."
  - "cmd/lym/*_test.go, internal/adoptcli/deps_test.go and doc.go: the record asks for jointly edited files to be read whole for broken or doubled comments; read these files whole, not only the diff hunks, and check the false-sort-claim fix against the code."
  - "Scope: the goal also asks to confirm or correct each kept claim from the f7465bd8 commit body; check whether the diff covers each kept claim the body names, and that no code behavior changed beyond removing ResolveWithAnchor."
---

Seed focus for round 1, derived from the Goal in the decision record and the diff stat of the review range.
