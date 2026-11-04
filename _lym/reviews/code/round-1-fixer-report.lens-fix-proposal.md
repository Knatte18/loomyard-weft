```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1 (replace "Sketch" with "decision record"):** I found no case where the change is wrong. The sentence still reads correctly with `--record`. `ly:board` and the `discussed-task-split` note already use "decision record". The search found no other file that uses "Sketch" for this.
- **F2 (drop "discuss before writing", the reviewer's alternative):** I found no case where the change is wrong.
  - Adding the rule under `## Notes` instead would have contradicted `SKILL.md:15`, "A proposal goes on the board as a note at once". Dropping the phrase was the sound choice.
  - The button session still gets the rule from its own section, `SKILL.md:57`, because it is opened with `--skill ly:board-policy`.
  - The `description` and the INDEX row still match each other.
  - The search found no other copy of the dropped phrase. `docs/overview.md:373`, `contracts/orchrole/orch-role.md:206` and `internal/orchcli/doc.go:21` describe the skill without quoting it.

Tool calls:
1. Searched the whole repo outside `_lym` and `.git` for "discuss before writing" and `board-policy`. My context lacked any search outside `plugins` and `contracts`.
