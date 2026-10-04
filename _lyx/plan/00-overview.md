---
format: 5
approved: false
language: go
---

# Plan: pattern-init

loomyard's invariants move from `CONSTRAINTS.md` into `PATTERN.md` at the repo root, a thin overview of one-line `PATTERN-*` entries with background files under `pattern/`, and the overview is inlined into the prompt of every lyx agent that designs, plans, edits or judges.
The same pass carries several related changes:
- it retires `manifest/` into the board and package docs;
- it retires fabric's PATTERN-residue report;
- it extends the fabric-vocabulary scan beyond production Go;
- it fixes the model-spec leftovers;
- it gives every spawn site a role constant;
- it closes the PR #301 ide follow-ups;
- it removes `shedrun.Seed.Parent`.

The decision record, including its "Operator additions (2026-10-04)" section, is the answer key.

## Card Index

1 — retire-pattern-residue — remove fabric's PATTERN-residue report and its pull-envelope key
2 — pattern-warp-root — PATTERN at the warp worktree root, overview inlined, whitespace-only inactive
3 — pattern-designer-role — RoleDesigner directive on Discussion-Write
4 — pattern-judge-role — RoleJudge directive on the Bouncer judge; role doc rewritten
5 — pattern-content — write PATTERN.md and its background files from CONSTRAINTS.md
6 — pattern-format-checker — pure PATTERN format checker plus the real-file test
7 — constraints-stencil-repoint — stencils stop naming CONSTRAINTS.md; citation and recovery tests follow
8 — constraints-repoint-cmd — cmd/lyx and tools citations name PATTERN entries
9 — constraints-repoint-internal-a-l — internal citations, batcher through lyxcwd
10 — constraints-repoint-internal-m-z — internal citations, mergeresolve through websterengine
11 — manifest-content-moves — board notes and package-doc moves out of manifest/
12 — manifest-retire — delete manifest/, fix inbound links, rewrite the docs lifecycle
13 — constraints-docs-and-delete — repoint docs, specs, crucible and skill; delete CONSTRAINTS.md
14 — vocab-sweep-tests-core — reword weft/warp in cmd, contracts, tools and early internal tests
15 — vocab-sweep-tests-loom — reword weft/warp in loomcli, loomrecipe and loomshed tests
16 — vocab-sweep-tests-rest — reword weft/warp in the remaining internal tests
17 — vocab-sweep-docs — reword weft/warp in docs, specs, crucible, README and the skill
18 — vocab-scan-extend — widen the fabric-vocabulary scan with closed allowlist and exempt-identifier lists
19 — model-spec-leftovers — short-form spec doc, template links, landingshed fixtures
20 — spawn-role-constants — role constants at every spawn site plus a literal-role scan
21 — hub-workspace-fabric — prime path via WorktreePath, portal anchor kept, existing folders only
22 — ide-spawn-warnings — log level by spawn target and the relative-setting warning
23 — seed-parent-retired — drop Seed.Parent and its fallback; old seeds still decode

## Shared Decisions

### Decision: entry-map

- **Decision:** Every `CONSTRAINTS.md` section maps to exactly one PATTERN entry by this table.
  Cards 5, 7–10 and 13 use these names verbatim, and no other card invents one.
  The `##` topic grouping in `PATTERN.md` follows the Topic column.
  "bg" marks an entry with a background file `pattern/PATTERN-<name>.md`, the exact set card 5 creates.

  | `CONSTRAINTS.md` section | Entry | Topic | bg |
  |---|---|---|---|
  | Cwd Resolution Invariant | `PATTERN-cwd-resolution` | Paths and geometry | bg |
  | Told-Geometry Invariant | `PATTERN-told-geometry` | Paths and geometry | bg |
  | Lyxdirs Single-Declarer Invariant | `PATTERN-lyxdirs-single-declarer` | Paths and geometry | |
  | Durable-vs-Ephemeral State Invariant | `PATTERN-durable-vs-ephemeral-state` | Paths and geometry | bg |
  | Hub Containment Invariant | `PATTERN-hub-containment` | Paths and geometry | bg |
  | Hub Suffix Invariant | `PATTERN-hub-suffix` | Paths and geometry | bg |
  | CLI / Cobra Invariant (including its package-naming rule) | `PATTERN-cli-cobra` | CLI | bg |
  | Cliwire Sole-Wiring Invariant | `PATTERN-cliwire-sole-wiring` | CLI | bg |
  | Config Strictness Invariant | `PATTERN-config-strictness` | CLI | bg |
  | Refusal Way-Forward Invariant | `PATTERN-refusal-way-forward` | CLI | bg |
  | Fabric Vocabulary Invariant | `PATTERN-fabric-vocabulary` | Fabric and git | bg |
  | Fabric Git Invariant (warp + weft) | `PATTERN-fabric-git` | Fabric and git | bg |
  | Fabric Destruction Chokepoint Invariant | `PATTERN-fabric-destruction-chokepoint` | Fabric and git | bg |
  | Fabric Write-Side Containment Invariant | `PATTERN-fabric-write-containment` | Fabric and git | |
  | Mutation Record Invariant | `PATTERN-mutation-record` | Fabric and git | bg |
  | Pair Teardown Invariant | `PATTERN-pair-teardown` | Fabric and git | bg |
  | Batten Bookend Invariant | `PATTERN-batten-bookend` | Fabric and git | bg |
  | GitHub Auth Invariant | `PATTERN-github-auth` | Fabric and git | |
  | gitrepo Client Boundary Invariant | `PATTERN-gitrepo-client-boundary` | Fabric and git | bg |
  | gitexec Checked-Call Invariant | `PATTERN-gitexec-checked-call` | Fabric and git | |
  | Never Force-Add Invariant | `PATTERN-never-force-add` | Fabric and git | |
  | Shed Producer-Seam Invariant | `PATTERN-shed-producer-seam` | Shed and loom | |
  | Shed Recipe Registry Invariant | `PATTERN-shed-recipe-registry` | Shed and loom | |
  | Shed Verb-Set Invariant | `PATTERN-shed-verb-set` | Shed and loom | bg |
  | Shed Run-Directory Invariant | `PATTERN-shed-run-directory` | Shed and loom | bg |
  | Transient Stop Invariant | `PATTERN-transient-stop` | Shed and loom | bg |
  | Driver Choice Single-Site Invariant | `PATTERN-driver-choice-single-site` | Shed and loom | bg |
  | Treadle Runner-Seam Invariant | `PATTERN-treadle-runner-seam` | Shed and loom | |
  | Plan Generation Invariant | `PATTERN-plan-generation` | Shed and loom | bg |
  | Ref-Shape Registry Invariant | `PATTERN-ref-shape-registry` | Shed and loom | bg |
  | Glyph Conversion Chokepoint Invariant | `PATTERN-glyph-conversion-chokepoint` | Shed and loom | bg |
  | Gate Self-Check Parity Invariant | `PATTERN-gate-self-check-parity` | Shed and loom | bg |
  | Verified-Tree Invariant | `PATTERN-verified-tree` | Shed and loom | bg |
  | Batcher Registry+Config Invariant | `PATTERN-batcher-registry` | Shed and loom | |
  | Review Round Invariant | `PATTERN-review-round` | Shed and loom | bg |
  | Planparser, Discussionparser, Summaryparser and Recipe-Format Sole-Parser Invariants | `PATTERN-sole-parsers` | Shed and loom | bg |
  | Agent Name Invariant | `PATTERN-agent-name` | Agents and prompts | bg |
  | Stencil Ownership Invariant | `PATTERN-stencil-ownership` | Agents and prompts | bg |
  | Producer Pointer-Rule Invariant | `PATTERN-producer-pointer-rule` | Agents and prompts | |
  | Friction Capture Invariant | `PATTERN-friction-capture` | Agents and prompts | bg |
  | Completion Signal Invariant | `PATTERN-completion-signal` | Agents and prompts | bg |
  | Shuttle Provider-Seam Invariant | `PATTERN-shuttle-provider-seam` | Agents and prompts | bg |
  | Shell Mechanics Seam | `PATTERN-shell-mechanics-seam` | Agents and prompts | |
  | Pane Binary Resolution | `PATTERN-pane-binary-resolution` | Agents and prompts | bg |
  | Live-Substrate Spawn Observability | `PATTERN-spawn-observability` | Agents and prompts | bg |
  | gitkit, Modelspec, Tokenvocab, Buildinfo, Standalonestate, Pattern and Friction Leaf Invariants | `PATTERN-leaf-packages` | Packages | bg |
  | Testkit Invariant | `PATTERN-testkit` | Testing | bg |
  | hubforge Fabric-Fixture Invariant | `PATTERN-hubforge-fixtures` | Testing | bg |
  | Test Tier Purity Invariant | `PATTERN-test-tier-purity` | Testing | bg |
  | Hermetic Git Test Environment Invariant | `PATTERN-hermetic-git-tests` | Testing | bg |
  | Tmux Test Isolation Invariant | `PATTERN-tmux-test-isolation` | Testing | bg |
  | Sandbox Suite Coverage | `PATTERN-sandbox-coverage` | Testing | |
  | Dev/Prod Binary Separation | `PATTERN-dev-prod-binary-separation` | Testing | |
  | Quarry CGO Requirement Invariant | `PATTERN-quarry-cgo` | Testing | bg |
  | Markdown Link Integrity | `PATTERN-markdown-link-integrity` | Docs | bg |
  | Comment Line-Break Convention | `PATTERN-comment-line-breaks` | Docs | bg |
  | Documentation Lifecycle | `PATTERN-documentation-lifecycle` | Docs | |

- **Rationale:** D1 lets near-duplicate leaf and sole-parser sections merge, and names are chosen to read clearly.
  One table keeps cards 5, 7–10 and 13 from drifting apart.
- **Applies to:** cards 5, 7, 8, 9, 10, 12, 13, 18, 20

### Decision: pattern-text

- **Decision:** `PATTERN.md` and every `pattern/PATTERN-*.md` state rules for the code only (Operator additions 6).
  They never say how an agent, a run, a stencil, Claude, a skill or a development workflow uses them.
  Naming loomyard's own packages, the `lyx` CLI and its types is fine.
  Every `.md` uses semantic line breaks; an entry line stays one line.
- **Rationale:** operator decision of 2026-10-04; the injection and the agent instructions live in the stencils and `internal/pattern`.
- **Applies to:** cards 5, 12, 18, 20

### Decision: directive-root

- **Decision:** `pattern.Directive` and `pattern.File` take the warp worktree root, never the anchor path.
  The directive stencils carry one required `{{.pattern_overview}}` marker and the fixed background pointer literal `pattern/`.
  Every role's call site passes its spawn's worktree root.
- **Rationale:** `PATTERN.md` sits next to `go.mod` at the repo root (Operator additions 1–2).
- **Applies to:** cards 2, 3, 4

### Decision: vocab-sweep

- **Decision:** A file in the extended scan's scope names neither `weft` nor `warp` (case-insensitive substring, as the scan matches) outside the `vocab-exempt` identifiers.
  - Local identifiers and test names are renamed with neutral words: `repo`, `pair`, `sibling`, `records`, `code`, `fabric`.
  - Comments and strings are rephrased so no side needs naming.
  - A string compared against production output that names a side is not reworded by a sweep card; it is reported as a deviation for card 18 to judge.
- **Rationale:** D10, "nothing outside fabric should know them".
- **Applies to:** cards 14, 15, 16, 17, 18

### Decision: vocab-exempt

- **Decision:** These owner-set exports and env-var names are the closed list of tokens a non-owner file may spell.
  The extended scan strips exactly these whole tokens before matching.
  Renaming them is out of scope, recorded as the board note `fabric-api-vocabulary`.

  | Token | Declared in |
  |---|---|
  | `PairWarpWorktree` | `internal/hubforge` (`*Hub` method) |
  | `PairWeftSibling` | `internal/hubforge` (`*Hub` method) |
  | `WeftBare` | `internal/hubforge` (`Hub` field) |
  | `WarpBare` | `internal/hubforge` (`Hub` field) |
  | `PrimeWeft` | `internal/hubforge` (`*Hub` method) |
  | `WeftWorktree` | `internal/fabricengine` |
  | `WeftWorktreePath` | `internal/fabricengine` |
  | `WeftBranchName` | `internal/fabricengine` |
  | `WeftRepoRoot` | `internal/fabricengine` |
  | `CommitWeftPaths` | `internal/fabricengine` |
  | `WarpLyxLink` | `internal/fabricengine` |
  | `WEFT_SKIP_GIT` | fabric's env-var name |
  | `WEFT_SKIP_PUSH` | fabric's env-var name |

  The `weftname` package name in an import path stays governed by the existing `weftname` import rule and its `geometry_test.go` exemption, not by this list.
  Card 18 may add an entry only for an owner-set spelling reported by cards 14–17, with a reason naming its declaration.
- **Rationale:** approved by the operator through the orchestrator on 2026-10-04; it keeps the scan closed (a new side-carrying identifier still fails) without renaming fabric's API in this task.
- **Applies to:** cards 14, 15, 16, 17, 18

## verify:

go build ./...
go vet ./...
go vet -tags integration ./...
go vet -tags smoke ./...
go test ./...
go test -tags integration ./...
