# A landing commit message ends with a stray Created using block

`landing-message-stray-block` · Task · bug, landing · draft

## Seen (orch, 2026-11-01)

The landing commit `229638f7` ("Rename the Bolt checkout directory from _board to _bolt") ends with a stray block after the "To check by hand" list: a line "Created using", then the subject again and several lines copied from the message's own body, before the credit lines.
`git log -1 --format=%B 229638f7` shows it.
The plain credit block of `commit-signature-credit` should be the only thing after the body; something in how the landing message is assembled appended a fragment of the description.
