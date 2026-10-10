# Card 8 — pattern-overview-room

**Create:**
- `pattern/PATTERN-batcher-registry.md`
- `pattern/PATTERN-shuttle-stop.md`
- `pattern/PATTERN-parent-directive.md`

**Edit:**
- `PATTERN.md`

**Intent:** Make room in the `PATTERN.md` overview for card 9's and card 11's shed-verb-set line and card 12's `PATTERN-failure-stop-visible` line, under the decision record's PATTERN overview cap constraint and the Shared Decision `pattern-overview-cap`.
Eight overview lines are shortened; every clause a line drops is held in its entry's background file, so no invariant is dropped and the background is where a reader finds the detail.
The lines become, verbatim:

- `PATTERN-review-round` — "Running a review and fix round: review on disk before any target is touched, every finding fixed or disputed with evidence, converged on a judge verdict." with its background link; the background already states the false-premise evidence rule and the round-2 fresh-key carry to `## Open risks`.
- `PATTERN-batcher-registry` — "Choosing webster's execution unit: the batch the active `batcher.yaml` profile derives through `internal/batcher`'s registry, recorded in `state.json` at first init." with a link to the new background file, which states that the kind is chosen by `internal/batcher`'s registry and the profile by `batcher.yaml`'s `active:`, and that the recorded partition is replaced only by a rebaseline.
- `PATTERN-agent-name` — "Forming or parsing an agent name: only `internal/agentname`, formed once by reed; the spawning module owns its role names." with its background link; the background already states the two name grammars and that a role name is a constant of the spawning module.
- `PATTERN-shuttle-stop` — "Stopping a shuttle run's strand from Go outside `internal/shuttleengine`: only through shuttle's stop verb, which settles the run's record first." with a link to the new background file, which states that `loomcli`'s removal of the loom driver's strand and `orchcli`'s of the orch strand are exempt, as no attach probe or Wait reads their records.
- `PATTERN-parent-directive` — "Writing a spawned role's top-level stencil: it renders the parent directive, and no stencil tells an agent to ask the operator. (test)" with a link to the new background file, which states that the discussion role's interactive questions come from the `{{.mode_rules}}` marker and that the orch stencils are exempt.
- `PATTERN-gogit-read-helper` — "Reading through go-git in `internal/gitrepo`: only through `readGoGit`, which retries a read once after a repack. (test)" with its background link; the background already states that `readGoGit` is the sole caller of `goGit()` and that a retry repeats the whole read.
- `PATTERN-comment-line-breaks` — "Writing a Go comment: semantic line breaks, one sentence per line, checked by `lyx loom lint-comments` alone and never a review finding. (test)" with its background link; the background already states that the rule covers changed comments, has no column limit and is checked on new breaks.
- `PATTERN-no-denied-recovery` — "Naming a way forward or a step in a refusal, stencil or spec: never a command the agents' settings deny; lyx performs that step itself. (test)" with its background link; the background already lists the denied forms.

Every new `pattern/` file is named by its entry's link, as `internal/pattern`'s check requires.
The overview goes from 16375 bytes to about 15841, measured with `wc -c PATTERN.md`.

**ImpactSummary:** Doc-only: eight overview lines reworded and three background files added; `internal/pattern`'s overview and orphan checks are the only readers, and the overview the agent prompts inline shrinks.
