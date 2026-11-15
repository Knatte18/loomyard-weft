```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the affected sets of F1 (stale pause-holds-everything docs) and F3 (literal `LYM_RUN_PROCESS` sites).

- **F1, LOW:** `internal/hibernateengine/doc.go:123` still reads "The pause holds sessions and the network calls of other processes." After this change, a pause holds the network calls of a run's own processes only. An operator verb passes. The sentence is the same kind of stale claim as the `adoptcli/doc.go` paragraph the fixer reworded, and the review did not name it. A reword to "the network calls of a run's own processes" covers it. My grep also matched `internal/adoptcli/cli.go:35`, `internal/reportcli/doc.go:72` and `internal/reportcli/timing.go:177`. They say a pause holds runs, which is still true, so I raise no finding on them.
- **F3, no finding:** The search found three literal sites. The first is the kit's literal at `internal/testkit/tmuxkit/tmuxkit.go:540`, which the review named. The second is the fixed `shuttleengines_test.go`. The third is `internal/reedengine/lifecycle_test.go:358`, which the review did not name. Its input and its comment spell the literal, but the code under test uses `pauserecord.RunProcessEnv`. A rename leaves the literal entry in the output, so that test fails and the drift is caught.
- **F2:** not a set premise, so I did not check it.

Calls made:
1. A grep across the Go and Markdown files for the "pause holds" phrasings, and one for `LYM_RUN_PROCESS`. I needed every site that repeats either claim, and my context held only the files the review named.
2. A `sed` read of `internal/hibernateengine/doc.go` lines 118–128 and `internal/adoptcli/cli.go` lines 30–38. I needed the surrounding text to judge whether the two hits claim a pause holds every process.
