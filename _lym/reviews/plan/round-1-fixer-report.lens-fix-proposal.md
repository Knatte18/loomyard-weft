```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: I challenged each proposal the fixer carried out as proposed (F1, F3, F4, F6, F8, F9, F10, F11, F13), looking for a case where the change itself is wrong.

**F6, LOW.** Changed text: card 8's new Intent sentence ("`claudeengine/doc.go`'s resume and prober paragraphs are rewritten to match … the resume check, `SessionHolder` and the prober read every known directory's registry as one set").
- **Summary:** the proposal puts the registry-sentence rewrite in card 8, but card 7 is the card that widens `sessionHolder` and `processLiveness` to every known directory. Card 7's commit therefore still ships a doc that says one registry.
- **Evidence:** card 7's Edit list re-signs `sessionHolder` and `processLiveness` with `registryDirs []string`, and its `TestSessionHolder` and `TestProcessLiveness` rows pass two registry directories. `internal/shuttleengine/claudeengine/doc.go:424` ("The session prober (SessionProber) reads process liveness from the same registry") and `:433` ("The resume check and the prober share one reader of registry entries") are false from card 7 on. Card 7's new doc paragraph names only the project-directory, plugins-file and config-directory sentences.

The other proposals held up:
- **F1:** the second caller of `primeShuttleRunner`, at `cancelcmd.go:275`, only stops strands, so passing a told worktree changes nothing there.
- **F3:** `MachineKeys` sorts its paths (`layers.go:190`), so `account_pins` before `accounts` is the right order.
- **F4:** only `relaunch.go:51` forces `SkipSkillLoadTurn`, so a resume that starts through the start path can still take a load turn, and the case is reachable.
- **F8:** card 15 really does settle a relaunch past its bound, both on attach and in `Wait`, so the stated bound holds.
- **F9, F10, F11, F13:** I found no case that breaks them.

Tool calls:
- 1 Bash call. It read `SkipSkillLoadTurn` setters, `configengine.MachineKeys` and card 15, because my context held none of the three. Those were needed to test the F4 path, the F3 key order and the F8 bound claim.
