# Fixing code review round 1 of the autonomy level change

F2's proposed fix asked for prose in the `**Door:**` and `**Blast radius:**` lines, which the description validator refuses.
Neither the review nor the fix instructions name that format or point at `lym loom validate-description`, so the section was written invalid and only the self-check forks caught it.
A review rubric that checks a `fix` on the change description against its spec would avoid this.
