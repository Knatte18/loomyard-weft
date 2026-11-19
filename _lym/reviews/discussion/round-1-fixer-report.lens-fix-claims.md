```yaml
lens: fix-claims
outcome: reported
```

Task: check that the claims the fix added or rewrote hold against the code at HEAD.

**LOW** — D3, "it creates the prime's records branch fresh at the anchor-marker commit, never from `origin/<prime-branch>-weft`."
The ordering this sentence states is wrong. `CloneHub` creates the records branch first, through `suffixWeftPrimaryBranch` at `internal/fabricengine/clone.go:320`. That function runs `bornWeftPrimaryBranch`, which gives the branch a real commit if it is unborn (`clone.go:530`). The anchor marker is committed onto the branch afterwards, through `commitAnchorMarker` at `clone.go:395`. So the branch is never "created at" the anchor-marker commit. It is created fresh and then the anchor marker is committed on it. Suggested wording: "it creates the prime's records branch fresh, never from `origin/<prime-branch>-weft`, and commits the anchor marker on it". The seed's "holding the anchor-marker commit" stays true.

These changed claims hold against the code:
- **Seed step placement:** the board push follows the Bolt commit of `.lym-warp`, `.lym-shortname` and the hub-wide config, and comes before the junction wiring (`internal/fabriccli/clone.go:73-96`, then line 107).
- **Leased delete on an absent branch:** it fails its lease (`internal/gitrepo/push.go:330-331`).
- **Plain delete:** it pushes to `dest.url` (`push.go:314`). The gate records a removal only on `deleted` (`internal/fabricengine/destroy.go:1631-1634`).
- **Upstream write:** it is `refs/heads/` (`internal/gitrepo/pushdestination.go:114`), which the widened D2 sentence now covers.
- **Board history:** 5a3ab0c956 holds the original body, and b8e9afe900 opens with the error object after the header.
- **Git documentation:** the cited git-push(1) and git-fetch(1) behaviour matches what git documents (`remote.<name>.push` applies only when no refspec is given; `--force-with-lease=<refname>` takes its expected value from the remote-tracking ref).
- **Rerun with another `--subpath`:** it no longer meets `adoptRecordedAnchor` on the prime's records branch (`clone.go:361`), provided `CloneHub` skips the adopt checkout, which the design now states.

Calls:
1. Grepped `internal/fabricengine/clone.go` for the order of `suffixWeftPrimaryBranch`, `commitAnchorMarker` and `adoptRecordedAnchor`. My context held only lines 370–540 of that file, not the call order.
2. Read `_bolt` commit b8e9afe900's copy of the board body, to verify the support log's new claim about it. My context held only that commit's log line.
