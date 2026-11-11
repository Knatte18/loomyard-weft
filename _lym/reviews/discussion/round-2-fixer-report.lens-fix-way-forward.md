```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that every criterion, miss, question and step the fix added can be reached and leads to progress.

Ways forward checked:
- D3: an importing package whose union is empty counts as `none selected` and is left out of the selected invocation. The new D3 criterion with an unwatched guard test can be reached, and the gate passes.
- D2: testmap repeats its dirty check after a package's tests and discards that map when a file is dirty. A later run rebuilds it once the tree is clean.
- D4: the parameter-type change criterion can be reached.
- D5: these new misses can each be reached, and each leaves a whole run that makes progress:
  - an embedded file in a subdirectory
  - a const in a linked package with no covered block
  - a method receiver changed from value to pointer
  - an unparsable file
  - a `//go:` directive line
- The second Open risks question (Q11) gives the parent two options it can act on.

Finding:

- **LOW.** Text: the last D5 bullet, "or the working tree holds such a file the record lacks."
  - Summary: as worded, a git-ignored file under a linked package's directory is a miss that no testmap rebuild can ever clear.
  - Evidence: D2 records only tracked files. testmap's dirty check reads `fabric.Status` (`internal/gatecli/testmap.go:270-277`), which does not list ignored files. So a rebuilt map still lacks the file, and the miss stays until someone deletes it.
  - No such file exists today: `git status --ignored` lists none under `internal`, `cmd` or `contracts`.
  - Way forward: limit the clause to untracked files that are not ignored, which the D2 dirty check already makes testmap skip until they are committed or removed.

Calls made:
1. `git status --ignored --porcelain -- internal cmd contracts`: checks whether ignored or untracked files exist in package directories. My context did not show ignored-file state.
