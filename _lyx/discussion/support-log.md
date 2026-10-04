# Support log — pattern-init

## Interview

The session ran autonomously; no operator answered.
Every design point was settled from the board entry, the codebase and recent commits, and is recorded as an auto-pick in the Question ledger.

1. Read the board entry: the operator's 2026-10-04 PATTERN format, five folded-in entries, the manifest strike list and the `seed.Parent` note.
   Read the board list: no other entry plans PATTERN, manifest or role-constant work; `convergence-plan` owns the model trial; `hardener`, `semantic-index` and `creel` already exist as notes.
2. Explored `internal/pattern`: the directive is a pointer today, pure-existence active check, three roles, marker-free stencils.
   The webster prefix-recovery, plan, rework, master and burler templates already carry `pattern_directive`; Discussion-Write and the Bouncer judge do not.
3. Found that `_lyx` is the weft checkout and that `main-weft` never advances on landing (fabric doc).
   PATTERN written in this pair would therefore never reach `main`; this produced D6, the landing carry.
4. Found no Go test reading `CONSTRAINTS.md`'s content; only comments and the citation test name it, so deletion is a repoint, not a test migration.
5. Located the vocabulary scan in `internal/lyxcwd/enforcement_test.go` (production Go plus `internal/**/*.md`) and listed non-Go hits in `contracts/`, `docs/`, `plugins/`, `crucible/`.
6. Checked the model-spec leftovers: the dead link appears in every config template's comments.
7. Checked the ide follow-up sites: `resolveSpawnTarget`'s single WARN, and `BuildWorkspace` splicing settings verbatim.

## Rejected alternatives

- **PATTERN in the code tree instead of `_lyx/`**: avoids the landing carry, but contradicts the operator's "weft, invisible to the code repo" and `internal/pattern`'s fixed path.
- **Copying PATTERN to the parent by hand after every landing**: no code, but every future invariant would be lost the first time someone forgets; `CLAUDE.md` makes invariants land with their task.
- **Merging PATTERN three-way at landing**: weft is never a merge participant by design; a refusal on divergence is simpler and divergence is rare.
- **Format checker only as a Go test with the rules inline**: the rules could not be fixture-tested apart from loomyard's own file.
- **A `lyx pattern check` verb**: nobody asked; the test runs under every verify.
- **Real-file test skipping whenever `PATTERN.md` is absent**: a fabric worktree would pass silently with PATTERN gone.
- **Validating the format at render time and refusing**: would stop runs over a formatting slip; the test already guards loomyard.
- **Keeping empty `PATTERN.md` active**: renders an empty directive and trips `stencil.Fill`'s non-empty marker rule.
- **Reusing `RoleReviewFix` for the Bouncer judge and `RoleImplementer` for Discussion-Write**: their wording tells the agent to fix or edit, which those prompts forbid.
- **Keeping `CONSTRAINTS.md` as a redirect stub**: two files for one authority; stencils naming it would keep pointing at nothing useful.
- **A `lyx pattern init` verb**: one file write needs no verb.
- **Rewriting relative-path setting values when splicing into the hub workspace**: a heuristic over arbitrary extension keys that can corrupt a non-path value; a warning plus a documented limitation is honest and bounded.
- **Writing the hub workspace file into the prime's anchor dir**: relative settings would resolve, but the file would sit in the prime's tree and dirty it.
- **Leaving board notes for the hub to add after landing**: splits the task's own scope across actors; upserts by slug are idempotent.
- **Dropping the long-form model-spec grammar**: breaks existing configs for no gain; the task asked only for the short form in docs and fixtures.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Where does the format test live, given PATTERN sits in `_lyx/`? — A checker in `internal/pattern` with fixture tests, plus a real-file test that skips only when `_lyx/` is absent (D2).
- Q2 (auto-pick): Size caps? — 16 KiB overview, 320 characters per entry line (D2).
- Q3 (auto-pick): How does an empty `PATTERN.md` behave once inlined? — No directive (D4).
- Q4 (auto-pick): Which directive wording do Discussion-Write and the Bouncer judge get? — Two new roles, `RoleDesigner` and `RoleJudge` (D5).
- Q5 (auto-pick): How does PATTERN reach `main` when weft never merges? — Finalize carries the PATTERN paths into the parent, refusing on divergence (D6).
- Q6 (auto-pick): Keep a `CONSTRAINTS.md` stub? — No, delete and repoint (D7).
- Q7 (auto-pick, asked by the board): Is a `lyx pattern init` verb wanted? — No (D8).
- Q8 (auto-pick): Who writes the Someday board notes? — The implementer, via `lyx board upsert` (D9).
- Q9 (auto-pick): Where do design docs of items already on the board go? — Appended to those entries under `## Design` (D9).
- Q10 (auto-pick): Allowlist or full sweep for the vocabulary scan? — Sweep, with a closed allowlist for fabric's own docs and measurement reports (D10).
- Q11 (auto-pick): Does the long-form model spec stay parseable? — Yes; only docs and fixtures change (D11).
- Q12 (auto-pick): Prime-resolution WARN on task spawns? — Debug for task spawns, WARN for the prime (D13).
- Q13 (auto-pick): Relative-path settings in the hub workspace? — Warn and document; no rewriting (D13).
- Q14 (auto-pick): Is it safe to remove `seed.Parent` now? — Yes, with old seeds still decoding; the plan confirms no pre-`aa67fd1d7` run is live (D14).
- Q15 (open, for the hub at landing): this task lands on the installed `lyx` without the carry, so the hub copies PATTERN into the prime's `_lyx/` by hand once (Open risks).
