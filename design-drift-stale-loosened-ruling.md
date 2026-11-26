# Drift-Verify reuses a stale loosened ruling after the moved checks land

`drift-stale-loosened-ruling` · Task · bug, drift · done

Imported from [issue #249](https://github.com/Knatte18/loomyard/issues/249).

## What was seen

A darn run deleted a test harness file (`Program.cs`) whose checks moved into a test class.
The drift judge ruled the deletion `loosened`, with three check groups `uncaused`, and Drift-Verify halted.
The parent recorded a decision to add the three missing checks and sent the run back to the writer, which committed them in a new commit.
Drift-Verify then halted again with the same three `uncaused` checks.
`_lym/drift/judged.json` still held the ruling from the first judge spawn, keyed on the diff hash of the deleted file, which had not changed.
No new judge spawn and no drift-judge parent ask came for the new commit.

## Workaround

A decision told the writer to commit an empty commit with one `Loosened-Check` trailer per check group, naming the commit that added the checks as the cause, then a goto to the writer row and a resume.

## Expected

When a later commit in the range touches the test files the stored ruling compares against, such as the class a deleted check moved to, Drift-Verify re-judges the loosened file instead of reusing the cached ruling.
At least, the halt's way forward should name the parent's ways out of a ruling that the code has since answered.
