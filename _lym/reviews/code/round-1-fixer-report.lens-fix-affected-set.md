```yaml
lens: fix-affected-set
outcome: reported
```

F1 and F4 hold; I found no problems.

- **F4:** the `doc.go` count at HEAD is 161, and 130 files changed since 3d3b4812. Both of the summary's "162" mentions were changed. The "130 of the 161" sentence is therefore correct.
- **F1:** the restored list matches the base list the review named, which is all the review asked for. The per-slug struct, though, has fields the restored text doesn't name: `way_forward`, `warning`, and the per-slug dobby, action and prompt-source fields. The base doc omitted them too, so this isn't a regression. At most it is a `LOW` gap if the help text's "every field of the envelope" is read literally. The review's fix did not ask for them, and I'm not raising a finding.
- **F2 and F3:** neither has a set premise (a set of tests, callers or sites under one key), so I left both unchecked.

Calls made:
1. One Bash call, which counted the `doc.go` files tracked at HEAD, counted the files changed since 3d3b4812, and listed the `json` tags in `internal/adoptcli`. My context lacked all three.
