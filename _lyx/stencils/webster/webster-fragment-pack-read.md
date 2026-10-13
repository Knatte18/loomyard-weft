<!-- This is the pack read line of webster's fork prompt.
     RenderForkPrompt reads it through stencilstore when begin-batch built a kick-start pack, fills its one marker, strips this banner and trailing newline,
     and fills the text into the optional kickstart_pack marker of webster-body-implementer, which ends the prompt, so the read line is the prompt's last line.
     The one marker is pack_path, the batch's pack file, which holds the one-turn read instruction and the pack rendered from webster-fragment-pack-section.
lyx-stencil: sha256=873f37459645e7979e441026f6933192b1a8e33a810f529bb93b4abd5aa41ba3 build=1e39afc10eb44299d7bb150adf86bcb6bf84a8a8 time=2026-10-13T14:42:47Z -->

## Start here

Now read `{{.pack_path}}` and do what it says.
