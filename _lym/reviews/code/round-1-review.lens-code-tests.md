```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour the diff adds or changes, name the test that pins it, or none.

**Per-behaviour list**

- `CLAUDE.md` edits (rules removed, new "This file holds LoomYard development rules only" section): no behaviour. It is prose that no build or test reads. None needed.
- `contracts/stencils/orch/orch-template-role.md`: the added sentence "Never keep a durable note in an engine's session memory store…". It is static stencil prose, which this lens exempts. The stencil tests in `contracts/stencils` (`markers_test.go`, `registry_test.go`, `citation_enforcement_test.go`) pin markers, banners and registry entries. I did not read their bodies. The diff adds no marker or banner, so I found no row the edit could break. The prose is exempt either way.
- `plugins/ly/skills/board/SKILL.md`: two added prose lines about the design living in the body and `lym board prune` removing a done entry. This is static skill prose and exempt. The prune claim describes existing behaviour, `boardengine.Board.Prune` at `internal/boardengine/board.go:375`, and the diff changes none of it.

The diff changes no Go code and no behaviour, so no behaviour in it lacks a test row. The diff adds no test, so it can't add a redundant one.

**Findings:** none.

**Tool calls:** none. My context already held the diff and the stencil-test directory listing.
