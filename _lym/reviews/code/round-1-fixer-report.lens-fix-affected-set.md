```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: no findings.

- **F1:** a repo-wide grep for `gateProgram`, `<shortname>:GATE`, and the "held turn end of the fixer" and card-fixer hold-notice wording found no remaining references in `.go`, `.md` or `.yaml` files.
- **F3:** a repo-wide grep for the once-per-reach and once-per-visit wording found these sites:
  - `contracts/orchrole/orch-role.md:287` has the re-call clause on the next lines.
  - `internal/loomcli/doc.go:196` has the re-call sentence added directly below.
  - `internal/loomcli/rownotice.go:48` has the doc-comment sentence added directly below.
  - `internal/shedrun/rownotice.go:16` has the added sentence.
  - `internal/loomengine/template.yaml:61`, `internal/loomengine/config.go:563` and `docs/overview.md:1490` all state the re-call.
  - The remaining hits (`rownotice.go:2` "each time the run reaches one", and `rownotice_test.go:59`) say nothing that contradicts the re-call behavior.

1. One Bash call with two greps (F1 leftovers, F3 wording) over the repo, because my context held only the files the fixer read, not a repo-wide sweep.
