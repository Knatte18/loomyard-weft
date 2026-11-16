```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: no findings.

I checked these sets:
- **F8 (record writers and readers):** the non-test uses of the remote-control record are `recordRemoteControl` (`start.go:666`, `start.go:754`, `remotecontrol.go:144`), `ReadRemoteControl` (`remotecontrol.go:209`, `start.go:749`) and `RemoteControlOn` (`remotecontrol.go:134`).
  - Every write goes through `recordRemoteControl`, which now has the engine guard.
  - The only reader of the "no record means no word" rule is `remoteControlBarWordAt`.
- **F8 (docs):** the other places that mention the button or the bar word still agree with the new behavior.
  - `orchcli/doc.go:263-278`, the `rc-bar` and `rc-on` help text, and `orchengine/doc.go:120` match.
  - `reedengine/doc.go:203` ("drawn while its state line holds Lit or Unlit") and `panebin.go:297` are still accurate.
  - `template.yaml:11-12` ("turns it on in a session running without it") is not contradicted by the new refusal on a lit button.
- **F6:** `adoptListedStrand` has two callers (`start.go:685` and `start.go:694`), which are the spawn-watcher branch and the default branch. The carry-over sits inside it, so both are covered.
- **F1:** the paste-path test wrappers that embed `SessionCycler` are `parserlessPasteEngine` and `clearlessPasteEngine`. Both now embed `RemoteController` and their constructors set it. Every `checkPastedSessionCommand` caller, including the new `TurnOnRemoteControl` one, runs over `slashPasteEngine`, which has the method.

Tool calls:
1. grep of non-test callers of `RemoteControlOn`, `recordRemoteControl`, `ReadRemoteControl` and `adoptListedStrand`, because the F6 and F8 call-site sets were not in my context.
2. grep for `rc-off`, `rc-on`, `RC button` and `remote-control.json` across `.go`, `.md` and `.yaml` files, to find every doc or config site that describes the button's state.
3. grep for `SessionCycler`-embedding test wrappers and `checkPastedSessionCommand` callers, to check the F1 set of engine shapes.
