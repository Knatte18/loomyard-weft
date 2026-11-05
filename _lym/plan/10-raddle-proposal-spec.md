# Card 10 — raddle-proposal-spec

**Create:**
- `contracts/specs/raddle-proposal-spec.md`
- `plan:contracts/specs#RaddleProposalSpec` -> `var RaddleProposalSpec []byte`

**Extends:**
- `contracts/specs#entries`
- `contracts/specs#TestRegistry_NamesAndDefaults`

**Intent:** D5's format contract, read through `internal/stencilstore` like every spec (`PATTERN-stencil-ownership`).
`raddle-proposal-spec.md` pins `_lym/raddle/proposal.yaml`, parsed only by `internal/raddleparser` (card 12):
- The file is one object in YAML's JSON-compatible flow form, so a JSON object is a valid proposal (the reason, stated in the spec: the sole parser imports the standard library and `internal/discussionparser` only).
- Keys: `sketch` (the Sketch's slug); `shape`, one of `tapestry`, `tasks`, `project`, `beam`; `project`, `{slug, title}`, present exactly for `project` and `beam`; `board_checked`, a list of `{slug, revised}`, `revised` the entry's `revised` stamp in RFC 3339 as `lym board list` prints it; `parts`, a list; `cost`.
- A part: `slug`, `title`, `type` (a type label of the board's vocabulary, never `sketch`), `recipe` (`tapestry` or `darn`), `profile` (a profile of `profiles.yaml` holding a block for that recipe), `decisions` (the Sketch decision ids it builds, such as `D3`), `packages` (a list of `{path, size}`, `size` the rough changed lines), `contract` (`{takes, delivers, exports}`, each text), `depends_on` (part slugs or board slugs), `open_risks` and `criteria` (the Sketch's Open risks and Acceptance criteria bullets it carries, each the bullet's whole text as the record holds it), `pattern_changes` (the PATTERN entry ids it carries), `expected_cost_usd` (a number, or null when a cell it rests on reads "too few runs") and `cost_line` (one line saying what the figure rests on).
- `cost`: `expected_usd` and `line` for the proposal, `next_best` (`{split, expected_usd, line}`, the next-best split in one line), and `cells` (the `lym report cost` cells both figures rest on, each `{recipe, profile, stored_record}`).
- Shape `tapestry` has exactly one part, which builds every decision.
- Every filed slug, each part's and the project's, is new: free on the board and in the raddle store, and not the Sketch's.
- The spec lists the checks of D6 and D8 by name, each as the finding the parser or the re-check names, and states that the approval hash is the SHA-256 of the file's bytes.
`specs.go` embeds it as `RaddleProposalSpec` and registers it in `entries` under the name `raddle-proposal-spec` in a `raddle` family; `TestRegistry_NamesAndDefaults` gains the name.

**ImpactSummary:** One new registered spec; the registry tests read the entries list, and no engine reads the spec until the raddle stencil points at it.
