```yaml
lens: existing-paths
outcome: reported
```

Paths traced, per decision (survives / breaks):

- D2 token file under `cacheDir()`: githubclient writes only its own `credentials.json` there — survives.
- D2 channel/operator ids in machine layer `<Board>/.lym/config/slack.machine.yaml`: per hub; no reconcile rewrites a machine file (`internal/configengine/doc.go:42-46`) — survives.
- D3 relay record under `runproc.HubOwner` × prime anchor change: `fabriccli` `EndDobbies` → `runproc.EndSparing(..., dobbyOwner(prime)=HubOwner, ..., RolePush, RoleSync)` ends every HubOwner process not a push or sync; `EnsureDobbies` restarts dobbies only (`internal/fabriccli/anchor.go:411`, `internal/fabriccli/anchordeps.go:248-268`) — **breaks** (finding 1).
- D3 relay record × `lym tender` orphan check: `orphanOwner` never treats HubOwner as orphan (`internal/tenderengine/processes.go:117`) — survives.
- D3 relay record × `pawlwiring`/`orchcli` `EndRole(..., RoleDobby)`: ends dobby role only — survives.
- D3 `lym slack stop` (`EndRole`) × D3 prime dobby restart each pass while `slack`: next pass restarts it — **breaks** (finding 3).
- D3 relay `read --wait` × creel daemon ending on replaced binary: reserved unacked mail released and read again (`internal/creelengine/doc.go:37-38, 98-99`) — survives (at-least-once).
- D5 `SLACK` mailbox × creel retention: `retain` carries every unread message and the program's read forward past 7 days (`internal/creelengine/journal.go:301-331`, `internal/creelengine/doc.go:68-71`) — D5's bound misreads it (finding 2).
- D5 reply to `[slack]` message: goes to the original sender = `SLACK` program mailbox, which stays known across retention (`internal/creelengine/doc.go:104, 71`) — survives.
- D4 `@<slug>` post to `<shortname>:<slug>:orch` × beam torn down before its orch reads: program mail to a hub orch with no listed strand stays under the orch's name and counts for no sweep (`internal/creelengine/doc.go:55, 271`) — **breaks silently** (finding 4).
- D4/D6 posts × pause covering the orch: creel still stores the mail (`internal/creelengine/doc.go:256-259`) — survives.
- D6 presence notices × beam orchs whose strand is down: mail waits in the pool (`internal/creelengine/doc.go:55`) — survives.
- D6 resume pointer × `resume-context` hook: renders from persisted state through `orchengine.ResumeContext` (`internal/orchcli/resumecontext.go:59`) — survives provided that function reads the presence file.
- D8 reminders to `SLACK` before relay's first read: unreachable because `away` waits for that read (D6) — survives.
- D8 wait start from history `At` × folded entry: `appendOrFold` keeps the first call's `At`, folding only budget-exempt Stuck entries (`internal/shedengine/run.go:725-750`) — survives, but the clock does not restart on a folded re-entry (bound territory; not reported here).

Findings:

1. MEDIUM · design · decision-record.md D3 — A prime anchor change ends the relay, unstated and unbounded. `EndDobbies` ends every HubOwner process except push/sync, so the relay's new role is ended too, and `EnsureDobbies` restarts only dobbies. With presence `slack` the next prime dobby pass brings it back; with presence `terminal` it stays down until `lym slack start`, so the operator's Slack messages go unread and replies wait in creel. Evidence: `internal/fabriccli/anchor.go:411`, `internal/fabriccli/anchordeps.go:248-268`, `internal/runproc/end.go:62-64`. Fix: add the relay's role to the spare list, or start the relay in the same restart step, or state the bound in D3.

2. MEDIUM · consistency · decision-record.md D5 Bound line — "creel's seven-day retention bounds how long mail waits for a relay that stays down" is false: retention carries every unread message and the program's read into the new day file before deleting old files, so mail to `SLACK` waits with no time bound and a relay that returns after a long stop posts its whole backlog to the channel. Evidence: `internal/creelengine/journal.go:301-331`, `internal/creelengine/doc.go:68-71`.

3. MEDIUM · design · decision-record.md D3 — `lym slack stop` ends the relay through `EndRole`, but D3's own restart rule (prime dobby each pass while presence `slack`) restarts it on the next pass, so `stop` lasts only until then. The record does not say whether `stop` refuses, sets presence `terminal`, or is only temporary. Evidence: decision-record.md D3 bullets "started … by each pass of the prime's dobby while presence is `slack`" and "`lym slack stop` ends it".

4. LOW · design · decision-record.md D4 — An `@<slug>` message to a beam torn down before its orch reads it waits forever and nobody is told: creel keeps unread program mail to a hub orch under the orch's name and never sweeps it back; the relay's slug check runs only at post time, and no Slack answer follows. Evidence: `internal/creelengine/doc.go:55, 271`.

Tool calls:
1. Grep for every `runproc.End`/`EndSparing`/`EndRole` caller and `HubOwner` use — to find paths that end the relay's record; manifest held no callers.
2. Read `anchordeps.go` `EndDobbies`/`EnsureDobbies` span, `tendercli` `EndOwner` span, `tenderengine` `orphanOwner` — to see what those callers end and restart.
3. Grep for `EndDobbies` call sites — to learn the spare-role list.
4. Read `internal/fabriccli/anchor.go:395-420` — the call with its spare list.
5. Search for the PR-Review producer and a status `since` field — to check D8's wait-start premise; found no `since` field and nothing usable.
