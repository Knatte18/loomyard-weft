# Card 9 — docs-chair-and-advisors

**Prosa:**
- `docs/overview.md`
- `contracts/specs/shed-recipe-spec.md`

**Intent:** Bring the two cross-module docs in line with the landed design.
In `docs/overview.md`, the loom config key list names `discussion_producer`, which is `single` or `seats` and selects the Discussion-Write row's producer, and `discussion_advisors`, the advisors' model-spec list, one advisor per entry and empty for none, read only under `seats`; the Discussion producer paragraph keeps the single-agent description and adds the second producer: a chair that interviews and writes the two files while advisors study the task, the board, the code and PATTERN in parallel and answer over the session message channel, each writing a notes file beside the discussion files, all running as seats of `MultiLLMProducer` built from a Go table in `internal/loomengine`, with the chair and advisor stencils named and the chair stencil described as including the discussion stencil.
In the recipe spec, the paragraph on the `DiscussionWrite` entry gains the `DiscussionSeats` entry beside it: the same row's second producer, over a told table (`Env.DiscussionTable`) with the parent-review preparation and the row's gates on the chair, which `internal/loomrecipe` substitutes for the row's engine when `Env.DiscussionSeats` is set, so a recipe reader sees why the recipe file names one engine while the built row may carry the other.
Every inline link resolves (PATTERN-markdown-link-integrity), and `PATTERN.md` is not edited, so the overview cap decision needs no offset.
