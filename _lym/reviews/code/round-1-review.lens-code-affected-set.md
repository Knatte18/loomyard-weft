```yaml
lens: code-affected-set
outcome: reported
```

**Task: lens code-affected-set. For each change the diff makes, the set it forces outside its own lines.**

**List: per change, the set found**
1. **PATTERN.md line for `PATTERN-agent-addressing`:** matches the declared text verbatim. The background file, `pattern/PATTERN-agent-addressing.md`, adds a `## Slack boundary` rule. Covered.
2. **PATTERN.md line for `PATTERN-child-processes`:** matches the declared text verbatim. The background file carries the intro, the HubOwner list and the Reap-children exception. Covered. No top-level `PATTERN-reap-children` or `PATTERN-slack-boundary` was added.
3. **`PATTERN-config-layers`:** `slackengine` is listed as a degrading caller, and `cmd/lym/configstrictness_test.go` pins it. Covered.
4. **`PATTERN-cwd-resolution`:** `slackengine` is added to the bound packages. The package imports no `lymcwd`, and `hubgeom` builds its paths. Covered.
5. **`PATTERN-daemon-wakeups`:** the background names the relay loops, and `cmd/lym/daemonwakeups_test.go` adds `internal/slackengine/relay.go`. Covered.
6. **New `lym slack` module in `cmd/lym/main.go`:**
   - The "Available modules" text, the mount, `seamsignature_test.go`, the `sandbox_coverage_test.go` exclusion with its reason, `configreg` (the `slack` module row) and `docs/overview.md` are all updated. Covered.
   - Audience: see F2.
7. **New `runproc.RoleSlackRelay`:** the `runproc` doc comment and `record.go` are updated, and `PATTERN-child-processes` names the relay under `HubOwner`. Covered.
8. **`creelengine` seams (`ErrProgramClaimInStrandPane`, `CodeProgramClaimInStrandPane`, `PaneOf`, `IsOrchStrand`):** `client.go`, `server.go`, `identity.go`, `protocol.go` and `creelengine/doc.go` carry them. `creelcli/doc.go` and `goclient.go` carry `ReadWait` and `HubStrands`. The old doc line "no other refusal carries a code" is rewritten. Covered. My context holds no other reader of the old "only refusal that carries a code" claim, and I made no search for one.
9. **Anchor change restarts the relay (`fabriccli`):** `anchor.go`, `anchordeps.go`, `doc.go` and the verb's help text all say it. Covered.
10. **Prime dobby tick (`beamcli`):** the `dobbydeps.go` change, the `doc.go` text and the `overview.md` line for `beamcli` all say it. Covered.
11. **Orch bring-up (`orchcli`):** `start.go`, `cli.go` and `doc.go` say it. `paths.go` feeds the presence path, and `orchengine` adds the `SlackPresencePath` field and `WithPresence`. The presence line is covered by `resumecontext` changes and tests that I did not read here. Covered.
12. **`githubclient.CredentialDir`:** the package doc names it, and `hubgeom/doc.go` and `lymdirs` doc are updated. Covered.
13. **`contracts/orchrole/orch-role.md` `## Slack` section:** I read it against D7's four rules and the Review criterion. All four are present, and no existing rule is dropped. The added `[slack relay]` rule is not in D7. It follows from the relay's own notices (`relayNoticePrefix` in `inbound.go`), so it is consistent. Covered.
14. **Commit messages:** every commit message is a bare `N: card-name`. None carries a `Pattern-Departure:` trailer, so there is nothing to check.

**Findings**

- **F1, LOW, scope**
  - Location: `internal/configcli/configcli.go:467-470` (`buildConfigLong`).
  - Summary: the diff rewrites the editor-resolution paragraph of `lym config`'s help text. `configcli` is not in the record's scope list, and the rewrite has no connection to the slack module.
  - Evidence: the diff hunk at `configcli.go` @@ -467. The old and new wording differ ("The editor is $VISUAL, then $EDITOR, ..."), and no D-decision or card licenses it.
  - Fix: revert the hunk. If a gate forced it, record that reason with the change instead.

- **F2, LOW, consistency**
  - Location: `internal/slackcli/cli.go`, `operatorAnnotations()` and its use at `loginCmd`, `setupCmd`, `startCmd`, `stopCmd`, `awayCmd`, `backCmd`, `statusCmd`.
  - Summary: every verb carries `AudienceOperator`, and the orch's role file renders the operator command index (`orchcli/start.go:54`, `orchcli/doc.go:28`). So `login` and `setup` also appear in the orch's index, though both refuse in any pane (D2). D9 says only `away`, `back` and `status` are also in the orch's index.
  - Evidence: `internal/clihelp/annotations.go:22-32` defines `AudienceOperator` as "the operator, or the orch acting for them". I found no audience that lists only some operator verbs for the orch.
  - Fix: check whether `clihelp` has a way to keep `login` and `setup` out of the orch index, such as an index note, and use it. Otherwise record in `slackcli/doc.go` that the operator audience covers the orch's index, so D9's wording is read that way.

**Tool calls (2 of 6)**
1. Grepped for `AudienceOrch`, the "creel, heddle" module lists and the `cmd/lym` test diffs. My context lacked the `cmd/lym` test hunks and any list of the audience constants.
2. Grepped the audience constants in `clihelp` and the "command index" references. My context lacked what an operator-audience command means for the orch's index.
