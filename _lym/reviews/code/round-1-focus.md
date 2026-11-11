---
round: 1
focus:
  - "internal/machinename/machinename.go: does Name fall back to the hostname only for no resolvable directory or no file, and return an error naming the file's path and `lym machine name` for every unreadable, undecodable, empty-name or invalid-name file?"
  - "internal/machinename/machinename.go: does Names drop only empty entries of `names`, keep a non-empty entry that fails the name rule, and resolve the current name as Name does, relative HOME or LOCALAPPDATA errors included?"
  - "internal/standalonestate/standalonestate.go: do UserConfigDir and AbsoluteUserConfigDir match the record's D2 on Linux and Windows variables, and does githubclient.CredentialDir return exactly the directory it did before, a relative XDG_CONFIG_HOME included?"
  - "internal/battencli/dobbyreads.go and internal/beamcli/dobbydeps.go: does DobbyLandings match a Land history entry against every current and former name, and does the Landings seam read them per call through machinename.Names?"
  - "internal/battencli/owner.go, internal/battencli/arm.go, internal/beamcli/dobbydeps.go, internal/boardengine/ownerset.go: do the name-error wrappers keep the machinename way forward and add no hostname fix, and does any user-facing message still point at the hostname?"
  - "pattern/PATTERN-leaf-packages.md, pattern/PATTERN-owner-machine.md, internal/githubclient/leaf_enforcement_test.go: does the diff land the declared leaf-packages change and the owner-machine background changes, and does the import allowlist hold only the standard library and internal/standalonestate?"
  - "internal/machinename/doc.go and _lym/landing/summary.md: does the package doc describe the machine file and its precedence over the hostname rule, and does every claim in the summary hold against the diff?"
  - "Test economy across the new and changed tests: do the machinename tests read the file only through their seam path, and does any new test repeat the coverage of another named test?"
---

Scope check: the record's Exports list names `machinename.FilePath`, `machinename.ReadFile` and `machinename.CheckName`, so confirm the diff exports each.
The `internal/machinecli` verb is a separate part and is not in this diff's scope.
