```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. For each finding whose premise is a set, I re-ran its search and checked that the fix covers every member.

Unchecked: F3, F6–F8, F10–F14 have no set premise. F4's set (upsert rows that never reach `normalizeTask`) holds only the two kinds of row the review named, and the fixer covered them. F15 is decision-class and out of scope.

**F1 and F2: covered.** The only test assertions of the old wording are `internal/battencli/lifecycle_integration_test.go:1523`, `internal/orchcli/refusal_integration_test.go:127` and `internal/orchcli/cli_test.go:155`. Cards 23 and 19 now cover all three, through `stepRecordsPrimeRefusal`, `refusalCase` and `TestRefuseNonPrime`. No other test asserts a fragment of either refusal, such as "not the prime worktree" or "re-run it from there".

**F9: covered.** The sites naming the prime's session are `cli.go:172` (`battenVerbTexts`) and `cli.go:337` (inside `Command`), and card 24 now carries both. The other matches (`drive.go:288`, the reed doc's prime-session mark, `docs/overview.md:619`) are about the tackler's strand and the session mark, not the `--window` site.

**[LOW] Card 23, F5's set (prime-only help text in the verbs card 23 moves to `ResolveBattenPrime`): the fix rewrites two members of the set, not all of them.**
The rewrite sentence names only batten's `Command` Long and `cleanupCmd`, plus the adopt and cleanup package docs. Five more members still say prime-only:
- `internal/battencli/lock.go:30`: `lockCommand`'s Long says "Both verbs run from the hub's prime worktree only". `lockCommand` is not on card 23's Edit list; only `lockStore` is.
- `internal/battencli/cancelcmd.go:45`: `cancelCmd`'s doc says "the prime-only end of a run for good". `cancelCmd` is on the Edit list, but the Intent never says this text is rewritten.
- `internal/battencli/overview.go:449`: `overviewCmd`'s Long says "runs from the prime only". It is on the Edit list but not named in the rewrite.
- `internal/adoptcli/cli.go:24`: `adoptcli#Command`'s Long says "It runs from the hub's prime worktree only." It is on the Edit list, but the Intent names only adopt's package doc.
- `internal/cleanupcli/cli.go:27`: `cleanupcli#Command`'s Long says "It runs from the hub's prime worktree only and refuses elsewhere." It is on the Edit list, but the Intent names only cleanup's package doc.

**[LOW] Card 19, F2's sibling set: the integration file's own comments still describe a prime-only rule.**
`internal/orchcli/refusal_integration_test.go:3`, `:21` and `:118` (refusalCase's doc says "asserts the prime-only refusal") still describe the old rule. Card 19 rewrites `refusalCase` to assert the new opening, so its doc comment goes stale unless the card says it changes too.

Tool calls:
- grep for prime-only wording under `internal/battencli`, `internal/adoptcli` and `internal/cleanupcli`: to rerun F1/F5's search; my context had only the cited lines.
- grep for prime-only wording under `internal/orchcli`: to rerun F2's search over every orch site.
- grep for "prime's reed session" in batten, the reed doc and `docs/overview.md`: to rerun F9's search.
- sed of `lock.go`, `overview.go`, adopt's `cli.go` and cleanup's `cli.go` around their matches: to find which symbols hold the remaining help texts.
- grep of test files for other fragments of the old refusal texts: to finish F1/F2's set of tests asserting the old wording.
