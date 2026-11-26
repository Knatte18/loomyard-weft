# The GitHub PR body is not updated when the description file changes

`pr-body-not-synced` · Note · bug, landing

Imported from [issue #228](https://github.com/Knatte18/loomyard/issues/228).

## Seen

An analysis of the parent's PR-Review findings in the C# hubs found that the same findings were sent again in about five rounds.
The agent fixed the description file but the pull request's body on GitHub was not updated, so the parent read the old text again.
Seen in five separate rounds across several runs.

## Why it makes no sense

The description file is the one source of the PR title and body.
A PR whose body differs from the file the run holds is wrong from the first push, and a review round is spent on text that has already been fixed.

## Idea

Make the PR body follow the description file on every push, and refuse to end a round while the two differ.

- Read first where `pr update` runs for a task's pull request, and which path writes the description file without calling it (`lym beam describe` runs the `pr update` sequence for a beam; check the task path).
- After each push, compare the GitHub PR body with the file, and update it or halt with a way forward naming the difference.
- Add a test that edits the description file after the PR exists and expects the body to follow on the next push.
