# Card 2 — creel-post-parent

**Edit:**
- `internal/creelcli#postCmd`
- `internal/creelcli#clientSeam`
- `internal/creelcli#seam`
- `internal/creelcli#newRig`
- `internal/creelcli#TestCreelCLI`

**Extends:**
- `internal/creelcli/doc.go`

**Uses:**
- `plan:internal/orchroute#Asker`
- `plan:internal/orchroute#Live`
- `plan:internal/orchroute#Paths`

**Intent:**
Add `--parent` to `lym creel post` (D1): `lym creel post --parent <text>` takes no receiver argument and posts to the caller's parent orch, resolved at post time from the caller's worktree with `orchroute.Asker`, the rule batten's run notices use: from a pair the walk from the pair's recorded parent to the nearest live orch, from a beam or the prime the walk starting at that worktree itself, the prime's orch at the end.
The worktree is the one the cwd resolves to, the location `hubClient`'s mode resolution already reads.
The success envelope carries `receiver`, the resolved strand name, beside `id` and `sender`.
`--parent` combines with `--wait-reply`, `--interrupt` and, from card 6 on, `--expect-reply`, as a named receiver does.

Refusals, each reached by a `TestCreelCLI` row and each naming its way forward:
- `--parent` with a receiver argument is refused before any call: "post --parent takes the text alone; the forms are: post <receiver> <text>, post --parent <text> and post --reply <id> <text>".
  The two existing form refusals of `postCmd` ("post --reply takes the text alone; …" and "post takes a receiver and a text; …") list the same three forms, so every form refusal names `--parent`.
- `--parent` with `--reply` is refused before any call, naming `post --reply <id> <text>` without `--parent` as the way forward.
- A resolved worktree whose orch state records no strand (`Asker` answers not recorded) is refused with "creel: post --parent found no orch strand recorded for this worktree's parent; way forward: post to a strand by name, as lym reed list lists them", and nothing is posted.
The way forward needs no denied step.

Bound: the flag adds no privilege.
It only picks a receiver the caller could already name, and the daemon applies its usual sender and receiver rules to the post.

The liveness predicate the resolution uses is a new `clientSeam` field, `live func(worktree *lymcwd.Location) (bool, error)`, set to `orchroute.Live` in the production `seam`.
`newRig` copies the previous seam's `live` into the seam it installs, so a row that sets no predicate resolves with the production liveness; a row that needs a live beam orch sets `seam.live` itself for its duration.
A nil `live` never reaches the resolution.

`TestCreelCLI` gains rows over the rig's hub with a beam `b`, a member pair `m` of `b` and a pair `p` forked from the prime (`hubforge.AddPairWith`), orch state seeded with `orchroute.Paths` and `orchengine.SaveState`, and the resolved orch names listed as strands on the rig's fake delivery:
from `p` the post goes to the prime's orch; from `m` with `b`'s orch live it goes to `b`'s orch; from `m` with `b`'s orch not live it goes to the prime's orch; from `b` with its orch live it goes to `b`'s own orch; each row asserts the envelope's `receiver` and the message's holder through `creel log`.
The three refusals above are rows too, asserting exit 1 and the message; the first two run before the rig's daemon starts, and the row then starts it and reads `creel log` to assert that no `posted` event was journaled.
`internal/creelcli/doc.go` adds the `--parent` form and its resolution rule to the post paragraph, and `post`'s Long help gains the form and an example.

**ImpactSummary:** One new flag on creel post and one new seam field; no existing post form changes.
