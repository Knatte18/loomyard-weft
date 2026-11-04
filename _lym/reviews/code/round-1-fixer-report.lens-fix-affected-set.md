```yaml
lens: fix-affected-set
outcome: reported
```

no findings

Unchecked findings: none. I reran the search behind each finding whose premise is a set.

- **F1:** the degrading-callers list appears only in `pattern/PATTERN-config-layers.md:59`, and `ideconfig` is now in it. No other doc lists that set.
- **F7:** `ideconfig.LoadConfig` has two non-test callers in `internal/ideengine/spawn.go`, at lines 89 and 138 (`seed` and `OpenTask`), and both now load over `ConfigDirs(own)`. The third caller is the `configreg` registry row at `internal/configreg/configreg.go:158`, which is hub-wide and takes no location.
- **F6:** `workbench.editorAssociations` is read or written only in `applyMarkdownPreview`, so the new non-map guard covers every site in `internal/vscode/config.go`.

Tool calls:
1. grep for `ideconfig.LoadConfig` in non-test Go files, to list every caller for F7.
2. grep for `editorAssociationsKey` and `workbench.editorAssociations` in non-test Go files, to list every site for F6.
3. grep for `hibernateconfig` with `degrad` and `envconfig` in docs and Go files, to find other copies of the degrading-callers list for F1.
