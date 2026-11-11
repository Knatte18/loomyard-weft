```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** Each function or branch I ran, the case, and the result:

- `WorkspaceFolderNames`, board with a project-less pair, a nested project and a missing project: **correct**.
  - Entry `adopt` has no project, so it is left out and its folder keeps the bare slug.
  - Entry `inner` is a project under beam `bm`, so it is named `P2: inner`.
  - Entry `deep` is under `inner`, so it is named `P6: deep`.
  - Entry `orphan` names project `gone`, which has no handle, so it is left out and keeps its slug.
  - Beam `bm` has no project, so it is left out.
- `WorkspaceFolderNames`, board file absent: **correct**. `ReadTasks` returns an empty list with no error (`parentbeam.go`), so the result is an empty map. The caller treats it as "no names", with no Warn noise on hubs with no board.
- `WorkspaceFolderNames`, board file unreadable: **correct**. It logs at Warn and returns nil, and a nil map lookup yields "", so every pair keeps its slug.
- `InstallPairFolderNamer` and `pairFolderNames`, namer never installed: **correct**. The load is nil, so it returns nil and every pair keeps its slug.
- `InstallPairFolderNamer`, restore function: **correct**. It stores the previous pointer back, which is nil when none was installed.
- `InstallPairFolderNamer(nil)`: a programmer error. It would panic on the dereferenced call, which is acceptable under the fail-loud rule, so not a finding.
- `regenerateHubWorkspace`, namer called before the lock: **correct**. A namer that takes the board's swap lock never nests inside the hub workspace leaf lock.
  - A pair created while the regeneration waits for the lock is still listed, because the names are keyed by slug and returned for every entry.
  - That a caller holds no board swap lock when it calls `RegenerateHubWorkspace` is not verified. I did not check the call sites at `anchorchange.go:539` and `pawlwiring.go:116`.
- `hubWorkspaceFolders`, hub loop: **correct**.
  - Every listed pair appears once, sorted by slug, after the prime and Bolt, whatever its parent.
  - `folder := pair.folder` is a copy, so the rename does not leak into the beam files.
  - Beam files still use `member.folder`, which carries the bare slug.
  - A namer value of "" or an absent key leaves the slug.
  - A pair with no origin record, or an unreadable one, has parent "" and is still listed in the hub file.
- `hubWorkspaceFolders`, beam flow, sub-beam `c` under `b` with `n` under `c`: **correct**. The hub file lists `b`, `c`, `m`, `n`, and each beam file is unchanged.

**Findings:** none.

**Unverified, outside my tool budget:**
- `RegenerateHubWorkspace` is also called from `internal/ideengine/spawn.go:120`, `internal/fabricengine/anchorchange.go:539` and `internal/loomcli/pawlwiring.go:116`. They all run inside the `lym` binary, where the root pre-run installs the namer. I did not check whether a caller outside the `lym` pre-run exists, for example a standalone `cliwire` binary. If one does, it would regenerate the file with bare slugs and flip the names back.
- Whether any caller holds the board swap lock across `RegenerateHubWorkspace`.

**Tool calls (1 of 6):**
1. Ran `grep -rn "RegenerateHubWorkspace("` and printed `parentbeam.go` lines 30–90. My context lacked the production callers of `RegenerateHubWorkspace` and the behaviour of `ReadTasks` on an absent board.
