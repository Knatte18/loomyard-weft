```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The search covers `internal/selfreportcli` and `internal/selfreportengine`. The only remaining `"body"` and `"body-file"` strings are:
- the two constant declarations at `cli.go:27` and `cli.go:30`;
- a doc comment at `cli.go:125`;
- the JSON request field `body` read in the tests (`cli_test.go:235` and `selfreport_test.go:164`), which is not a flag name.

Every flag-name use in `cli.go` now goes through `bodyFlag` or `bodyFileFlag`. That covers both flag definitions, the `MarkFlagsMutuallyExclusive` call, and the two `Changed` and two `GetString` calls in `resolveBody`.

Tool calls:
1. grep for `"body"` and `"body-file"` literals in `internal/selfreportcli` and `internal/selfreportengine`. My context could not show whether other files held flag-name literals.

I left F2, F3 and F4 unchecked, since their premises are single test rows and one format string, not sets.
