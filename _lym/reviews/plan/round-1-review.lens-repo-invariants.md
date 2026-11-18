```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked three things: test seams and the tier tags of new test files, doc text the card makes false, and whether the card can pass its gate in the state the earlier cards leave. Two `pattern/` passages were stale or close to it: one went into finding 5, and the other holds because card 14 rewrites it.

**Per card, each invariant checked**

- **Card 1:** The only `Shell` implementers are `posixShell` and `pwshShell` (internal/shell/posix.go:40, pwsh.go:48), so the seam holds. No doc text goes false. Order holds.
- **Card 2:** Seam: `claudeengine#Config` has no `//lym:seam` marker, so that holds. **Broken:** `internal/configreg/configreg_test.go:198-215` pins `configengine.MachineKeys` per module. A module with no case, which includes claude today, expects `[]string{}`, so the new `accounts` and `account_pins` keys fail it and no card lists that test. Doc text holds, since card 2 lists claudeengine/doc.go under Extends.
- **Card 3:** No import cycle: hubgeom → shuttleengine is safe, because `go list -deps ./internal/shuttleengine` holds no hubgeom. `keepWarmWith` and `TestKeepWarm` are listed. Doc text holds.
- **Card 4:** Held. It adds new files and `lymdirs` constants and touches no seam struct.
- **Card 5:** Held.
- **Card 6:** Held. No seam test pins `ScrubbedEnv`, and its tmuxkit mirror is listed.
- **Card 7:** **Broken doc text** in two files:
  - internal/shuttleengine/claudeengine/doc.go:142 says "A worktree's project directory is `&lt;home&gt;/.claude/projects/&lt;dir&gt;`".
  - claudeengine/doc.go:714 says "The config directory is CLAUDE_CONFIG_DIR, else `.claude` under the home directory".
  - claudeengine/doc.go:674 places the plugins file under `CLAUDE_CONFIG_DIR` or `~/.claude`.
  - internal/shuttleengines/doc.go:30-31 says `RecordReaders` builds "with a zero shuttle config and no config directories, as SessionNamers builds its engines".
  - Card 7 lists both doc.go files only under Extends, yet it rewrites these sentences.
- **Card 8:** **Broken doc text:** claudeengine/doc.go:416-433 describes one registry, "whose registry entry names a live pid" and "reads process liveness from the same registry". Card 8 widens the check to every known registry, and lists the doc only under Extends.
- **Card 9:** Held.
  - `registration_test.go` requires every `Command()` to be mounted, and card 9 mounts it in `newRoot`.
  - `clitree_test.go` checks the form of every command.
  - `testmain_test.go` holds only `TestMain`, so having no tier tag is fine.
- **Card 10:** Held.
- **Card 11:** `ReedOps` implementers are reedengine, shuttleengine/fakes_test.go and shuttlefake, all covered, so the seam holds. **Broken doc text:**
  - internal/reedengine/doc.go:471 says Resume "relaunches a non-live strand that has no list or a missing path". A held strand now breaks that, and card 11 lists the doc only under Extends.
  - pattern/PATTERN-shuttle-provider-seam.md:120 says resume "drops a non-live strand whose list is non-empty and fully present". A held strand is no longer dropped. No card edits that passage: card 14 edits only the Shuttle stop section.
- **Card 12:** **Broken doc text:**
  - internal/shuttleengine/doc.go:376-400 (Stop verb) says the stop forms remove only "a live strand" or "an own-window strand whose pane is present". A held strand is now removable.
  - The same section says a run whose strand "tracks with its pane binding cleared, may have a live agent". `StopStepRuns` now skips that branch for a relaunching record.
  - doc.go:904 says the carry-over reads a run as unfinished when it is "reading running with its waiter gone". A relaunching record within its bound is now excluded.
  - Card 12 lists shuttleengine/doc.go only under Extends.
- **Cards 13 to 16:** Held, apart from the doc findings above.
- **Card 14:** Held. It edits the background's Shuttle stop section, and the PATTERN.md line stays verbatim.
- **Card 17:** Held. I did not verify `orchengine.Session` implementers beyond `fakeSession` and `runnerSession`.
- **Card 18:** Held.

**Findings**

1. BLOCKING, consistency, `_lym/plan/02-claude-account-config.md`.
   - **Summary:** Card 2 classifies `accounts` and `account_pins` as `CategoryMachine`. `configreg`'s table pins every module's `MachineKeys`, and claude falls through to the default `[]string{}`. The registry test then fails, and no card lists it.
   - **Evidence:** internal/configreg/configreg_test.go:198-215.
   - **Fix:** list the test function enclosing configreg_test.go:198 under card 2's Edit, and add a claude case wanting `["account_pins", "accounts"]`.
2. MEDIUM, consistency, `_lym/plan/07-known-config-dirs.md` and `_lym/plan/08-resume-carries-transcript.md`.
   - **Summary:** claudeengine/doc.go states that the project directory, config directory, plugins file and session registry are the single `~/.claude` (or `CLAUDE_CONFIG_DIR`) one. Cards 7 and 8 make those statements false, but list the doc only under Extends, which only adds text.
   - **Evidence:** internal/shuttleengine/claudeengine/doc.go:142, :416-433, :674, :714.
   - **Fix:** move claudeengine/doc.go to Edit on cards 7 and 8, and say which sentences change.
3. MEDIUM, consistency, `_lym/plan/07-known-config-dirs.md`.
   - **Summary:** shuttleengines/doc.go says `RecordReaders` and `SessionNamers` build their engines with "no config directories". Card 7 gives both a `dirs` parameter, yet keeps the doc under Extends.
   - **Evidence:** internal/shuttleengines/doc.go:30-31.
   - **Fix:** move internal/shuttleengines/doc.go to card 7's Edit.
4. MEDIUM, consistency, `_lym/plan/12-run-relaunching-record.md`.
   - **Summary:** shuttleengine/doc.go's Stop verb section and its carry-over paragraph state the old rules. Under those rules a stop removes only a live strand or a present own-window one, a cleared-binding strand is held, and a running record with its waiter gone is unfinished. Card 12 changes all three and lists the doc only under Extends.
   - **Evidence:** internal/shuttleengine/doc.go:380-381, :399-400, :904.
   - **Fix:** move internal/shuttleengine/doc.go to card 12's Edit.
5. LOW, consistency, `_lym/plan/11-reed-hold-and-relaunch-ops.md`.
   - **Summary:** reedengine/doc.go and the PATTERN background both describe resume as relaunching or dropping every non-live strand. Card 11's held strand is neither relaunched nor dropped. No card edits the background passage, and card 11 lists reedengine/doc.go only under Extends.
   - **Evidence:** internal/reedengine/doc.go:466-471; pattern/PATTERN-shuttle-provider-seam.md:120.
   - **Fix:** move reedengine/doc.go to card 11's Edit, and add the done-when paragraph's held-strand exception to the background edit card 14 already makes.

**Tool calls**

1. Grepped the `//lym:seam` markers repo-wide, to find seam tests on structs the cards touch. The context had no seam list.
2. Grepped the seam markers for the touched packages and hubgeom's imports, to check seams and a possible import cycle for card 3.
3. Ran `go list -deps` on shuttleengine and grepped the claudeengine, reedengine and shuttleengine doc.go files, to check for a cycle and find doc text the cards make false.
4. Read the Stop verb section of shuttleengine/doc.go, grepped pattern/ and docs/, and grepped shuttleengines/doc.go, to confirm the stop-verb, background and doc text against cards 7, 11 and 12.
5. Grepped the implementers of `SetWaitMark`, `orchengine.Session`, `Shell` and the `newRoot` registration, plus the tests that pin `MachineKeys` and `ScrubbedEnv`, to check the interface-growth and registration invariants.
6. Read configreg_test.go's machine-key table and the cobra pattern's test oracles, to confirm card 2's break and card 9's registration.
