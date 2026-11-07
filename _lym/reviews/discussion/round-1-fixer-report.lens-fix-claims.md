```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

Claims checked:
1. D1 Bound: the websterengine read scans the `worktree` lines. Holds: `internal/websterengine/gitwrap.go:328-345`.
2. D1 floor bullets on `pathsort`/`fspathcmp`, the 2.31.0 porcelain lines, `get_linked_worktree` returning NULL, `strbuf_add_real_path`, and a relative gitdir needing 2.48. These hold as the fetched git sources report them.
3. D1 floor bullet on a missing HEAD. See finding 2.
4. D2 TB bullet: `Chdir` and `ArtifactDir` exist. Holds: `go doc testing.TB` lists both under `go 1.26`.
5. D2 Bound: `os.Setenv`, `os.MkdirTemp`, `git config --global` into the one hermetic global config, and a kept `Context()`. Mostly holds; see finding 1. `internal/gitkit/hermetic.go:59` sets `GIT_CONFIG_GLOBAL` to one lymtemp file per binary.
6. D2 Bound: relocation rewrites only the template root. Holds: `pathReplacer` and `findRootReference` in `internal/hubforge/template.go`.
7. D4: the non-bare initialisers write five keys and the bare ones write three. Holds: `internal/gitkit/gitkit.go:131-159` and `internal/hubforge/hub.go:457-478`. No other `init` call exists in either package.
8. D7: "unequal to its upstream tip, ahead or behind". Holds. `SeedFabricConfig` commits through `Bolt.Commit`, whose push is `_ = spawnDetachedPushFn(...)` at `internal/fabricengine/commit.go:152`, and that returns `ErrSpawnUnderTest` in a test binary (`spawn.go:71-73`). The board is left ahead.
9. D8: the battencli task does not edit `notices_integration_test.go`. Holds: `beamHub` is at `beamguard_integration_test.go:26`, and `noticeHub` and `TestNoticeSite` are in `notices_integration_test.go`.
10. D2 and D4 criteria rewrites: these match the claims above.

Findings:

- LOW — D2 Bound, "Relocation rewrites only the template root, so a path such a call leaves in the template reaches every copy unflagged."
  This sentence frames all four calls as leaving a path in the template, but two of them leave no path there.
  `git config --global` writes the hermetic file under lymtemp, outside the template (`internal/gitkit/hermetic.go:46,59`). `os.Setenv` changes the process environment.
  The damage from those two is to every later test in the binary, not to the copies. The Bound does not state that harm, though the review still bounds it.

- LOW — D1 floor paragraph, "Sources: … `refs.c` at tags `v2.43.0` and `v2.53.0`", and the missing-HEAD bullet.
  The fetch of `refs.c` was truncated. The quoted failure path came from a model summary, not from verbatim page text. The fetcher also said the files backend's flags were unverified.
  The claim is consistent with the 2.53.0 experiment in D1's Evidence and with the identical `add_head_info` at both tags. But "Sources: refs.c" overstates what was read.

Calls made:
- Read `internal/gitkit/hermetic.go:50-80` and `internal/fabricengine/commit.go:140-156`. My context showed neither where `HermeticGitEnv` points `GIT_CONFIG_GLOBAL` nor how `Commit` handles the push spawn.
