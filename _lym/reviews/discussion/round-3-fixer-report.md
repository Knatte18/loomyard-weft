# Fixer report — discussion round 3

## Changes

- F1 (BLOCKING), fixed differently: D2's outbound worker starts a receiving hub's daemon at most three times, a per-hub count past which it only reconnects, each start logged and reaped; the retry bullet says the cap holds across retries and new posts; the `PATTERN-child-processes` Constraints bullet adds "caps spawn attempts by count"; a D2 criterion tests three starts across the hour with the message still returned at the hour.
  The proposal had a new post reset the count; I reset it only when that hub's daemon answers a call or the sending daemon restarts, since a strand posting once a minute would otherwise allow about 180 starts in the hour, which `pattern/PATTERN-child-processes.md:16` forbids (fix-proposal lens).
- F2 (MEDIUM): D3's drop-notice bound is keyed on the sending hub, which step 2 resolved through D1, so the window map holds at most one entry per hub of the machine record.
  The first five drops from a hub in an hour each send a notice; the fifth says further drops are journaled only and names `lym creel log --since 1h`; the first drop after the hour opens a new hour and its notice counts the drops since.
  The bound says why this meets the operator's added decision.
  The D3 flood criterion covers one or many claimed strands.
- F3 (MEDIUM): the journaled post time to the nanosecond joins the `deliver` request (D2), the duplicate tuple of steps 3 and 6 and the confirm tuple of steps 4 and 5; the id clash text names a re-created hub's later post time; a D3 bound states the residual case of a clock standing at the same instant again; a D3 criterion row tests an identical message from a hub whose ids restarted.
- F4 (LOW): D7's pre-deploy bound states as fact that the old trust sentence follows any orch strand of any hub, `orch-N` included, whatever the stamp says, names the long-lived orch of a hub that lists no trusted hub, and bounds it by a restart of every pre-deploy session right after the deploy (D9's setup and a plan-writer deploy note), stating the window before the restart as residual exposure.
- F5 (LOW): D2's return notice names the reason as a creel code, or the receiver's answer with line breaks made spaces, cut on a rune boundary to 200 bytes, escaped inside one quote as data; a D2 bound line states what the relay lets through; a D2 criterion row tests the quoting.
  I placed the bound in D2, beside the routing edge, rather than in D3's list, as the fix allowed.
- Support log: a Round 3 section records each finding and the self-check.

## Self-check

- fix-consistency MEDIUM: D6 still said the untrusted stamp makes a pre-deploy session read it right — fixed as proposed: D6 now defers to D7's bound.
- fix-consistency LOW: D9's review criterion missed the restart step — fixed as proposed.
- fix-consistency LOW: "five a hour" typo in the support log — fixed as proposed.
- fix-claims: no findings.
- fix-proposal MEDIUM: a new post reset the spawn count, leaving it uncapped — fixed as proposed: the count resets on an answer from that hub's daemon or a sending daemon restart.
- fix-proposal MEDIUM, merged with fix-way-forward MEDIUM: the rollout bound claimed an ordering only the sender controls — fixed with my own change: D7 states the window between deploy and restart as residual exposure and orders the restart right after the deploy, and D9 and the deploy note match. I did not add a rollout flag that holds cross-hub delivery, since that is new mechanism the review never asked for and the residual statement settles the claim.
- fix-proposal LOW: D7 named "an orch", narrower than the old sentence's "an orch strand" — fixed as proposed: `orch-N` named.
- fix-proposal LOW: the quoted reason was not escaped and the cut could split a rune — fixed as proposed.
- fix-proposal LOW: the counted notice's place in the next window was unstated — fixed as proposed: it is the new hour's first notice.
- fix-way-forward LOW: the fifth notice named bare `lym creel log` — fixed as proposed: it names `lym creel log --since 1h`, which `internal/creelcli/cli.go` accepts.

## Disputed

None.

## Deferred

None.
