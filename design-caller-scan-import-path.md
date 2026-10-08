# Caller coverage: resolve imports by path

`caller-scan-import-path` · Note · enhancement, quarry

`plan-glyph-precision` added the `caller-uncovered` plan-gate check (D5).
Its caller scan matches by package clause name, so it cannot see a caller that imports the package under an alias or a dot import, nor a caller in a file an earlier card creates.
Both are accepted bounds in that task's decision record.
Resolving imports by path would close the first gap; worth building only if planners hit it.
