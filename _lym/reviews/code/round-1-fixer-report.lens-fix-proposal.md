```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1**: The new bullet is "Read a file whole, or every span you need from it, even though the Read tool advises reading only the part you need."
  - It names its own rule, so the override no longer depends on position, and the two "Never" bullets stay unchanged.
  - It partly repeats the preceding bullet ("a file whole, or every span you need from it, in one message").
  - The repetition drops no instruction and contradicts nothing, so it is not a defect.
- **F2**: Step 4's four sub-steps are now `1.` to `4.` at a 3-space indent.
  - Markdown renders them as a nested ordered list under item 4, with numbering restarting at 1, so they do not collide with the top-level steps 1 to 4.
  - The `\n` escapes in the JSON string are intact, and the package gate passed (89 of 91 targets passed, 0 failed, 2 skipped), so no test pins the old bullet markers.

Tool calls made: none. My context already held the changed `shipped.json` text and the gate result.
