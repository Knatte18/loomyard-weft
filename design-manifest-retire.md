# Retire manifest/: Someday into bolt, durable content into package docs

`manifest-retire` · Planned · feature

## Goal

Retire `manifest/`: first bring roadmap.md's surviving Someday items into bolt in the new tier format, then move the durable reference content into package docs, fix inbound links and delete the folder.
Millhouse has no ownership of `manifest/` (operator, 2026-10-02).

## Someday items still to triage with the operator

Not yet in bolt: `raddle`, `hardener`, `webster-parallel-execution`, `reed-multi-window`, `warp-visibility`, `baseline-drift-gate`, `config-overlay`, `fabric-reset-to-sha`, `finalize-discrepancy-doc`, `shedrecipe-capabilities`, `doctor`, `session-sync`.
Proposed strikes, awaiting the operator: fabric Windows verification (dropping Windows and CLAUDE.md's fslink claim), `semantic-index`, curation-triage automation, VS Code opt-in per worktree (check `batten-open-ide` first), plugin packaging, the shuttle tools-restriction and per-round provider selector, bulk-mode clusters, fabric's ordinary-monorepo verbs, the reed Slack relay.
## manifest/ retirement

The triage was done in the bolt review of 2026-10-01/02; what survives is already in bolt.
This task moves the durable reference content into package docs, fixes inbound links and deletes the folder (Millhouse has no ownership of `manifest/`).

- `manifest/HANDOFF.md`: the hub run loop goes to the `internal/orchcli` package doc; the rest is spent.
- `hardener.md`: the text-review vs behaviour-review distinction and the respawn rationale go to `internal/treadleengine`'s doc.
- `fabric-windows-verification.md`: the Linux-only caveat goes to the `internal/fabricengine` and `internal/fslink` docs.
- `webster-parallel-execution.md`: the 42-card case study and the index-race rationale go to `internal/websterengine`'s doc.
- `review-finding-classification.md`: the convergence case study goes to `internal/burlerengine`'s doc.
- `curation-triage.md`: one line in the boardengine doc ("anyone adds notes, the orchestrator curates"); the automation is struck.
- Inbound links: `internal/treadleengine/doc.go`, `internal/fabricengine/livestate_doc_test.go`, `contracts/specs/loom-plan-spec.md`, `docs/overview.md` (lifecycle text and design links), `crucible/README.md`, `crucible/review-prompt-template.md`, CLAUDE.md's Task completion rule, `.claude/agents/mill-done-gate-fixer.md`.


## From `manifest-to-bolt`: Move everything in manifest/ into bolt and retire the folder

Operator 2026-10-01: LYXHUB takes over all development of loomyard; everything in manifest/ goes into bolt.
The operator also says much of it is outdated: triage each roadmap Someday item with the operator (keep as tier 3 or tier 2, merge, or strike) and carry its designs/<name>.md over as the bolt entry's design doc.
Afterwards: delete manifest/, and update CLAUDE.md (Task completion section names manifest/roadmap.md and manifest/designs/), docs/overview.md's Documentation Lifecycle and any stencil or test that names manifest/.