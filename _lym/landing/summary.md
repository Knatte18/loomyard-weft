# Close the command-help test gap as already fixed by a69580709

This change modifies no code and adds no test.
The bug is that a card could push a command's help past the byte cap that `TestCLIForm_EveryCommand` holds,
and neither its card-level nor its batch-level test run included that test.
a69580709, already on `main`, fixes it.

Closes #274

## What already covers it

Since a69580709, the test scope of a `--card` run on Go code holds the cards' packages
and every package of the module whose build or tests reach one of them.
A card that names a package the main command package imports therefore has the main command package,
and with it the help test, in its scope.
The `card` and `batch` runs both use that scope by default.

A gap remains: the widening starts from the packages a card names, not the ones it changes.
A package the card changes without naming adds nothing to the scope, so the help test can still be left out.
For a card whose intent covers every instance of one kind across the repository,
a69580709 covers this with a plan rule: the card lists every file holding one and names the search that found them.

## Design decisions

`TestCLIForm_EveryCommand` gets no guard-test mark.
The mark would add nothing for the reported bug and would add work to scoped test runs.
It would run the test in the remaining cases where a `--card` scope leaves it out, which do not justify that cost.

## To check by hand

- Not run: no reproduction of the original failure, a command's help pushed past the cap in a card's package.
  That the main command package joins such a card's scope is read from a69580709, not observed.
- If `go list` cannot list a module, the `--card` scope keeps the cards' packages alone and says so on stderr,
  so the help test can be left out of that run.

## Merge danger

**Door:** two-way
**Blast radius:** none
