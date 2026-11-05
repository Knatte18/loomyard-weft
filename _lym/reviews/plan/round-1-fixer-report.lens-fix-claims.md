```yaml
lens: fix-claims
outcome: reported
```

I checked every claim the fix added or rewrote against the code at HEAD.

**LOW: `pullrequest.Open` cannot be passed directly as `open`**
- Changed text: card 14, in the `landDeps` bullet: "`PullRequestMode` calling `parentRuleRequired` ... and `pullrequest.Open` as `open`".
- Summary: the claim describes a value that won't compile as written. `open` is declared as `func(originURL string) (pullrequest.API, error)`, but `pullrequest.Open` returns `(*Client, error)`. Go does not let a func returning `*Client` stand in for a func type that returns the `API` interface, so `hubLandDeps` needs a one-line wrapper.
- Evidence: `internal/pullrequest/client.go:113` reads `func Open(originURL string) (*Client, error)`, against the signature card 14 creates for `parentRuleRequired`.

All other claims hold:
- **Card 1 and card 5 split:** `loomrecipe.RetiredRows` is applied at `internal/loomcli/arm.go:288` through `loomCLI.renameRetiredRows`. That method is defined in `darnwiring.go:178` and calls `shedengine.RenameProducers`. The card cites `arm.go` as where the rename is applied, which is accurate.
- **Card 14, `parentRuleRequired`:** the claims match `resolveWithParentRule` for a run with no record (`pullrequestmode.go:113-139`):
  - It reads no rules when `checkOrigin` finds a missing fact.
  - A token failure, or a non-transient failure with no record, gives one Warn and no rule.
  - A transient failure is returned.
  - `ErrTokenUnresolvable` and `ErrTokenRejected` exist (`internal/githubclient/token.go:81,87`). `githubclient.New` resolves the token lazily, so a token failure shows up on the call rather than at `open`. The text says "from `open` or the call", so it covers that.
- **Card 3:** `Repositories.GetCombinedStatus` exists in go-github v75 (`repos_statuses.go`).
- **Card 10:** with the new removal order, every crash point leaves either the land request standing (which the next wake accepts, since the entry is not interrupted) or the await marker gone. So the claim that no crash leaves the marker standing with the request gone holds, for the accept path only.
- **Card 12:** a producer error is persisted as failed (`internal/shedengine/run.go:410`), and `BeamAwait.Call` returns only Done, Yield or an error (`beamawait.go:71-94`).
- **Card 15:** the "a no passes … Finalize lands it with no pull request" claim matches card 11's beam rows. With `taskHasCommits` false the resolver answers no before it checks any origin fact (`pullrequestmode.go:83-85`).
- **Cards 5 and 7:** the cited doc texts exist where the cards say: `dobbyreads.go:214`, `dobbyengine/doc.go:119`, and the pawl paragraph in `loomrecipe/doc.go`.

Tool call made: one grep and sed run. I read `renameRetiredRows`' file and its `RenameProducers` call, `githubclient.New`'s token resolution, the token error variables, and whether go-github has `GetCombinedStatus`. My context held none of these.
