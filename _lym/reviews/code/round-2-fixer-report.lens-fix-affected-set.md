```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the F12 fix.

Findings I left unchecked: none. F12 is the only finding.

no findings

I checked three sets:
- **Repo-reading tests in `cmd/lym` without an `only` marker:** my approximate awk scan showed only fixture or non-walking tests, and watched guards that already carry `//lym:watch`. The C# fixture compile no longer appears in it.
- **Tests that spawn a real `dotnet`:** the search showed `exec.LookPath("dotnet")` only in the two `internal/gatecli` files and `cmd/lym/csharpfixtures_integration_test.go`. All three are now `upkeep`. The other hits are command strings in config fixtures.
- **Sibling "a whole gate always runs it" and "watched guard" claims:** I saw none made false. The remaining doc comments with that sentence are on other watched tests, and the fix did not change their markers.

Tool calls:
1. awk over `cmd/lym/*_test.go` for repo-reading tests lacking `lym:only`: my context had only truncated directive output.
2. grep for tests spawning `dotnet`: the earlier search listed file names for a narrower pattern, and I reran the set's search independently.
3. grep for sibling claims about whole gates, watched guards, the C# fixture compile and upkeep: my context had no repository-wide search for these phrases.
