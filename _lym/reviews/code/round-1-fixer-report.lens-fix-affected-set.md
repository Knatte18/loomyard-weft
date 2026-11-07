```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

I reran each finding's set search. The fix covers every member I found.

- **F1 (stamped-line budget):** `widestStampedLine` is the only stamped-budget builder, and `validateStampedLine` is its only caller. `postOtherHub` (`crosshub.go:66`) and `receive.go:219` both go through it. The unstamped `widestLine` and `LineFits` callers (`verbs.go`, `loomcli/decision.go`) deal in local lines with no `SendingHub`, so they need no repository. Every other `MessageLine` site renders a real message, and its stamp already comes from `HubStamp`.
- **F2 (journal fields):** every site that builds or copies a `Message` from a record carries `SendingRepository`: `fold.go:518`, `:600` and `:706`. The remaining `SendingHub` sites without it are `LogEntry` (`fold.go:637`; the review rejected adding the repository to `log`), reads of `msg.SendingHub` (`sweep.go`, `verbs.go:259` and `:940`, `receive.go:235`), and the dropped record (`receive.go:334`). None of them copies a message. The fix's tests cover both the replay path and the carry path.
- **F3 (envelope field):** `messageFields` (`cli.go:326`) is the only place that emits `sending_repository`. The test asserts it both present and absent.
- **F4 (reinterpretation wiring):** `RewriteShortnamesToRepositories` has one production caller, the `creel` registration. Its only resolver is `repositoryOfShortname`, and `LookupRepository` has one production caller, `configreg`. The new tests cover the registration, the classification in `markUnresolvable`, and `LookupRepository`.
- **F5 (drop versus return):** `ErrShortnameUnresolvable` has one producer (`markUnresolvable`) and one consumer (the rewrite). `markUnresolvable` marks `ErrUnresolved` and `ErrNoRepository`. Both come from `Lookup` or `RepositoryName`, the only sources of a drop-class error. `UserStateDir` and unreadable-record errors pass through unchanged.
- **F6 (name validation):** `hubregistry.RepositoryName` is the only place that turns an origin URL into the stamp's name. Both of its callers, `creelcli/hubdirectory.go:85` (feeding `receive.go:282`) and `hubsverb.go:103`, go through the validated function. The other `ParseOwnerRepo` callers (`fabriccli`, `landingshed`, `hubissues`, `pullrequest`) never reach the creel stamp. They are outside the finding's premise.

Calls made:
1. Grep for `RepositoryName(`, `LookupRepository(` and `ParseOwnerRepo(` outside tests, to cover the F6 and F4 caller sets.
2. Grep for `RewriteShortnamesToRepositories` and `ErrShortnameUnresolvable` outside tests, to cover F5 and F4. It ran in the same Bash call as 1, 3 and 4.
3. Grep for `SendingHub` sites that lack `SendingRepository` in `creelengine` and `creelcli`, to cover F2 and F3.
4. Grep for `MessageLine(`, `HubStamp()`, `widestLine(`, `widestStampedLine(` and `LineFits(` outside tests, to cover F1.
5. Read `verbs.go:934-948` and `cli.go:632-646`, to check that the leftover `SendingHub` sites copy no `Message`. My context did not show those spans.
