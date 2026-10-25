```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged the proposals carried out for F1, F2, F3 and F5. F4 was fixed differently, so I left it out.

**Fixed as proposed and challenged:** F1, F2, F3, F5.

1. **MEDIUM**, F2. Changed text: D10, "the refusal names first updating the warp checkout by hand to a commit that tracks it, such as by merging the target warp branch into the current one, then the `--set` and then the checkout."
   Summary: this way forward rewrites the code history of the branch being left, and the refusal does not name that cost. It does name the `--set` cost to that same branch.
   Evidence: the warp is checked out on the current branch. Any by-hand update that makes it track the target's anchor commits onto that branch, either a merge of the target warp branch or a commit adding the directory. That branch is shared by every machine that checks it out, which is the same reason D10 names the `--set` cost ("shared by every machine that checks it out").
   The merge can also conflict, and nothing names a way forward for that.
   The case: branch X is at anchor `.` and branch Y added `svc/` and re-anchored there. A checkout from X to Y leaves the operator to merge all of Y's code into X just to change anchors.
   Either the refusal should name this cost as it names the `--set` cost, or the design should pick a route that does not modify X's code.

2. **LOW**, F5. Changed text: Acceptance criteria, "a live dobby of a covered worktree starts no pass and no task, `dobbyengine.EnsureLive` reports each covered dobby it did not start".
   Summary: "covered worktree" and "covered dobby" are not defined anywhere in the decision record.
   Evidence: D7 step 1 says "every dobby whose geometry the worktree's anchor reaches, the worktree's own and for the prime every dobby of this machine". A test writer reading only the criterion cannot tell that for the prime, "covered" means every dobby on this machine.
   The criterion should use step 1's wording.

3. **LOW**, F1. Changed text: D10, "Its move of the pair's code branch is unaffected, since that move changes no marker…".
   Summary: this is an unflagged asymmetry with the weft pull two paragraphs earlier.
   Evidence: the weft pull "stops that pair before updating either side". The origin reconcile now moves the code side and leaves the weft unmoved.
   The stated reason holds: the warp gaining a commit that tracks `B` helps D7's validation. But the pull refuses for the same situation without that benefit, and the decision record does not say why the two differ.
   The fix-consistency lens may want to check this as well.

No breakage found for F3: a skipped marker-changing parent still carries its own marker, and a markerless parent's tree gives D5's value.

Tool call: I read `internal/fabriccli/remove.go:200-250` to check how the run-scoped reconcile caller reads a stopped weft outcome, because my context did not show that caller. It was inconclusive, so I raised no finding from it.
