```yaml
lens: fix-affected-set
outcome: reported
```

Two gaps. `PATTERN-shed-driver` has three review-only members without the marker, and `PATTERN-github-auth` could not be checked.

**Findings**

- **MEDIUM**: `pattern/PATTERN-shed-driver.md`, the `Rule:` lines at 106, 110 and 114.
  - Summary: `PATTERN.md` marks `PATTERN-shed-driver` with `(test)`. Three of its member rules carry no `(test)` and no "Held by review alone" line. The fix therefore leaves this cluster overclaiming enforcement, which is F1's own premise.
  - Evidence: the search above lists only these three lines, so the fix covered every other member of the `(test)`-carrying clusters. The review's "same pattern on" list did not name `PATTERN-shed-driver`.
  - Scope check: the search keyed on a `Rule:` line not ending in `(test)` with no marker line within two lines after it.

**Unchecked**

- `PATTERN-github-auth`: `PATTERN.md` lists it with `(test)`, but `pattern/PATTERN-github-auth.md` does not exist. The entry may have no background file, so it has no members to mark. I did not confirm that.

**Tool calls**

1. One Bash loop over every `(test)`-carrying entry's background file. My context lacked those files except the ten the fixer read, and it held no `PATTERN.md` entry list.
