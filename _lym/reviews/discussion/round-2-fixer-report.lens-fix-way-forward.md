```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every refusal, step, row and test the round-2 fix added can be carried out and leads somewhere.

**Ways forward the fix introduced**
1. D7: the ancestor-walk refusal. It reuses "run it from a shell window of the hub's server", which can be carried out.
2. D7: `lym reed attach` refuses a cwd in no worktree whose canonical parent is the hub root. It names no way forward.
3. D6: `attach-scope` refuses a caller that carries a barrier. There is no stated path for when the scope cannot be made.
4. D3 and D13: `boot` and `window` refuse a worktree deeper than a direct child of the hub root. The worktree can be carried out.
5. D3: `lym area stop` lists scopes by name over the bus. The host side has the bus, so it can be carried out.
6. D8: `lym area init` writes `shortname` and `lym area start` copies it into the entry. There is no stated path for when no shortname record exists.
7. The new acceptance criteria (D3/D8 orphan, survivor and later hub; D3/D7/D13 nested hub; D8 origin rewrite; D13 title; D6 attach scope). Each can be built from the state it names.

**Findings**

- **MEDIUM**: D6 attach bullet, "asks the supervisor's `attach-scope` verb to move it into the scope `lym-area-&lt;hub key&gt;-attach-&lt;pid&gt;.scope`".
  - Summary: the record says nothing about what attach does when the verb refuses or the scope cannot be made over the bus. If attach goes on without the scope, the orphan escape that F27 closes comes back.
  - Evidence: D3's start states "a scope that cannot be made refuses the start". The new attach step states no matching refusal, and the D8 bullet relies on "every orphan of them" being in the scope.
  - Way forward needed: attach refuses with the verb's reason and names `lym area stop` / `lym area start` as the way forward.
- **LOW**: D7, "`lym reed attach` under an area hub takes that hub, refuses a cwd in no worktree whose canonical parent is the hub root".
  - Summary: this new refusal names no way forward. D7's shared way forward, "run it from a shell window of the hub's server", does not apply to attach, whose job is to reach that server.
  - Evidence: D7's way-forward sentence sits ahead of the exemption list. Attach is exempt, so that sentence does not cover attach's own refusal.
  - Way forward needed: name `cd` into a worktree that is a direct child of the hub root.
- **LOW**: D8, "`lym area init` writes the area file's `shortname`, read from the hub's shortname record before its area is first on".
  - Summary: there is no step for a hub whose shortname record is missing. Once the area is on, the entry can never get a shortname, so the hub stays unreachable across hubs with nothing to say why.
  - Evidence: the record is written at clone time (`internal/fabricengine/clone.go:584`) and by `shortnamecommit.go:31`. The refusal in `hubregistry.go` ("run lym creel inbox in that hub once") shows a hub can lack a record. D2's start checks do not cover a missing `shortname`.
  - Way forward needed: init, or D2's start check, refuses naming how to record the shortname first.

Outside my lens, for the fix-claims lens: the D3 prefix `lym-area-&lt;hub key&gt;-` is ambiguous when a hub key plus `-` prefixes another hub's key. An example is a hub directory named `&lt;base&gt;-&lt;8 hex&gt;-…`. In that case D3's stop-by-name would stop another hub's scopes.

Tool calls:
- grep for `WriteShortname` / `ReadShortname`: my context did not show whether every hub has a shortname record from creation.
