## Goal

```markdown
## Seen (operator and hub orch, 2026-10-31)

Comments in the Go source go stale as the code moves on.
Two found today: `internal/fabricengine/weftwiring.go` still speaks of a future `_lym/raddle/` squash-merge-back, though raddle will never be built; and many comments spelled the `_board` path instead of naming its constant (the rename to `_bolt` in `bolt-dir-rename` has since replaced the spelling itself).
There are surely more, in every package, against `scribe:prose`'s rule never to pin a perishable specific.

## Goal

A comment-cleanup campaign over the whole repository.

- A worker fixes each comment or text that no longer matches the code, names something gone or planned and dropped, or pins a perishable specific: a count, a list of current callers, a path or name spelled where a constant or identifier should be named.
- Workers change comments and text only, never code.

## Design (operator, 2026-10-31)

- One darn run, whose agent orchestrates the whole campaign: it splits the work and spawns the workers, many in parallel; not one darn per package, which would be far too many runs.
- Scope: Go comments and docstrings, each package's `doc.go`, the stencils and the docs.
- Models by scope: Haiku 5.5 takes the comments and docstrings of code files; `doc.go`, stencils and docs need a stronger model, Sonnet 5.5, as the `docs` profile's writer is.
- Every Haiku 5.5 worker is scoped to modules so that its context stays under 100k tokens, since below that Haiku 5.5 is cheap (operator, 2026-11-04): one package per worker, a large package split by file groups, small packages grouped while the sum stays under the bound.
  The writer sizes each worker's scope before it spawns it.
- No review step of its own: most comments are assumed sound, and this run is itself the check on them; the darn's usual Code-Review is enough.
- The run may start beside `tier1-test-revision`, which only writes a report (operator, 2026-11-06).

## Workers (hub orch, 2026-11-03)

The writer spawns its workers as subagents through its own Agent tool, with the tool's model override set to `haiku` or `sonnet` by the scope above; no lym verb is needed.
Each worker gets its scope from the bound above, edits only comments and text in it, and reports what it changed; the writer commits.

Related: note `hardener` is another autonomous campaign run by an orchestrating agent.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-06): Size Haiku workers by peak context, under 100k tokens

Decision: Before the fan-out, cut each Haiku worker's scope so its peak context stays under 100k tokens. The three pilot Haiku workers peaked at 218,860, 219,497 and 269,342 tokens, so the 200 KB scope bound does not hold. Measure a worker by its peak context in its transcript, not by the size of its source files, and check the first workers of the new size before starting the rest.

Rationale: The task body sets the 100k bound because Haiku 5.5 is cheap below it (operator, 2026-11-04). What a worker reads to check a comment counts toward its context, so source size alone underestimates it.

### Added after Discussion (operator, 2026-11-06): A worker over 100k tokens is a cost, not a failure

Decision: Size Haiku workers to stay under 100k tokens, but a worker that goes over it now and then needs no rerun and no stop: keep its work and go on.

Rationale: Operator, 2026-11-06: going over is no crisis, but Haiku 5.5 is very cheap under 100k context and much dearer above it.

### Added after Discussion (operator, 2026-11-07): No Code-Review, and no code may change

Decision: This run skips Code-Review: code_max_bounces is 0. Since no review follows, the writer itself proves the rule that code stays as it was: before the run leaves Darn-Write, the whole diff of .go files against the branch point holds only whole-line comment changes, with every other changed line reverted, and the gate is green.

Rationale: Operator, 2026-11-06/07: a Code-Review of a diff this size is not realistic, and the code itself must not change.

### Added after Discussion (parent, 2026-11-07): Goal: all docstrings and doc files follow the five scribe skills

Decision: Operator, 2026-11-07: the campaign's goal is wider than stale comments. Many workers in this repository's history did not load the scribe skills, so many docstrings are poor. All docstrings, doc.go files and doc files shall be brought in line with scribe:prose, scribe:code-quality, scribe:testing, scribe:golang-comments and scribe:golang-testing, which state in detail what is written. Every agent that writes in this run, each worker included, loads all five with the Skill tool before it reads its scope. A true comment that breaks these skills is rewritten, so the worker brief's 'never restyle, reword, reflow, re-wrap, shorten or improve' rule and 'fix only what is defective' go, and every file already covered is covered again. The fixes made so far stay. Code still must not change, the comment-only proof stands, stencils keep the brief's limits, and the aim of about 75k context stands. Start no new worker on the old brief: change .scratch/worker-brief.md first, run a few scopes as a pilot, and check in their transcripts that all five skills loaded and what the context peaks were before fanning out.

Rationale: The operator's point with this run was prose that follows the skills; the entry's body named only stale comments, and all 502 workers so far loaded no skill.

### Added after Discussion (parent, 2026-11-07): Three skills, not five

Decision: Operator, 2026-11-07: this corrects 'Goal: all docstrings and doc files follow the five scribe skills'. Only the skills that dictate prose, docstrings and doc.go files are needed: scribe:prose, scribe:golang-comments and scribe:code-quality. scribe:testing and scribe:golang-testing are not loaded. The rest of that decision stands. The operator talks to this run directly.

Rationale: The operator narrowed the list to the skills that govern this text.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
