```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the fix introduced can be reached from the state it names and leads to progress.

**[MEDIUM] Retrying after a lost board-push race always fails**

- **Changed text:** D3 says a seed push that fails "for any other cause" has the way forward "the same clone again". "Any other cause" includes a failed lease.
- **Problem:** When the board push loses its lease on absence, another hub's board now sits at `refs/lym/heads/main`. Rerunning the same `--single-repo` clone then hits D3's first refusal, because origin now holds the namespaced board branch. The retry that refusal names, the same clone without `--single-repo`, joins the other hub instead of seeding this one.
- **Second problem:** Two concurrent seeds whose primes are on different code branches lease different records refs, so neither records lease fails. The loser can therefore leave its pushed `<prime>-weft` records branch beside the winner's board. The way forward never mentions that leftover branch.
- **Evidence:** The review's own interleaving under F5 shows the loser's board push refused while the winner's board lands. D3's refusal bullet reads "with `--single-repo`, an origin holding the namespaced board branch; way forward: the same clone without `--single-repo`".
- **Suggested change:** Give a failed board lease its own way forward. It should say that origin now holds another hub, and offer a one-URL clone of that hub, or another origin. It should also say what happens to the loser's leftover records branch.

The other ways forward the fix introduced all hold:
- the transport-failure rerun, and the leased replacement of a leftover;
- the `lym-bolt`-only refusal's cleanup steps;
- the new D3 and D4 acceptance criteria, which can each be built over a bare origin.

Out of scope for this lens: dropping the `.lym-anchor` arm means an f06acf39 origin whose `lym-bolt` is already deleted, but whose code branches still track `.lym-anchor`, now passes the refusals. The fix-proposal lens should judge that.

Tool calls: none.
