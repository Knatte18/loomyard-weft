# Burler plan fix, round 2: a decision finding the fixer cannot act on

F1 is a `decision`-class finding whose fix starts with "ask the parent to record an Added after Discussion decision".
Round 1 raised the same finding, and its fixer deferred it.
The fixer's prompt forbids any message about the review to another session, and the decision record is outside the fixer's write surface.
So the fixer cannot ask for the decision and cannot record it, and each round can only defer the finding again.
Until something outside the fix step routes a `decision` finding to the parent, the loop will keep re-raising it.
