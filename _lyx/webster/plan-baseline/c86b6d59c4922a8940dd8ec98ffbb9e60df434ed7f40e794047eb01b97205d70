# Card 13 — `lyx loom decision add`

**Create:**
- `internal/loomcli/decision.go`
- `internal/loomcli/decision_test.go`

**Edit:**
- `internal/loomcli/cli.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/review_test.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Uses:**
- `internal/discussionparser#AppendDecision`
- `internal/discussionparser#AddedDecision`
- `internal/loomcli/review.go`
- `internal/loomcli/circling.go`

**Intent:**
D8: a new verb `lyx loom decision add [<slug>] --by parent|operator --title <text> --decision <text> --rationale <text>` appends one decision to the decision record, re-runs the discussion check, and commits the record.

Shape (`PATTERN-cli-cobra`): a `decision` group with `add` under `loom`, each with `Short` and `Long`, JSON envelopes through `output`, mounted in `cli.go`'s `AddCommand` list.
Like `review` and `circling`, the group carries its own `PersistentPreRunE`; the loom parent's pre-run skip in `cli.go` adds `decision` beside them, and the loom root's `Long` text names the group.
The pre-run resolves the target worktree with `resolveReviewTarget` (slug or self-addressing task worktree, refusing from the prime without a slug), and checks no caller identity.

`decision.go` holds a `decisionVerb(out, slug, deps, input)` body over injected deps (status reader, append, commit, clock), so every refusal is reachable from an untagged test.
It appends through `discussionparser.AppendDecision` with the date from the clock in UTC, then commits.
The commit goes through the discussion commit seam Discussion-Review already uses: extract `wire`'s `CommitDiscussion` closure body into a named function over a `*lyxcwd.Location` that both `wire` and the verb call, keeping its pathspec and message.
The success envelope names the slug, the record path and the appended heading.

Refusals, each naming its way forward, with a `refusal-spec.md` row under a `decision:` prefix and a reaching test (`PATTERN-refusal-way-forward`):
- no decision record exists: the run has not finished Discussion; way forward: let Discussion-Write write the record, or start the run with `lyx loom start`;
- the run's status names `Discussion-Write` as current producer in state `running`: the writer owns the file; way forward: message the Discussion-Write session with the decision instead;
- an empty `--title`, `--decision` or `--rationale`: way forward names the flag to fill;
- a `--by` other than `parent` or `operator`: way forward names the two values;
- a check finding after the append: the record is restored; way forward: fix the named section and re-run;
- from the prime with no slug, and an unknown slug: the shared target-resolution refusals, with rows like the `circling:` ones.

What this lets through: any shell addressing the run can add an entry to the answer key, the Plan-Write agent included, and `--by` is an unverified label.
It is bounded by being append-only, one entry per call, under an `Added after Discussion` heading with the claimed label and date, in a committed diff; it never routes the run, never touches the support log or the plan, and an approved plan is not re-planned by it.
The file header states this bound, as `circling.go`'s does.

Docs: `docs/overview.md`'s loom module line lists `decision add`, and the loom verb paragraphs describe it beside `validate-discussion`.

Tests in `decision_test.go`: a clean add over a temp record appends the entry, leaves the prior text byte-identical, calls commit once and returns the heading; each refusal above returns its message and leaves the record and commit untouched.
`review_test.go`'s group loop gains `decision` for the prime and unknown-slug refusals.

**ImpactSummary:** One new verb group; `wire`'s discussion commit closure becomes a named function with unchanged behaviour.

**Commit:** 13: lyx loom decision add records a post-Discussion decision
