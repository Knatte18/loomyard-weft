```yaml
lens: fix-proposal
outcome: reported
```

**Findings**

1. **LOW** — `internal/burlerengine/prompt.go:330`, the `FixScopeSource` string.
   - The first paragraph still orders unconditionally: "You commit each finding's fix on its own, once green, before starting the next finding."
   - The group paragraph, added by the F1 and F2 fixes, orders findings in one group to be fixed together and committed as one commit.
   - A fixer that reads in order meets the contradiction before the exception, which is the improvisation F2 described.
   - The F2 proposal added text after the unconditional sentence and did not qualify the sentence itself.
   - Evidence: the grep shows that line as the only "before starting the next" occurrence, so nothing elsewhere qualifies it.
   - A one-clause qualifier would close it, for example "Except for a group below, …".

2. **LOW** — the chaining rule in `prompt.go` and the `FixScopeSource` bullet in `doc.go`.
   - Chaining is transitive, so one broadly shared file collapses a whole review into one commit.
   - Concrete case: every finding names `doc.go`, or one finding touches a hub file that every other group touches. All findings then land in one commit.
   - That defeats the `doc.go` bullet's own stated benefit that `git log` shows which findings landed if the round dies mid-fix.
   - This is the cost of the F1 proposal. It is the correct trade, because the alternative is a finding in two commits, but the bullet's claim now holds only for non-chained groups.

3. **no finding** — F3, F4, F5 and F6.
   - F5's wording refers to the `Fix` field, which exists at `verdict.go:83`.
   - I found no counter-case for F3, F4 or F6.

**Calls made**
1. `grep -n "Fix " verdict.go` and a grep for the commit-order phrases: confirmed the `Fix` field exists, and that no other text qualifies the unconditional commit-order sentence. My context did not show those spans.
