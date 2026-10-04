# Decision record — pattern-init

## Goal

loomyard's invariants move from `CONSTRAINTS.md` into `PATTERN.md` at the repo root, a thin overview of one-line `PATTERN-*` entries with background files, and every lyx agent that touches design, code or review gets that overview inlined into its prompt.
`manifest/` is retired into the board and package docs in the same pass, since both sweeps repoint the same docs, stencils and `CLAUDE.md`.
The folded-in repo conventions land with it: the fabric-vocabulary scan reaches beyond production Go, the model-spec leftovers are fixed, spawn sites name their agent roles through module constants, the ide hub-workspace follow-ups are closed, and `shedrun.Seed.Parent` is retired.

## Scope

In:

- `PATTERN.md` and `pattern/PATTERN-<name>.md` at the repo root, tracked warp files, carrying every invariant now in `CONSTRAINTS.md`; `CONSTRAINTS.md` deleted.
- `internal/pattern`: inlined overview, format checker, two new roles (Discussion-Write, Bouncer judge), empty-file handling, paths resolved from the warp worktree root.
- The `contracts/stencils/pattern/` directive stencils gain an overview marker; Discussion-Write's and the Bouncer judge's templates gain the directive.
- `fabricengine`'s PATTERN-residue report retired (see Operator additions).
- Repointing every file that names `CONSTRAINTS.md` (`CLAUDE.md`, stencils, the ly-drive skill, `contracts/specs`, code comments and doc.go files, `docs/`, `citation_enforcement_test.go`, the webster template test).
- `manifest/` retirement: board notes for the surviving Someday items, durable reference content into package docs, inbound links fixed, folder deleted, `CLAUDE.md`'s and `docs/overview.md`'s lifecycle text rewritten.
- Fabric-vocabulary scan extended to non-Go files and test files outside the owner set; existing hits swept.
- Model-spec leftovers: `contracts/specs/llm-model-spec.md` examples, the dead `docs/reference/model-spec.md` link in every template comment, `internal/landingshed/config_test.go`'s long-form fixtures.
- Agent role constants at every spawn site, and the Agent Name Invariant's ownership sentence.
- The four ide hub-workspace follow-ups from PR #301.
- `shedrun.Seed.Parent` and its fallback removed.

Out:

- A `lyx pattern init` verb (see Decisions).
- The `fable[high]` model trial (moved to `convergence-plan`).
- Webster's `master` → `webster` role rename (owned by `webster-verify-gate`).
- Per-task or per-card PATTERN entry selection.

## Decisions

### D1 — PATTERN overview format

Decision: `PATTERN.md` opens with a `# PATTERN` heading and one sentence of framing, then a few `##` topic headings (paths and geometry, CLI, fabric and git, shed and loom, agents and prompts, testing, docs — the plan picks the final grouping).
Under each heading every entry is exactly one bullet line, shaped:
`` - `PATTERN-<kebab-name>` — <when it applies and what it requires>[ (test)][ — [background](pattern/PATTERN-<kebab-name>.md)] ``.
The one-liner names both the trigger and the requirement, so an agent can judge relevance from the line alone.
`(test)` marks an entry enforced by `go test`.
An entry with no link is complete as written.
The `<kebab-name>` is chosen to read clearly, not derived from the old heading.
A background file is `pattern/PATTERN-<kebab-name>.md` at the repo root, next to `PATTERN.md`, named exactly as its entry, holding the rationale, owner sets, enforcement-test names and edge cases that do not fit a line.
Every `CONSTRAINTS.md` section becomes one entry; sections that are near-duplicates (the several sole-parser and leaf invariants) may merge into one entry when one line covers them.
Rationale: operator format of 2026-10-04; one grep on the entry name finds the entry, its file and every citation.

### D2 — Format checker in `internal/pattern`, run against loomyard's own PATTERN by `go test`

Decision: `internal/pattern` gains a pure checker over a PATTERN root that reports findings for:
names not matching `PATTERN-[a-z0-9-]+` or duplicated;
an entry bullet spanning more than one line;
an entry line longer than 320 characters;
an overview larger than 16 KiB;
an overview that is empty or whitespace-only;
an overview with no entry bullet;
a link that does not resolve or does not point at `pattern/<the entry's own name>.md`;
a file under `pattern/` with no entry.
The checker is unit-tested on fixtures for each finding kind.
A second test locates the module root (the directory holding `go.mod`) and runs the checker on its `PATTERN.md`.
That test never skips: a missing `PATTERN.md` or one that fails the checker fails it.
The checker is not called at prompt-render time.
Rationale: the operator asked for a format test with a size cap; a library checker keeps the rules in one place and testable on fixtures, and the real-file test keeps loomyard's PATTERN honest under every verify.
`PATTERN.md` is a tracked warp file that merges normally, so every tree that carries the test also carries the file.

### D3 — The overview is inlined, background files stay pointers

Decision: each `contracts/stencils/pattern/` directive stencil keeps its instruction section (binding, wins over the rest of the prompt, violations are blocking) and replaces its "Read _lyx/PATTERN.md in full" bullet with a `{{.pattern_overview}}` marker.
`pattern.Directive` reads `PATTERN.md` at the warp worktree root and fills that marker with the file's content verbatim, then strips the banner as today.
The "read the background files under `pattern/` that touch your change" bullet stays a relative literal pointer.
The stencils' marker-free rule is lifted; their banners and `internal/pattern`'s doc ("Why the pointer stays relative") are rewritten to say the overview is inlined because it is guaranteed seen, saves a read turn per agent and stays cache-stable, and that background files stay pointers.
`stencilstore.Validate`'s shipped-vs-on-disk marker check covers the new marker unchanged.
Rationale: operator decision of 2026-10-04.

### D4 — Empty overview means no directive

Decision: a `PATTERN.md` that exists but holds only whitespace yields no directive (empty string, no error), instead of being active as today.
A read error on an existing file stays an error, as today's unreadable-file rule.
The existing pinned edge-case tests are updated to the new rule.
Rationale: with the overview inlined, an empty file would render a directive with nothing in it, and `stencil.Fill` rejects an empty required marker.
Bound: only whitespace-only content is treated as inactive; any other content, however malformed, is inlined, and format violations are the D2 test's job, not a runtime refusal.
An emptied loomyard PATTERN does not slip through, because the D2 checker reports an empty overview and an overview with no entry bullet.

### D5 — Two new directive roles: Discussion-Write and Bouncer judge

Decision: `internal/pattern` gains `RoleDesigner` (Discussion-Write: decisions must comply with the entries, or name the entry they change and why) and `RoleJudge` (Bouncer judge: a finding that cites a `PATTERN-*` entry violation is never downgraded below blocking on the judge's own say-so).
Each gets its own directive stencil in `contracts/stencils/pattern/`.
`loom-template-discussion.md` and `bouncer-template-judge.md` gain an optional `pattern_directive` marker, filled at their render sites the same way Plan-Write and Burler fill theirs.
The judge still never opens the artifacts; its directive is worded for judging findings, not for reviewing.
`internal/pattern`'s doc section "Why three roles, not one" is rewritten for five.
Rationale: operator asked for the directive on both agents; the existing three variants are worded for editing or forking and would ask these two agents to do something their prompts forbid.

### D6 — Removed (2026-10-04)

The landing carry of PATTERN into the parent pair, and its fabric operation, are removed: PATTERN is a warp file and lands through the ordinary merge (see Operator additions).

### D7 — `CONSTRAINTS.md` deleted, every reference repointed

Decision: `CONSTRAINTS.md` is deleted in this task.
`CLAUDE.md`'s first section becomes "`PATTERN.md` is authoritative", says lyx-spawned agents get the overview inlined, and that a new cross-cutting invariant goes into PATTERN in the same commit as the code.
Stencils that today say "read `CONSTRAINTS.md` at the repo root if present" (loom discussion, plan, rework; webster master and prefix-recovery) drop that clause and refer to the inlined PATTERN section instead.
`citation_enforcement_test.go`'s "if present" rule is replaced by a rule that no stencil names `CONSTRAINTS.md`.
`internal/websterengine/template_test.go`'s cold-recovery assertion moves from `CONSTRAINTS.md` to the directive.
Code comments, doc.go files, specs and docs that cite "`CONSTRAINTS.md`'s X Invariant" cite the entry name instead (`PATTERN-<name>`).
Enforcement-test function names stay as they are; the entry's background file names its tests.
Rationale: the operator said PATTERN replaces `CONSTRAINTS.md`; keeping both would leave two authorities.

### D8 — No `lyx pattern init` verb

Decision: no init verb.
The format is documented in `internal/pattern`'s doc, and another repo starts PATTERN by writing one file.
Rationale: YAGNI; creating a file needs no verb, and no other repo has asked for one.

### D9 — `manifest/` retired

Decision: the implementer adds a tier-3 board note through `lyx board upsert` for each listed Someday item not already on the board (`raddle`, `webster-parallel-execution`, `reed-multi-window`, `warp-visibility`, `baseline-drift-gate`, `config-overlay`, `fabric-reset-to-sha`, `finalize-discrepancy-doc`, `shedrecipe-capabilities`, `doctor`, `session-sync`), with a `[label]` brief prefix, the item's roadmap text, and the content of its `manifest/designs/<name>.md` when one exists and describes unbuilt design.
Design docs for items already on the board (`hardener`, `semantic-index`, `reed-mailbox` → `creel`) are appended to that entry's body under a `## Design` section.
The struck items are removed with the rest of `roadmap.md`.
Durable reference content moves into package docs:
- `manifest/HANDOFF.md`: the hub run loop goes to `internal/orchcli`'s doc; the rest is dropped.
- `hardener.md`: the text-review versus behaviour-review distinction and the respawn rationale go to `internal/treadleengine`'s doc.
- `fabric-windows-verification.md`: the Linux-only caveat goes to the `internal/fabricengine` and `internal/fslink` docs.
- `webster-parallel-execution.md`: the case study and the index-race rationale go to `internal/websterengine`'s doc.
- `review-finding-classification.md`: the convergence case study goes to `internal/burlerengine`'s doc.
- `curation-triage.md`: one line in `internal/boardengine`'s doc, "anyone adds notes, the orchestrator curates"; the automation is dropped.

Inbound links are fixed in: `internal/treadleengine/doc.go`, `internal/fabricengine/livestate_doc_test.go`, `contracts/specs/loom-plan-spec.md`, `docs/overview.md`, `crucible/README.md`, `crucible/review-prompt-template.md`, `CLAUDE.md`'s task completion rule and `.claude/agents/mill-done-gate-fixer.md`.
`manifest/` is then deleted.
`CLAUDE.md`'s "Docs land in the same commit" section and `docs/overview.md`'s Documentation Lifecycle say: an unbuilt design lives in its board entry's body; a built design lives in its package's `doc.go`.
Bound: the board writes are upserts by slug, so a rework re-run rewrites the same entries instead of duplicating them; the implementer touches no other board entry.
Rationale: operator decisions of 2026-10-01 to 2026-10-03, recorded on the board entry.

### D10 — Fabric-vocabulary scan beyond production Go

Decision: `TestEnforcement_FabricVocabulary` (in `internal/lyxcwd`) extends its bare `weft`/`warp` rule to `*_test.go` files outside the owner set and to every `.md`, `.yaml` and `.go` file under `contracts/`, `docs/`, `plugins/`, `crucible/` and `cmd/`, plus `CLAUDE.md` and `README.md`.
It skips `_lyx`, `.lyx`, `sandbox/` fixtures, and an explicit allowlist of paths, each with a one-line reason in the test: docs that describe fabric itself (fabric's own spec and shared-lib docs) and measurement reports under `docs/benchmarks/` and `docs/research/`.
The `weftname` import rule keeps its test-file exemption for `internal/lyxcwd/geometry_test.go`.
Existing hits outside the allowlist are reworded (`contracts/specs/refusal-spec.md`, the stencil claims test, `contracts/recipes/loom-recipe.yaml`, the ly-drive skill, crucible, and the `_test.go` files the scan flags).
Rationale: operator 2026-09-30, "nothing outside fabric should know them".
Bound: the allowlist is a closed list in the test; adding a path needs a reason line.

### D11 — Model-spec leftovers

Decision: `contracts/specs/llm-model-spec.md` shows every example in the short form (`model[effort]`) and states the short form as canonical; the long form stays parseable.
Every template comment's `docs/reference/model-spec.md` link points to `contracts/specs/llm-model-spec.md`.
`internal/landingshed/config_test.go`'s malformed-input fixtures are rewritten as malformed short-form specs that exercise the same failure paths.
Rationale: board entry; the parser is not changed, so existing configs keep loading.

### D12 — Agent role constants

Decision: every spawn site takes its role from a named constant declared in its own module (`loomcli` driver; `loomengine` discussion, plan, rework; `burlerengine`; friction; landing describe and conflict; `treadleengine` judge, triage, targeting; any other literal the plan finds).
No shared list; `agentname`'s role set stays open.
The Agent Name Invariant's PATTERN entry states that the spawning module owns its role names.
Rationale: operator 2026-10-02.

### D13 — ide hub-workspace follow-ups

Decision:

- `internal/fabricengine/hubworkspace.go` builds the prime path through `WorktreePath` instead of joining it by hand.
- `resolveSpawnTarget` logs a prime-resolution failure at WARN only when the spawn targets the prime; a task spawn logs it at debug, since only its title-bar color degrades.
- Removing the last pair no longer prunes `_portals/<anchor>`; only the pair's own entry under it goes, so the hub workspace never lists a missing folder.
  The emptied anchor directory stays in place; lyx never removes it, and a pair added under it later reuses it.
  The workspace folder list is built from existing entries only.
- Relative-path setting values spliced from the prime resolve against `_launchers/`; `vscode.BuildWorkspace`'s caller WARNs once per spawn, naming the keys, when a spliced setting's string value starts with `./` or `../`, and the `internal/vscode` doc states the limitation.

Rationale: the PR #301 review's four findings.

### D14 — `shedrun.Seed.Parent` retired

Decision: the `Parent` field and its read fallback are removed; parent comes only from the pair's origin record.
A seed file on disk that still carries a `parent` key keeps decoding (the key is ignored) and the run resolves its parent from the origin record; a missing origin `parent_worktree` behaves as it does today for a new run without one.
Rationale: PR-Gate note of 2026-10-02; the field is no longer written.
Bound: no run seeded before `aa67fd1d7` is live in the hub (to be confirmed by the plan, see Open risks).
A pre-`aa67fd1d7` seed that is still live resolves no parent and behaves as a new run without an origin `parent_worktree`.

## Constraints

- Runs alone and last on the board: it repoints `CONSTRAINTS.md` across most of the repo.
- `CLAUDE.md`'s semantic-line-break rule applies to every `.md` written, `PATTERN.md` and background files included.
- PATTERN files are tracked warp files at the repo root (`PATTERN.md`, `pattern/`), changed in the same commit as the code they govern.
- Fabric Vocabulary: code outside the owner set never names the two sides, including the extended scan's own messages where they leave fabric.
- Stencil paths in directive stencils stay relative literals (`pattern/`), never interpolated.
- `internal/pattern` stays a leaf (Pattern Leaf Invariant).
- No new configuration key.
- Plugins stay at `1.0.0`; the ly-drive skill edit ships through `update-plugins.sh` only when the operator runs it.

## Auto-mode assumptions

- The operator's 2026-10-04 PATTERN format on the board is final; no detail beyond it was asked.
- 320 characters per entry line and 16 KiB for the overview are adequate caps for loomyard's invariants after merging near-duplicates.
- Topic grouping and entry names are the plan's and implementer's call within D1's rules.
- The Bouncer judge benefits from a judge-worded directive; giving it the overview does not conflict with its never-open-the-artifacts rule.
- The implementer's agent can run `lyx board upsert` against the hub board.
- No run seeded before `aa67fd1d7` is still live.

## Open risks

- A run seeded before `aa67fd1d7` may still be live in the hub; the plan confirms none is before D14 removes the fallback.
- Inlining makes every directive-bearing prompt larger by the overview's size; the 16 KiB cap bounds it.
- Board writes from inside a run are not reverted if the PR is rejected outright; they are idempotent, not transactional.
- The D10 scan may flag many test files; the sweep size is unknown until the scan runs.

## Acceptance criteria

- `CONSTRAINTS.md` is gone; `grep -rn CONSTRAINTS.md` over the code tree finds nothing outside git history.
- `PATTERN.md` holds an entry for every invariant `CONSTRAINTS.md` held (merged entries count), and passes the checker.
- Checker fixture tests cover: bad name, duplicate name, two-line entry, over-long line, oversize overview, empty overview, overview with no entry bullet, unresolved link, link to another entry's file, orphan background file, and a clean overview.
- The real-file test fails when the module root has no `PATTERN.md`, never skips, and passes on this tree.
- `pattern.Directive` tests: overview content appears verbatim in every role's directive; whitespace-only `PATTERN.md` yields no directive; an unreadable file is an error; an absent file yields no directive; the background-file pointer stays the relative literal.
- Discussion-Write's and the Bouncer judge's rendered prompts carry the directive when PATTERN is active and omit it when not.
- `fabricengine`'s Pull result, `lyx fabric pull`'s envelope and their tests no longer carry a PATTERN residue.
- `citation_enforcement_test.go` fails a stencil naming `CONSTRAINTS.md`.
- `internal/websterengine/template_test.go` asserts the cold-recovery prompt carries the directive, not `CONSTRAINTS.md`.
- The extended vocabulary scan fails a fixture `.md` under `docs/` and a fixture `_test.go` outside the owner set naming `weft`, passes an allowlisted path, and passes on the swept tree.
- `manifest/` is gone; no file links into it; the listed board notes exist with their design content.
- No template comment names `docs/reference/model-spec.md`; `llm-model-spec.md` shows only short-form examples.
- `internal/landingshed/config_test.go` holds no long-form spec, and its malformed-input cases still hit the same failure paths.
- No spawn site passes a role as an inline string literal.
- The four D13 behaviours each have a test (prime-path construction, log level by spawn target, portal dir kept, relative-setting warning).
- An old seed carrying `parent` decodes and resolves its parent from the origin record.
- `go test ./...` and `go vet ./...` pass.

## Notes for the plan writer

- `internal/pattern/doc.go` sections to rewrite: "The active check is pure existence", "Why three roles, not one", "Why the pointer stays relative".
- The current directive call sites: `internal/websterengine/render.go`, `internal/loomengine/plan.go` and `rework.go`, `internal/burlerengine/engine.go`; the Bouncer judge renders in `internal/shedadapters/bouncer.go`; Discussion-Write renders in `internal/loomengine/discussion.go`.
- `internal/friction` already has a role-per-agent pattern (including an interview variant) that the new pattern roles can mirror.
- `PathspecFile`/`PathspecDir` are consumed today only by `fabricengine`'s PATTERN-residue scan in `pull.go`, surfaced by `internal/fabriccli/weft_verbs.go`.
- The vocabulary scan lives in `internal/lyxcwd/enforcement_test.go`.
- The manifest content-move list and inbound links are on the board entry (`lyx board get '{"slug":"pattern-init"}'`).

## Operator additions (2026-10-04)

Decision: PATTERN lives in warp, not weft.
Rationale (operator): weft is the lyx agents' scratchpad and every weft branch is deleted afterwards;
PATTERN is binding guidance for the code itself, so it is tracked code that changes in the same commit as the code it governs and lands through the ordinary merge.

1. Location: the overview is `PATTERN.md` at the repo root, next to `CLAUDE.md` and `go.mod`;
   background files are `pattern/PATTERN-<kebab-name>.md` at the repo root, next to `PATTERN.md`.
   An entry's background link is `pattern/PATTERN-<kebab-name>.md`, relative to `PATTERN.md`.
2. `internal/pattern` resolves the overview and the background directory from the warp worktree root it is told, not through `lyxdirs.LyxDirName`.
   `File`, `PathspecFile` and `PathspecDir` and the `doc.go` rationale change accordingly, and the leaf entry drops `lyxdirs` from `internal/pattern`'s import set if the package no longer needs it.
   The directive stencils' background-file pointer literal is `pattern/`.
3. D2: the real-file test checks the module root's `PATTERN.md` and never skips;
   the failure for a pair forked before PATTERN reached its parent, and its way-forward message, are dropped.
4. D6 is removed entirely, with its fabric operation, its acceptance criteria and its Open-risks item (the manual copy into the prime after landing).
5. D7: `CLAUDE.md`'s first section becomes "`PATTERN.md` is authoritative";
   a new cross-cutting invariant goes into PATTERN in the same commit as the code.
6. PATTERN's text files (`PATTERN.md` and every `pattern/PATTERN-*.md`) carry no tie to the tooling that develops the repo.
   They state rules for the code only, never how an agent, a run, a stencil, Claude or a development workflow uses them;
   the injection, the directive wording and the agent instructions stay in the stencils and `internal/pattern`.
   Naming loomyard's own code (packages, the `lyx` CLI, its types) is fine, since that is the code the rules govern;
   what is barred is text addressed to agents or describing the dev tooling's process.
   Review-enforced, not a format-test rule.

Consequence (Plan-Write, approved by the operator through the orchestrator): `fabricengine`'s PATTERN-residue report scans weft commits for the PATTERN paths, which no longer live in the weft, so it can only ever report nothing.
It is retired: `PullResult.PatternResidue`, `PatternResidueEntry`, `patternResidueCommits`, `parsePatternResidueRecords`, the residue field in `lyx fabric pull`'s envelope (`internal/fabriccli/weft_verbs.go`) and their tests are removed.
