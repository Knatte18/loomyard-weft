# Implementing cards 13 to 15: the owed-push check before the lock

Card 13 says the check before the lock runs when the pull-request mode reads `trail` or `review`.
`review` is the template default, and on an origin that no pull request can open on such a landing is local,
which the same card says never runs the check.
Read literally, every default local landing on an owed parent would stop, breaking card 12's landing on an owed parent.
For `review` the check before the lock therefore asks `ResolvePullRequest` whether a pull request would open;
a recorded pull request and `trail` are as the card states.

Card 14 asks for a non-ignored untracked file under the deleted directory to halt with today's reason.
Such a file halts the clean check before the merge-in, so the test puts it in behind a stopped first pass,
and the reason names the directory, since git collapses an untracked directory.
