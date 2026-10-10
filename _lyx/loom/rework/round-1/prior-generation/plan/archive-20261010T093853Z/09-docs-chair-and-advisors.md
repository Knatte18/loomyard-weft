# Card 9 — docs-chair-and-advisors

**Prosa:**
- `docs/overview.md`
- `contracts/specs/shed-recipe-spec.md`

**Intent:** Bring the two cross-module docs in line with the landed design.
In `docs/overview.md`, the loom config key list names `discussion_advisors` as the advisors' model-spec list, one advisor per entry and empty for none, and the Discussion producer paragraph describes a chair that interviews and writes the two files while advisors study the task, the board, the code and PATTERN in parallel and answer over the session message channel, each writing a notes file beside the discussion files, all running as seats of `MultiLLMProducer` built from a Go table in `internal/loomengine`; the two prompt stencils are named, and the sentence naming `prompt.go` and `discussion.go` as the composer is replaced.
In the recipe spec, the `MultiLLM` paragraph gains that a `DiscussionWrite` row runs on the same producer over a told table (`Env.DiscussionTable`), with the parent-review preparation and the row's gates on the chair, so a recipe reader sees both routes to the producer.
Every inline link resolves (PATTERN-markdown-link-integrity), and `PATTERN.md` is not edited, so the overview cap decision needs no offset.
