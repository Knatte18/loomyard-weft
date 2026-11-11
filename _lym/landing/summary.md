# Raddle shows each part's recipe and profile as table columns

In interactive mode, raddle puts its proposed parts to the operator as a table.
That table had a dependencies column only, so the operator had to read the prose to learn what each part will run as.

The table now has a column for each part's recipe, such as `darn`, and one for its profile, such as `solid`, besides the dependencies column.
The prose around the table keeps only the reasons for each recipe and profile and the alternatives set aside.

The change is the interactive branch of `raddleModeRules` in `internal/loomengine/raddletable.go`:
the prompt text and its doc comment.
Auto and perch modes are unchanged.

## Design decisions

- The prompt tells the agent never to repeat the recipe or profile in the prose, so the table is the one place they are stated.

## To check by hand

- No test covers the prompt text, and none was run.
  Run a raddle session in interactive mode and confirm the proposal table shows recipe and profile columns,
  and that the prose does not restate them.
