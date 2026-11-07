```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens on the `beamHubRegistration.Build`, `badReedUpRegistration.Build`, `beamHub` and `newBadReedUpFixture` changes. I found no defects.

**Per function or branch, the case I ran:**

- `beamHubRegistration.Build`, happy path: it runs `AddPair(the-beam)`, then `AddPairWith` for `member-with-pair` and `forked-child` with `Parent: the-beam`, then writes `board.json` and the shortname record. The steps and their order match the old `beamHub`. Correct.
- `beamHubRegistration.Build`, uncommitted files surviving the copy: `buildTemplateIn` fingerprints the root after `Build`, so both files are in the template. `relocateHub` copies them as text; they hold no template root path. `commitRelocatedBinding` stages only `.lym-warp` and amends with `--no-edit`, so the unstaged `board.json` and shortname file stay uncommitted in the copy. `requireBoardAtUpstreamTip` compares commits only, so those files don't trip it. Correct.
- `beamHubRegistration.Build`, failure of an `os.WriteFile`: `tb.Fatalf` goes through `templateBuildTB`, which records the failure and calls `Goexit`. `templateFor` then marks the slot failed and fails the caller naming the key `loomcli-beam-hub`. Correct.
- `beamHub`, concurrent callers: five tests and subtests copy in parallel. The slot mutex serialises the one build, and each caller gets its own `TempDir` copy. The subtest that deletes the shortname record only changes its own copy. Correct.
- `beamHub`, `location`: `lymcwd.ResolveWorktree(hub.PairCodeWorktree(name))` uses the copy's rebased paths. Correct.
- `badReedUpRegistration.Build`, mouse-line rewrite: the `mouse:` line is replaced with `not-a-mouse-value`. If the template has no such line, `replaced` stays false and `tb.Fatalf` fails the build with the same message as before. Correct.
- `badReedUpRegistration.Build`, `SeedConfig` then `AddPair`: the order is the same as before, so the pair forks from the seeded config. The seeded config text has no template-root path, only `providerlessBinary` (a constant) and shipped templates, so relocation leaves it valid. The seeded commit stays on the weft prime and is copied along with the tree. Correct.
- `badReedUpRegistration.Build`, the `.` shape's no-pair base: it adds no pair before `SeedConfig`. Correct.
- `newBadReedUpFixture`, boundary where `seed` is a no-op or writes into the pair: `lymcwd.Resolve` runs on the copy's worktree before `seed(t, loc)`, as before. `commitRetiredBatcherKey` and the `WriteSeed` calls act on the copy. Each call site gets a fresh copy, so `TestLoomFailedReedUpRefuses`'s three drivers cannot collide on a seed. Correct.
- `newBadReedUpFixture`, `badReedUpSlug`: it is `"loom-smoke-task"`, the same literal the old function used. `newWiredPairFixture` shares the slug but uses a different template key, so there is no clash. Correct.
- Key uniqueness: `loomcli-beam-hub` and `loomcli-bad-reed-up` differ from each other and from the shape names (`.+`, `.+pair`, `backend+`, `backend+pair`). `slotFor` would fail on a clash. Correct.

I made no tool calls. My inherited context already held everything I needed.
