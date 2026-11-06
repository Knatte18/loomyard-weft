```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set lens: reran each finding's search over the cards to check the fix covers every member.

Unchecked findings: none. F1 to F4 were all rerun.

- **F1 (Extends used where a card rewrites existing lines):** I listed every card's remaining **Extends:** refs and read the prose that touches them.
  - Cards 2 and 5 have no **Extends:** group left.
  - Card 1's `docs/overview.md` gains a new package-tree line, so it is additive.
  - Cards 6 and 9's `internal/creelcli/doc.go` gain new statements.
  - Card 8's `PATTERN-daemon-wakeups` gains a bullet.
  - Card 10's `PATTERN-agent-addressing.md` gains a section, and its `docs/overview.md` rewrite already sits under **Edit:**.
  - Cards 3 and 4 name their `doc.go` refs in no rewriting prose: the grep found them only in the ref lists.
- **F2 (false way forward):** "any lym creel verb" is gone from the cards. The remaining way-forward sites, card 1's lines 51, 56 and 61 and `00-overview.md:43`, all name `lym creel inbox`.
- **F3 (unstated failure paths and test inputs):**
  - The only signature of `listenHubFake` is in card 7, and no other card restates it. Cards 7 and 8 use the fake by behavior, so no row needs rewording.
  - Each transient answer listed at card 8's line 52 now has a code or quoting rule.
  - Other `Client.exchange` failures (`encodeLine`, handler errors) fall under the catch-all.
- **F4 (`registryDir` failure unstated):** `seam.registryDir` has two readers, `daemonHubs` (card 4, worded) and `hubs` (card 5, now worded and with a row).

no findings

Further tool calls:
- A listing of every card's **Extends:** refs and the prose naming them: my context held only the cards the review named, not 03, 06, 09 and 10.
- A grep for way-forward wording: my context lacked `00-overview.md` and cards 06, 09 and 10.
- A grep for `hubFake`, transient answers and `registryDir`: this needed the full card set, including the changed line numbers.
- A grep of the **Edit:**/**Extends:** sections and `doc.go` prose in cards 03, 04 and 10: my context did not hold cards 03 and 10.
