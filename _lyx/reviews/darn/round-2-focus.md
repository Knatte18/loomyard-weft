---
round: 2
focus:
  - "pattern/PATTERN-config-strictness.md and the PATTERN.md `PATTERN-config-strictness` entry: settle whether they name `runprofile.Load` (internal/runprofile/runprofile.go, which reads stacked layers through `configengine.StackLayers` without template fill) as a stance beside strict `Load` and `LoadOrTemplate`, and whether the entry's wording matches its background file."
  - "internal/boardcli/verbs_test.go: settle whether an unknown `profile` is tested as refused on each of the `upsert`, `upsert-batch` and `merge` paths, so that dropping `prepareRunSettings` from any one fails a test."
  - "internal/boardcli/runsettings.go `changed` computation: settle whether a profile change on a running pair's task yields the profile-specific notice, and whether re-sending the stored profile from a task-slug strand is accepted, each under a test."
  - "internal/configreg/runsettings.go second `checkLocalDocument` after `yamlengine.Merge`, and internal/configengine/set.go `setEntries` local-layer branch: settle whether a test reaches each (a merged document refused though profile and task settings are valid alone; a `--layer local` profiles `--set` write)."
---

Round 1 left its review gaps in these four places; the entries above are the questions to settle.
