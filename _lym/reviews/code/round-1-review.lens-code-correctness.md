```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, running each function and branch the diff changes on concrete inputs.

**Per function or branch, the case run, and the result**

- `fixScopeRules`, `FixScopeSource` branch, case: no finding shares a changed file. The text renders as "commit each finding's fix on its own … Never push." **Correct.**
- `fixScopeRules`, `FixScopeSource` branch, case: A and B both change `x.go`. They form one group with one commit whose subject is `…: fix A, B — …`. **Correct.**
- `fixScopeRules`, `FixScopeSource` branch, case: A changes `x.go`, B changes `x.go` and `y.go`, C changes `y.go`, and A and C share nothing. A–B touch a file in common, and so do B–C. A–C do not, so the rule can be read as two groups, {A,B} and {B,C}. That puts B in two commits, which can't happen. The text never says that groups chain through a shared member. **Not correct: the grouping is ambiguous.**
- `fixScopeRules`, `FixScopeSource` branch, case: A is fixed and committed, then B turns out to change a file A changed. The text says to commit each finding "before starting the next finding". It doesn't say whether the fixer must amend or fold A's commit into a joint one, or whether B still gets its own commit. Grouping needs the changed files known before fixing, and the rule doesn't say how the fixer gets them. **Ambiguous.**
- `fixScopeRules`, `FixScopeSource` branch, case: three findings share one file. The format shows two ids (`<finding-id>, <finding-id>`), but "names every id in it" covers N ids. **Correct.**
- `fixScopeRules`, `FixScopeOverlay` branch: unchanged. Run with `FixFan` empty and with it set. **Correct.**
- `fixScopeRules`, default branch: unchanged. **Correct.**
- Error paths: the diff adds none and changes no call that can fail. Nothing to run.
- `doc.go`, `burler-review-format.md`, `burler-step-fix.md`: prose only, with no code path, so they're outside this lens.

**Findings**

1. **MEDIUM / design.** Location: `internal/burlerengine/prompt.go#fixScopeRules`.
   Summary: the joint-commit rule doesn't say that groups chain through a shared finding. Take A and C, which share nothing, each sharing a file with B. The rule can be read as two overlapping groups, {A,B} and {B,C}, which would put B in two commits. Or it can be read as one group {A,B,C}. The fixer has to improvise again, which is the friction the board entry asks to remove.
   Evidence: `internal/burlerengine/prompt.go` source branch ("Findings whose changes touch a file in common form one group …"), lines 332–336 at HEAD.
   Fix direction: say that a finding which shares a changed file with any member of a group joins that whole group, so groups chain and each finding lands in exactly one commit.

2. **LOW / design.** Location: `internal/burlerengine/prompt.go#fixScopeRules`.
   Summary: the paragraph opens with an unconditional "You commit each finding's fix on its own, once green, before starting the next finding". Only a later paragraph carves out groups. Grouping depends on which files a fix changes, which a fixer often learns only while fixing, and the text doesn't say what to do once a finding is already committed alone. It could plan the groups from the findings' `location` files first, or fold later findings into a follow-up commit.
   Evidence: `internal/burlerengine/prompt.go` lines 328–336 at HEAD.

Tool calls: none. Everything needed was already in the inherited context.
