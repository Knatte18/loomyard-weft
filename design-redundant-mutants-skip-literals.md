# lym gate redundant never mutates a data literal, so a test that pins one is listed redundant

`redundant-mutants-skip-literals` · Note · bug

Found by the redundant-candidates-check trial on internal/boardengine. The mutation check changes operators and conditions, never a string, list member or map entry.

TestConfigTemplate_ResolvesToDefaults is a candidate covered by TestFabricReplay and kills one mutant, also killed by the cover. It alone asserts design_prefix, the two type descriptions and the empty labels map, so deleting it loses those checks.

TestRunHolds is a candidate covered by TestComputeLayers. It alone feeds Worktree-Create, Beam-Up, Finalize, a task or non-beam project at a hold-free producer name, and a beam with no or a done status. A mutant of the holdFreeProducers list would be one only a literal mutator makes.

Fix to weigh: add literal and list-member mutants, or list a candidate only when its covering tests assert every literal in its blocks.