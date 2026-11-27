---
round: 3
focus:
  - "Commits 33d4c4e7 and the one after it, 42a73a540, change where the C# fixture compile TestCSharpFixtures_CompileWithDotnet (cmd/lym/csharpfixtures_integration_test.go) runs. Settle whether its directive lines and doc comment agree with each other and with the gate keys that now run it, and whether an upkeep run reaches it, given the test's tier and the tiers an upkeep call needs listed."
  - "pattern/PATTERN-test-speed.md says which tests name upkeep, among them a test that builds a real .NET project, and which watched guards stay at the task gates. Settle whether the two rules can both hold for every test in the tree, and whether each test either rule covers is marked as it says."
  - "_lym/landing/summary.md says where each check now runs and why watched guards stay at the task gates. Settle whether those claims, the merge-danger section included, match the tree after commits 33d4c4e7 and 42a73a540."
---

The entries cover the change made for the C# fixture compile guard, which no review has read yet.
