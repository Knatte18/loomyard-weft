# Decision record — pattern-cleanup

## Goal

Cut `PATTERN.md` from about a hundred entries to about thirty, so the overview every spawned agent's prompt inlines shrinks well below its headroom line, without losing a rule.

## Scope

- Rewrite `PATTERN.md`: merge related entries into one survivor each, move package-local rules into their owning package's `doc.go`, drop rules another enforced home already states.
- Merge each absorbed entry's background file into its survivor's background file, move a moved entry's background into the owning `doc.go`, and delete a dropped entry's background file.
- Sweep every live citation of a removed ID — Go code and comments, `contracts/`, `crucible/`, `docs/` outside `docs/research/` and benchmark reports — to the survivor's ID, the owning package doc, or nothing.
- Out of scope: `internal/pattern`'s checker and caps, the `PATTERN-` citations in measurement reports under `docs/research/` and `docs/benchmarks/`, and test corpus files under `testdata/`.

## Decisions

### Merge groups follow the board entry's review, with four placements of my own

Decision: Twenty-two survivors absorb related entries as the review lists them, except that `hub-store-gc` joins `fabric-git` rather than `gitrepo-client-boundary`, and the three entries added after the review join a group: `row-entry-gate` into `shed-recipe-registry`, `gate-carry` into `gate-slot`, `tier-tags` into `test-speed`.
Rationale: `hub-store-gc` binds `fabricengine/housekeeping.go`, the fabric engine's own file, not `internal/gitrepo`; each later entry restates one facet of its survivor's subject.

### Background files merge by section

Decision: A survivor's background file keeps its own text and gains one `##` section per absorbed entry, carrying that entry's background text with its headings demoted one level; an absorbed entry with no background file contributes its overview rule as a bullet.
Rationale: `pattern.Check` requires every `pattern/` file to be named by exactly one entry, and no rule's detail is lost.

### Moved rules land in the owning package's doc

Decision: Twenty-one package-local entries move into a section of the owning package's `doc.go`, condensed to the rule, its bounds and its enforcement: `hub-wide-config` to `configreg`, `sparse-hub-base` to `hubreconcile`, `hub-suffix` to `lyxcwd`, `mutation-record` to `fabricengine`, `parent-lock` to `parentlock`, `shed-producer-seam` and `transient-stop` to `shedengine`, `recipe-owns-failures` to `landingshed`, `failure-stop-visible` to `shedverbs`, `driver-choice-single-site` to `loomcli`, `treadle-runner-seam` to `treadleengine`, `plan-generation` to `loomshed`, `glyph-conversion-chokepoint` to `planglyph`, `batcher-registry` to `batcher`, `review-round` to `burlerengine`, `agent-name` to `agentname`, `friction-capture` to `friction`, `pane-binary-resolution` and `wait-mark-display-only` to `reedengine`, `sandbox-coverage` and `dev-prod-binary-separation` to `tools/sandbox`.
Rationale: An agent changing such a rule's code reads that package's doc anyway, and each is either test-enforced with a message naming its owner or touched only from its package.

### Four entries are dropped

Decision: `quarry-cgo`, `markdown-link-integrity`, `comment-line-breaks` and `documentation-lifecycle` leave PATTERN with their background files.
Rationale: `CLAUDE.md` states the cgo build and the documentation lifecycle, `docslink_test` fails with a clear message on a broken docs link, and `lyx loom lint-comments` runs at every card gate.

### Citations of a removed ID are swept, research records are not

Decision: A citation of an absorbed ID becomes its survivor's ID; one of a moved ID names the owning package's doc; one of a dropped ID names the enforcing test or `CLAUDE.md`, or is removed.
Reports under `docs/research/` and `docs/benchmarks/` that record a past run, and `testdata/` corpus files, keep their text.
Rationale: No test checks that a cited ID exists, so a stale citation in live code or a contract misleads silently; a dated report records what was true at its run.

## Constraints

- `PATTERN-markdown-link-integrity` until it is dropped: every inline link in a `docs/` file resolves.
- `pattern.Check` stays green: one background file per linked entry, no orphan, no entry line over `MaxEntryLineChars`.
- `PATTERN-comment-line-breaks`: every Go comment written into a `doc.go` keeps one sentence per line.
- A moved rule keeps its enforcement: no test or scan is removed.

## Auto-mode assumptions

- The review's merge, move and drop lists are the operator's intended shape; this run applies them without a further interview.
- Section headings in the overview may be regrouped to fit the survivors.

## Open risks

- A moved rule is no longer in every spawned agent's prompt, so an agent outside the owning package can break a review-only rule without seeing it.
- A merged overview line names several clauses tersely; an agent may miss a clause that only the background file spells out.

## Acceptance criteria

- `PATTERN.md` holds about thirty entries and stays well under `HeadroomBytes`.
- `pattern.Check` over the repository reports no finding.
- No live file outside `docs/research/`, `docs/benchmarks/` and `testdata/` cites a removed PATTERN ID.
- Every moved rule appears in its owning package's `doc.go`.
- The scoped gate test run is green.

## PATTERN changes

- `PATTERN-cwd-resolution` — changes: Resolving a cwd or worktree root: only `internal/lyxcwd`; an engine is handed absolute paths and never imports it; hub-level containers and the hub workspace file stay reachable from the hub only.
- `PATTERN-durable-vs-ephemeral-state` — changes: Adding file: tracked under `_lyx`, never-tracked under `.lyx`, same subpath; only `internal/lyxdirs` names either, bar a `tools/` script's variable block.
- `PATTERN-config-layers` — changes: Loading config: only `configengine` reads, stacks, locates and judges layers against the template; a caller adopts one of strict `Load` or `LoadOrTemplate`, bar `runprofile`; after a binary change, only `hubreconcile.Ensure` reconciles.
- `PATTERN-refusal-way-forward` — changes: Adding webster, shed or loom refusal: message names the way forward and a test reaches it; no refusal, stencil or spec names a step agents' settings deny. (test)
- `PATTERN-fabric-git` — changes: Running git on warp or weft: only `internal/fabricengine`, in-process; destroying only in its `destroy.go`, hub writes via a hub `os.Root`, store gc only in `housekeeping.go`.
- `PATTERN-push-both-sides` — changes: Pushing run records from Go: code branch goes along; a push refused after retry leaves a failed-push record and creel notice. (test)
- `PATTERN-pair-teardown` — changes: Tearing down a pair: only via `internal/pairteardown`; a producer creating or destroying a task worktree never runs inside it. (test)
- `PATTERN-github-auth` — changes: Calling GitHub: auth only via `internal/githubclient`, no other production package runs `gh`, and only `lyx selfreport create` files an issue. (test)
- `PATTERN-gitrepo-client-boundary` — changes: Running git: `gitexec.Run`/`runChecked`, raw only at pinned `//gitexec:raw` sites; in `internal/gitrepo`, local reads via `readGoGit` and `dotgit`; transients via `.git/info/exclude`, never `git add -f`. (test)
- `PATTERN-owner-machine` — changes: Starting, resuming or landing a run or acting on wake: `fabricengine`'s origin reconcile first, then only where no other machine owns it.
- `PATTERN-shed-recipe-registry` — changes: Adding producer or recipe row: one `Constructor` map in `internal/shedrecipe`, bar `Implementer`'s; a row's value looked up by key, a side action a `Command` row, a config-off row its `entry_gate`.
- `PATTERN-shed-driver` — changes: Driving a shed run: generic verb bodies only in `internal/shedverbs`, the step-until-stop loop only in `internal/sheddriver`, run paths, ids and `Seed` only in `internal/shedrun`. (test)
- `PATTERN-gate-slot` — changes: Running gate build, test or verify: in held `internal/gateslot` slots, verify via `verifytree.Verify`, a tier skipped only on `lyx gate test`'s green record; a gate and its self-check call one function. (test)
- `PATTERN-sole-parsers` — changes: Parsing plan, discussion, summary, recipe or halt note, or classifying a plan ref: only its sole parser. (test)
- `PATTERN-stencil-ownership` — changes: Reading prompt or spec: at call time via `internal/stencilstore`, never embedded; an instruction file points at another producer's format contract, never restating it.
- `PATTERN-shuttle-provider-seam` — changes: Touching a provider or shuttle run: specifics only under `internal/shuttleengine/<engine>`, model-specs via `NewModelChoice`, a strand stopped via the stop verb, a negative finish after `allOutputFilesExist`. (test)
- `PATTERN-shell-guard-table` — changes: Guarding a shell command or building a pane-shell string: only `internal/shellguard`'s table, every engine rendering every row, and `internal/shell`, importing only stdlib.
- `PATTERN-pane-text-injection` — changes: Typing into an agent pane: reed's guarded paste-submit, an interrupt key only via shuttle's guards, a covered session held; the orch pane and creel journal only via creel. (test)
- `PATTERN-parent-directive` — changes: Writing a spawned role's stencil: it renders the parent and edit directives; its skills and color go on the launch spec, never asked of the agent. (test)
- `PATTERN-spawn-observability` — changes: Starting OS process from `lyx`: log the spawn via `internal/logger`; logging in a loop or poll follows its Level policy.
- `PATTERN-test-speed` — changes: Writing or running a test: via `lyx gate test` by gate key; one `tierN` tag, the lowest that fits; it pins behavior no test covers; injected clocks, `t.Parallel` by default. (test)
- `PATTERN-test-isolation` — changes: Testing package: hermetic git and `tmuxkit.Main`; shared support in one `internal/testkit/<kit>/`; hub fixtures via `hubforge`. (test)
- `PATTERN-told-geometry` — removes
- `PATTERN-hub-containment` — removes
- `PATTERN-lyxdirs-single-declarer` — removes
- `PATTERN-config-strictness` — removes
- `PATTERN-template-overlay` — removes
- `PATTERN-binary-change-reconcile` — removes
- `PATTERN-no-denied-recovery` — removes
- `PATTERN-fabric-destruction-chokepoint` — removes
- `PATTERN-fabric-write-containment` — removes
- `PATTERN-hub-store-gc` — removes
- `PATTERN-push-failure-visible` — removes
- `PATTERN-batten-bookend` — removes
- `PATTERN-agent-filed-issues` — removes
- `PATTERN-gogit-read-helper` — removes
- `PATTERN-gitexec-checked-call` — removes
- `PATTERN-never-force-add` — removes
- `PATTERN-origin-reconcile` — removes
- `PATTERN-recipe-setting-lookup` — removes
- `PATTERN-command-row` — removes
- `PATTERN-row-entry-gate` — removes
- `PATTERN-shed-verb-set` — removes
- `PATTERN-shed-run-directory` — removes
- `PATTERN-gate-self-check-parity` — removes
- `PATTERN-verified-tree` — removes
- `PATTERN-trimpath-safe-root` — removes
- `PATTERN-gate-carry` — removes
- `PATTERN-ref-shape-registry` — removes
- `PATTERN-producer-pointer-rule` — removes
- `PATTERN-completion-signal` — removes
- `PATTERN-model-choice-mapping` — removes
- `PATTERN-shuttle-stop` — removes
- `PATTERN-shell-mechanics-seam` — removes
- `PATTERN-creel-store` — removes
- `PATTERN-orch-pane-single-writer` — removes
- `PATTERN-pause-holds-prompts` — removes
- `PATTERN-guarded-interrupt` — removes
- `PATTERN-role-skills-typed` — removes
- `PATTERN-edit-directive` — removes
- `PATTERN-spawn-color` — removes
- `PATTERN-log-levels` — removes
- `PATTERN-test-tiers` — removes
- `PATTERN-test-economy` — removes
- `PATTERN-tier-tags` — removes
- `PATTERN-testkit` — removes
- `PATTERN-hubforge-fixtures` — removes
- `PATTERN-hub-wide-config` — removes
- `PATTERN-sparse-hub-base` — removes
- `PATTERN-hub-suffix` — removes
- `PATTERN-mutation-record` — removes
- `PATTERN-parent-lock` — removes
- `PATTERN-shed-producer-seam` — removes
- `PATTERN-transient-stop` — removes
- `PATTERN-recipe-owns-failures` — removes
- `PATTERN-failure-stop-visible` — removes
- `PATTERN-driver-choice-single-site` — removes
- `PATTERN-treadle-runner-seam` — removes
- `PATTERN-plan-generation` — removes
- `PATTERN-glyph-conversion-chokepoint` — removes
- `PATTERN-batcher-registry` — removes
- `PATTERN-review-round` — removes
- `PATTERN-agent-name` — removes
- `PATTERN-friction-capture` — removes
- `PATTERN-pane-binary-resolution` — removes
- `PATTERN-wait-mark-display-only` — removes
- `PATTERN-sandbox-coverage` — removes
- `PATTERN-dev-prod-binary-separation` — removes
- `PATTERN-quarry-cgo` — removes
- `PATTERN-markdown-link-integrity` — removes
- `PATTERN-comment-line-breaks` — removes
- `PATTERN-documentation-lifecycle` — removes

## Notes for the plan writer

- `lyxcwd`'s doc already owns the suffix's declaration site, so `hub-suffix`'s section lands there beside it.
- A survivor without a background file of its own, `shed-recipe-registry`, takes the absorbed `recipe-setting-lookup` file as its base.
