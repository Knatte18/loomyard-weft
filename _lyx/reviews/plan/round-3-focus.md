---
round: 3
focus:
  - "Caller completeness for signature changes: card 15 changes `ideengine.Spawn` to return a second value; settle whether every caller in the repository, including `internal/ideengine/menu.go`, is in an Edit list so `go build ./...` passes after that card."
  - "Test budget: `cmd/lyx/testdata/test-budget.yaml` caps each package's top-level `func Test` count and `go test ./...` runs the check; settle whether every card that adds or removes a top-level test function, card 14 on `internal/orchengine` included, lists the file and sets the row, or states that no function is added."
  - "Status bar height: card 5 pins `status 2`; settle whether every test, including the tmux-tier files `internal/reedengine/watchdog_integration_test.go` and `internal/reedengine/attachgeometry_integration_test.go`, that asserts a window height of client rows minus one is in the Edit list of the card that first breaks it."
  - "Retired text: for each string, key or pin a card retires (the `status-left` identity text, the `status_line` config key), settle whether a repository grep finds a remaining file asserting or citing it that no card lists, such as `tools/sandbox/SANDBOX-REED-SUITE.md` or `docs/shared-libs/yamlengine.md`."
---

Entries come from the round 2 findings; each states the question to settle against the current plan.
