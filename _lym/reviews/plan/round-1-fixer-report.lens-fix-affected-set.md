```yaml
lens: fix-affected-set
outcome: reported
```

Left unchecked: none.

F1: covered. The only rows delivering a differing reply-to are card 7 lines 94 and 98. Line 98 is settled by the clash check at step 3, ahead of the guard, so it still holds. Line 94 is the row the fix changed, and cards 6 and 8 have no such row.

F2: covered. The catch-all `bad-answer` also takes the plain `errors.New(resp.Error)` in `awaitAckConfirmation` (`client.go:300`).

F4: covered. No "any verb" claim remains apart from the none refusal, which the review's Rejected section keeps. `TestDaemon_Lifecycle`'s row and the overview decision were both updated.

F5: covered. The verb lists are in `docs/overview.md` at both sites, `internal/creelcli/doc.go` and `Command`'s `Long` in `cli.go`, and card 5 names all of them. The `overview.md:437` hit is the shuttle bullet, not a creel verb list.

F6: covered. It is a single member, now listed.

Finding:

- LOW. Concerns card 9's new sentence, "The Replies section of `internal/reportengine/doc.go` and the `RoleReplies` field comments name the `inbound` and `outbound` counts." The set of texts that go false is one member short. `ComputeReplies`' own doc comment at `internal/reportengine/replies.go:27` says "A reply counts only when its ReplyTo names a message the entries show as marked". An `outbound` entry counts through `Answers`, not `ReplyTo`, and its `ReplyTo` is an id on the other hub, so the sentence goes false under card 9. `ComputeReplies` is on the Edit list, but the Intent names only the other two texts. The `Long` at `internal/reportcli/replies.go:36` is generic ("the posts that expected a reply") and stays true.

Tool calls:
- grep of the plan cards for reply-to rows: the F1 search over cards 6 to 8 and 10 as one list.
- grep of `internal/reportengine`, `internal/reportcli` and `docs` for reply-count texts: my context held only `replies.go`'s head and `doc.go:206`.
- grep of the plan for way-forward claims and of the tree for creel verb lists: the F4 and F5 reruns.
- wider grep for `report replies` texts and verb lists: the `reportcli` wording and other list formats were not in my context.
