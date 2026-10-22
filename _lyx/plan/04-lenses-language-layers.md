# Card 4 — lenses-language-layers

**Create:**
- `internal/lenses/sets/csharp.yaml`
- `internal/lenses/sets/python.yaml`

**Edit:**
- `internal/lenses/template.yaml`
- `internal/lenses#TestConfigTemplate_DecodesAndIsSelfConsistent`
- `internal/configcli#TestDispatchLayers`

**Uses:**
- `internal/lenses#lensSets`
- `internal/lenses#loadTemplateConfig`
- `internal/configcli#hubFixture`

**Intent:**
Make the base template language-neutral and ship the `csharp` and `python` layers (D1, D2).

In `template.yaml`, the five lenses `go.yaml` now carries are reworded with no language named:
`error-handling` (failures swallowed or discarded, missing context on a propagated failure, a result carrying a failure signal ignored, a crash used where a reported failure fits, state left inconsistent on a failure path),
`test-gaps` (untested behavior, missing edge and negative cases, a parameterized test missing a case its own table invites, assertions too weak to fail on a regression),
`performance` (needless allocations or copies on hot paths, quadratic work over growing data, blocking calls where a non-blocking one exists, unbounded loops, memory or concurrent tasks, a hand-rolled path where the platform's library has a direct one),
`api-design` (naming and signature consistency, documentation of the public surface, backward compatibility of a changed signature, a shape that makes the common call easy and misuse hard),
`concurrency` (shared state touched without synchronization, races between concurrent tasks, task or resource leaks on early return, deadlock from lock ordering or blocking hand-offs, cancellation accepted but not honored).
`pattern-fit`, `code-pattern-fit` and `docs-consistency` are reworded to read the repository's own rules where it keeps them: the `PATTERN.md` entries and `pattern/` background files when the repository has a `PATTERN.md`, otherwise the rules its `CLAUDE.md` files state.
`code-pattern-fit` keeps its rubric references and its `Your list:` ending.
Every lens and fan name stays, and each reworded dormant lens keeps the emphasis line and no `ignore ` phrasing.
No `formerTemplateValues` entry is added in `configsync` (D1).
The header comment's `lens_set` paragraph gains that the base names no language and that the rules lenses read the repository's own rules.

`sets/csharp.yaml` holds C# wording of the five lenses: `error-handling` on nullable reference types and `IDisposable`/`using`, deferring to the repository's own rules on argument guards and exception handling so a repository that forbids guards or try/catch around its own code gets no finding asking for one;
`concurrency` on `async`/`await`, `Task` and thread-safety;
`test-gaps` on xUnit/NUnit-style coverage;
`performance` on allocation and LINQ cost;
`api-design` on public-surface and assembly compatibility.
It adds a `numerics` lens on numerical stability and units (floating-point comparison, overflow and loss of precision, unit consistency, out-of-range physical values), deferring to the repository's own rules on guards and exception handling, and overrides `review-code` and `full` as their base lenses in base order followed by `numerics`.
`sets/python.yaml` holds Python wording of the five lenses: exception handling; threads, `asyncio` and the GIL; pytest coverage and fixtures; interpreter-level performance; public API and typing.
Every layer lens keeps the emphasis line and no `ignore ` phrasing, which card 3's layer loop checks.

Tests.
`TestConfigTemplate_DecodesAndIsSelfConsistent` drops its card 3 assertion that the go layer's five lenses equal the base;
it adds that each of the five base lenses differs from the go layer's prose, decoded from `lensSets()["go"]`, and holds neither `goroutine` nor `panic`, the language-neutral base the first acceptance criterion names,
and, under `lens_set: csharp`, that `numerics` is defined and `ResolveFan` of `review-code` and of `full` ends with `numerics`.
`numerics` ends with the emphasis line: the base fans lack it, so card 3's layer loop checks it although csharp adds it to `review-code`.
`TestDispatchLayers` gains three `lenses` rows:
a standalone worktree (`noBoard`) with no `lenses.yaml` takes `--set lens_set=csharp`, and its check loads `lenses.Load` over the worktree and wants the five lenses equal to the csharp layer's prose, read by loading `lens_set: csharp` into a scratch anchor, and each different from `lenses.Load` over an anchor holding no `lenses.yaml`, so a composition that ignored the layer fails the row;
a hub worktree (`layer` empty) with no worktree `lenses.yaml` takes `--set lens_set=csharp`, wants code 1 and `--layer default`, and its check asserts the error lacks `way forward: --set the key` and `delete the lens_set line` and that the worktree file is absent;
the default layer over `seedBoard: lensesTemplate()` takes `--set lens_set=xyz`, wants code 1, `names no shipped layer` and `wantCommits: 1`, since the refusal happens inside the commit closure, and its check asserts the error names `go` among the shipped layers and the board file still equals `lensesTemplate()`.

**ImpactSummary:** prose-only change to five base lenses and the three rules lenses, visible to every hub until it picks a layer (D6); the self-consistency test's counts and fan orders are unchanged, and the new configcli rows exercise the selector end to end.
