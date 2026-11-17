```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each fix the fixer carried out as the reviewer proposed (F1–F6). F7 was fixed with the fixer's own wording, so it is out of this lens's scope.

**MEDIUM: D6 third bullet (F2) contradicts itself**
- **Changed text:** "`fabricengine.RegenerateHubWorkspace` passes them through unread to `internal/vscode`" and, two lines later, "The beam file's generated mark is fabric's own, so fabric refuses a member named as that mark the same way."
- **Summary:** fabric cannot pass the members through unread and also refuse one by its name.
- **Evidence:** both sentences are in the same bullet. Refusing the mark means fabric reads every member's key. The proposal itself asks for both ("passes … through unread" and "stays refused in fabric"), so the contradiction comes from the proposal. Either say that fabric reads only the member names, to check for its mark, or move the mark check into the builder by passing the reserved key in.

**LOW: D6 first bullet and its criterion (F4)**
- **Changed text:** "Its presentation sets `"reveal": "always"` and `"focus": false`."
- **Summary:** the record no longer says that the terminal panel now opens every time the key is pressed. With `silent`, the panel stayed hidden when the verb succeeded.
- **Evidence:** the proposal picked "reveal always" to make failures visible. The cost is that every successful `alt+shift+9` now opens the terminal panel over the editor. Neither the D6 rationale nor its bound states that this is accepted. A reader comparing it with `reed attach` would notice the change in behaviour.

**LOW: Added after Discussion (arbiter) "Per-body README folds are moot" (F1)**
- **Changed text:** the removal of D7, while that arbiter entry was left untouched as the proposal required.
- **Summary:** the untouched arbiter entry still says "as D7 states" and "D7's per-section Tasks, Notes and Done folds stand until the parent answers it", but D7 no longer exists.
- **Evidence:** in decision-record.md, that arbiter entry and the parent's entry "D7's collapsible sections stay only if another goal of the entry needs them" both point at a removed decision. Nothing in the main record says that D7 was dropped because of the parent's entry. The reason is recorded only in support-log.md. A one-line note where D7 used to be, such as "D7 removed after the parent's phone answer", would keep both references resolvable without editing the added entries.

I found nothing wrong with the F3, F5 or F6 proposals.

Tool calls: none.
