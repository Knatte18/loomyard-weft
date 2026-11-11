```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Lens: fix-affected-set. I reran each finding's search for sites the fix might have missed.

- **F1 (agent-facing promote guidance):** the only agent-facing description of promote's payload is `plugins/ly/skills/board/SKILL.md`, which the fix updated. The other mentions are generic. `board-policy/SKILL.md` and `docs/overview.md:463` don't show a payload, and the CLI help and `boardengine/doc.go` already cover `depends_on`. The remaining `promote` hits are `lym stencil promote` or historical research docs. Not checked: the error way-forward strings at `internal/boardengine/sketch.go:113` and `internal/boardengine/doc.go:578`. They print the plain `lym board promote '{"slug":...}'` form; I left them as correct for a refusal that has no dependency to carry.
- **F2 (notice assertions):** the CLI test and the engine `TestWriteSiblings` row now both cover promote. The review named only those two sites, so no other member is left.
- **F3 (edge paths):** the review named four paths: equal list, empty list refused, null `depends_on`, and a non-string element. Each has one row, and I found no further member to cover.

Calls made:
1. `grep "board promote"` across `.md` and `.go` files, to find sites naming the promote verb beyond `SKILL.md`.
2. `grep depends_on` and `set-deps` in `plugins` and `docs`, to find other guidance on ordering.
3. `grep promote` in `plugins` and `docs`, to find other agent-facing skills or docs on promotion.
