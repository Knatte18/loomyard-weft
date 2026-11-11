# Raddle shows each part's recipe and profile as table columns

In interactive mode the raddle agent puts its proposal to the operator as a table of parts.
Until now that table had only a dependency column,
so the operator learned each part's recipe and profile from the prose around it.

The interactive mode text in `raddleModeRules` (`internal/loomengine/raddletable.go`) now asks for a recipe column, such as `darn`, and a profile column, such as `solid`, beside the dependency column.
It also limits the prose around the table to the reasons for each recipe and profile and the alternatives set aside,
so the table states what each part runs as and the prose never repeats it.

## Design decisions

- The change is to the mode text alone. The proposal file already holds `recipe` and `profile` per part, so the proposal spec and its parser are unchanged.
- No test pins the new wording. `TestRaddleTable` already checks that the interactive prompt carries the interactive mode text and no other mode's text.
