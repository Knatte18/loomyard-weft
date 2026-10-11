# fork-wait-noise: gates run in the foreground

Batch `01-bash-timeout-ceiling` (cards 1–6) reported `status: done` at head `27d5feec496c0896e55fd69583b24839b5ec913e`, with no deviations from the declared file-ops.

- Launch and resume lines carry `BASH_MAX_TIMEOUT_MS` from one constant, and the raw-go steer became the foreground gate rule.
- The claude turn-end activity reading counts fork transcripts, and a pending fork tool call reads active up to the ceiling.
- The implementer and verify-fix bodies run every gate in the foreground and never background a call.
- The Master template reads an interim fork or fixer notice as no completion, and its idle-turn-end sentence is corrected.
- The `verify-module-wide` refusal, its spec row, the plan stencil and `CLAUDE.md` name the foreground route.
- The orch role stencil copies the gate rule into a test-running non-fork agent's brief.

`record-batch` returned informational scope-outside-plan warnings only, for touched test files and the activity reader symbols.
