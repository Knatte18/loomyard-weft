```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: check that the F1 fix covers every place where fabric's own text names the edit directive.

List of findings left unchecked: none. F1 is the only finding, and I checked it.

F1 searched with `grep -rniE "edit directive|edit-directive" internal/fabricengine internal/fabric*`. It returned no matches, so no other site in the fabric packages names the edit directive, and the fix leaves none behind.

no findings

Tool calls:
1. `grep` for "edit directive" and "edit-directive" in `internal/fabricengine` and `internal/fabric*`. My context didn't show the other files in those packages.
