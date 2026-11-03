```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the three as-proposed test fixes (F1, F2, F3) by building a case where each proposal is wrong or breaks something.

no findings

- F1 (`TestEngine_EntryPoints_ReadEngineBlocksForTheGeometrysDefaultEngine`): the test fails if any entry point passes `""`.
  - `Probe`/`Resume` are refused through `Table.Validate` with "no engine was named".
  - `Run` is refused the same way.
  - A `startSeat`/`composeSeatPrompt` that passed `""` would let `Run` start the seat, then fail it at prompt composition, so the test never sees a start and fails on the `awaitStarts` timeout.
  - Resume's only use of `DefaultEngine` is `Validate`, so that row covers the whole call site.
- F2 (`TestEngine_HalvesRenderEngineBlocksForTheirOwnEngine`): the review choice names `claude`, the fix choice names none, and `DefaultEngine` is `codex`. Each mutation breaks at least one assertion.
  - Swapped `opts.Review`/`opts.Fix` flips the engine words in both halves.
  - A dropped fallback leaves the fix half with `""` and `ReadSet` refuses.
  - Ignoring the choice engine in `seatEngine` gives the review step `Codex words.`
  - A `New` that drops `DefaultEngine` from the seat geometry makes the fixer seat stencil, which now includes the engine block, refuse in `Validate`.
  - The review half's fallback branch is covered only by the fix half, which is enough because `seatEngine` is shared.
- F3 (`TestComposePrompt_RendersEngineBlocksForTheHalfsEngine`): the explore stencil's `ReadSet` argument is now pinned, since an empty or wrong engine there fails or renders the other engine's words.
  - The appended include declares no marker, so the explore stencil's marker sets are unaffected.
- Moving the stencil and block seeding into the `seedEngineWords` helper changes nothing for the old rows.
  - It writes the same bodies and the same `claude`/`codex` copies, and the shipped bodies come from the `stencils` package, not from the seeded files.
  - The `stencilstore` import removal in `template_test.go` left no other use behind (the gate build passed).

Tool calls made: none; my context already held every changed file and the gate result.
