```yaml
lens: fix-affected-set
outcome: reported
```

Findings: no findings.

F1 is a set of stale TestMap mentions in the merge's touched files. I searched `internal/runproc` and `internal/orchcli` for `testmap`, `test-map` and `test map`, case-insensitive. The search returned nothing, so the renamed subtest name was the only one left.

Findings left unchecked: none.

Tool calls:
1. `grep -rniE "testmap|test-map|test map" internal/runproc internal/orchcli`. My context did not hold the current contents of the other files in those two packages.
