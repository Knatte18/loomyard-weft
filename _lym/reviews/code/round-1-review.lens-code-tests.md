```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour the diff adds, the test that pins it, or none.

**List**

The diff has no Go code and no test files. It changes five files: `docs/research/README.md`, `report.md`, and three proposal files under `docs/research/speed/testmap-upkeep-2026-11-11/`. All are static prose, which this lens exempts.

- README index line: static prose, no test required.
- `report.md`: static prose and comparison tables, no test required.
- `proposal-fable-a.md`, `proposal-fable-b.md`, `proposal-opus.md`: static prose, no test required.
- Behaviours added to code: none.

**Findings**

None.

Tool calls: none. My inherited context already showed the diff stat (docs only), so I needed no further reads.
