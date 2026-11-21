No findings.

- `embedPatterns`: ran the backtick line, the quoted name holding a space, the tab-separated line and a bare `//go:embed`; the first three yield the right globs and the bare one yields nothing. A malformed quote ends that directive's patterns.
- `embedPatternMatches`: `plain` against `plain/.hidden.json`, `plain/_x.json` and `plain/.d/x.json` is rejected; `plain/ok.json` is accepted; `all:embedded` against `embedded/_x.json` is accepted; a direct match is accepted whatever its name.
- `baseOverlay` and `inModule`: `inModule` is true for module root `.`, as the old inline check was; `isModulePackage` behaves as before for `changedTestsAt`.
- Failure paths: `failingTree` makes `goFiles` and `file` fail; the error wraps `boom` with the expected prefix and `baseOverlay` returns `nil, err`.
- The added rows fail with the old code: the dot, underscore and nested `.d` rows, the quoted-space row and the tab row.

Tool calls: none.
