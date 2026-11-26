```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer carried out as proposed (F27 to F32) by building cases where it goes wrong.

**MEDIUM: D7's ancestor walk can miss the area hub itself (F28)**
- Changed text: "it walks the canonical cwd's ancestors and refuses when any of them has a hub key, derived from its path as `reedengine.ServerName` derives it, with an area file."
- Summary: `ServerName` hashes a path that was made absolute but never resolved through symlinks. The walk hashes resolved ancestors. If the hub's key came from a path through a symlink, the two keys differ, no area file is found and no host verb refuses.
- Evidence: `internal/reedengine/server.go:304-312` (`cleanAbsHubPath` is `filepath.Abs` only) and `server.go:47-52` (the key hashes that path).
- Concrete case: `~/Code` links to `/data/Code`. The area file is keyed from `/home/knatte/Code/foo-LYMHUB`. The walk from `/data/Code/foo-LYMHUB/wt` derives a different key, finds no file and refuses nothing.
- The walk needs to derive each ancestor's key the same way the area file's key was derived, or the hub key needs to be derived from the canonical hub path everywhere.

**MEDIUM: nothing says what attach does when `attach-scope` fails (F27)**
- Changed text: "asks the supervisor's `attach-scope` verb to move it into the scope `lym-area-&lt;hub key&gt;-attach-&lt;pid&gt;.scope`" (D6).
- Summary: the record never says the attach refuses when the scope cannot be made. If it goes on, the client and every orphan of its `detach-client -E` command stay in the operator's shell's cgroup. Under the new cgroup test they then count as host processes, so the F27 fail-open comes back.
- Evidence: D3's start bullet says "a scope that cannot be made refuses the start". D6 says nothing like that for the attach client.

**MEDIUM: the F29 fix depends on an area mark that another daemon's rewrite can drop**
- Changed text: "the hub's machine-record entry, which `lym area start` writes and marks as an area hub's. … For an area hub, `hubregistry.Lookup`, the confirm's geometry and `RepositoryName` read that entry".
- Summary: D5's rationale says "the machine record is written by every daemon outside an area". The review held back that an older binary's `hubregistry.Record` rewrite may drop per-entry fields D5 and D6 add. If such a rewrite drops the area mark, the receiving daemon treats the hub as an ordinary one and reads the Bolt shortname file and the prime's `.git/config` again. That brings back the origin spoof.
- Evidence: D5's rationale text, and the review's held-back item on `hubregistry.Record`.
- A receiving daemon outside an area could instead decide area-ness from whether the area file for the hub's key exists. Only the operator writes that file.

**LOW: the `lym-area-&lt;hub key&gt;-` prefix is ambiguous between hubs (F27)**
- Changed text: "every scope the supervisor makes for the area (D6, D9) under the prefix `lym-area-&lt;hub key&gt;-`" and the inside test that uses it.
- Summary: hub B whose base name is hub A's full key plus a suffix gets scope names that open with A's prefix. B's processes then test as inside A. A's daemon would resolve them as A's own senders, and A's `lym area stop` would list and stop B's scopes.
- Evidence: `server.go:52` builds the key as `"lym-" + base + "-" + shortHash`, and the base comes from the hub's directory name. For example, A's key is `lym-foo-LYMHUB-12345678`, and B is `foo-LYMHUB-12345678-pane-LYMHUB`. Only the operator can create such a hub.

The F30, F31 and F32 proposals held up against every case I built.

Tool calls:
- `grep` of `cleanAbsHubPath` in `internal/reedengine/server.go`: my context showed `ServerName` but not how it cleans the hub path.
