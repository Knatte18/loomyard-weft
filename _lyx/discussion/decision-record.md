# Decision record — intake-own-repo

## Goal

`lyx board intake` reads, comments on and closes the issues of the repository the hub works on, never the hard-coded `Knatte18/loomyard`.
Selfreport stays the LoomYard-only bug filer.
The orch role stops running intake on its own and stops reviewing parent-review and PR-Gate requests.

## Scope

- Intake's GitHub side (list open issues, get one, list its comments, comment and close) moves out of `internal/selfreportengine` into a new package of its own.
- `lyx board intake list|import|close` resolves the hub's own warp repository and works against it.
- `internal/selfreportengine` keeps only `CreateIssue` and its helpers, still filing to `Knatte18/loomyard`.
- The orch role stencil (`contracts/stencils/orch/orch-template-role.md`): no inbox check at wakes or landings, intake only on the operator's word with the fork procedure kept, a finding in another repository filed to that repository's own issues, and Parent-review, PR-Gate and the hand-on fork bullets rewritten so the orch puts the alignment agent's open question to the operator instead of reviewing.
- Docs that say intake reads the selfreport inbox: `internal/boardcli`, `internal/boardengine` and `internal/selfreportengine` docs, `docs/overview.md`, and the `ly:board` skill.
- Out of scope: the hub-wide orch rules file on Bolt; see the decision on the override sections.

## Decisions

### Intake's GitHub side lives in a new package `internal/hubissues`

A `Repo` value carries an authenticated client and the owner and name of one repository; its methods are the four intake calls.
`Open(remoteURL)` parses the URL with `githubclient.ParseOwnerRepo` and builds the client with `githubclient.New`.
Rationale: intake never goes through selfreport, so its calls need a home that is neither selfreport nor the CLI layer, and a repository-bound value replaces the package-level seam, so the tests run in parallel.

### The hub's repository comes from its warp binding, else its warp origin

A new `fabricengine` function returns the `.lyx-warp` binding at the hub's board dir when one is recorded, and otherwise the `origin` remote URL of the location's warp worktree.
Rationale: the binding is the hub's recorded statement of which warp repository it pairs with; the origin covers a hub that predates the binding.
Git runs on warp only in `fabricengine`, so the origin read lives there.
Bound: a URL that is not a GitHub URL is refused with a message naming the URL; no fallback repository exists.

### `lyx board intake` resolves the repository only when an intake verb runs

The board command's pre-run keeps the resolved location, and the intake verbs open the repository from it.
Rationale: every other board verb works offline and must not pay for, or fail on, the repository resolution.
The detached sync child, which passes `--board-path`, never runs intake; an intake verb with no resolved location is refused.

### The hub-wide orch rules' override sections stay until after landing

The board entry asks that the "Intake only on the operator's word" and "No parent reviews" sections of `_board/_lyx/orch/rules.md` be removed in the same change.
That file lives on Bolt and is written only through `lyx orch rules`, so no task-branch commit can carry it, and removing it before the new role is deployed reinstates the old behaviour.
The parent (`ly:orch`) decided by message on 2026-10-21: the task branch carries the code, stencil and skill changes only, and the orch removes the two sections with `lyx orch rules` after this task lands and deploys.

### The orch puts an escalated review's open question to the operator

Parent-review: on a request, the orch reads only the brief's "Why it did not settle the review" line, puts that question to the operator in plain words, and submits the operator's answer with the brief's `lyx loom review` command; it never reviews and never forks a review.
PR-Gate: the same, with the reason the awaiting notice carries and `lyx loom approve` or `lyx loom reject <review-file>`.
The arbiter hand-on and the peer hand-on bullets: the orch reads the envelope's question and puts it to the operator, instead of forking to settle it.
Rationale: the alignment agent is the dedicated reviewer; a design question it could not settle is the operator's.
Bound: the orch still writes the review file a reject needs, holding the operator's answer and nothing of its own.

### Intake runs only when the operator asks

The inbox check leaves every wake and the `landed` notice; the intake fork procedure stays as the section the orch follows when the operator asks for intake.
A bug in lyx still goes to `lyx selfreport create`; a finding in another repository goes to that repository's own issues.

## Constraints

- `PATTERN-github-auth`: GitHub auth only through `internal/githubclient`.
- `PATTERN-fabric-git`: the origin read on warp happens inside `internal/fabricengine`.
- `PATTERN-cwd-resolution`: the board command keeps using `lyxcwd.Resolve` once; the new package derives no paths.
- `PATTERN-test-speed` and `PATTERN-test-economy`: the moved intake-call tests move with the calls and gain no duplicate; tests run in parallel now that no package-level seam is swapped.
- `PATTERN-tolerant-records`: no record format changes.
- CLAUDE.md "Docs land in the same commit": the new package's `doc.go` and `docs/overview.md`'s module table land with the code.

## Auto-mode assumptions

- The binding wins over origin when both exist and differ, since the binding is the recorded pairing.
- `lyx board intake` output keeps its shape; no `repository` field is added.
- The parent-review brief stencil stays as it is; the orch reads one line of it.

## Open risks

- A hub whose warp URL is a local path or a non-GitHub host can no longer run intake at all, where it used to read `Knatte18/loomyard`; the refusal names the URL.
- Until the orch removes the hub-wide override sections, they restate the new role; they agree with it, so no conflict arises.

## Acceptance criteria

- `lyx board intake list`, `import` and `close` request the paths of the repository they were opened on, not `Knatte18/loomyard`.
- The hub repository resolution returns the binding when recorded, the warp origin otherwise, and an error when neither exists.
- `internal/selfreportengine` exports no intake call and no longer documents intake; `CreateIssue` still files to `Knatte18/loomyard`.
- The orch role stencil holds no step that runs `lyx board intake` at a wake or landing, and no step that forks or performs a parent-review or PR-Gate review.
- The `ly:board` skill and the board docs describe intake as reading the hub's own repository.
- The intake call behaviours the old inbox tests covered (pagination, pull request exclusion, comment order, comment-then-close, failed close naming the posted comment, unresolvable token) stay covered.
