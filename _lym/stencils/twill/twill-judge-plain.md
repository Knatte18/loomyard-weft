<!-- This is the twill plain-judge part: the host-neutral steps that have the writer judge each lens finding and record whether it was fixed or rejected.
     internal/twilldirective renders it into the twill directive for a host that asks for no critical fixer, and the spawning module hands that renderer every host value, so this block names no file, step, verb or section of any one host.
     It declares these required markers:
     {{.rejection_rule}} and {{.disposition_place}}.
lyx-stencil: sha256=178ba11c199874b980ac6a096a16d6dc0f51a0d8f83c0d66eecb4dd8b1545881 build=2abb06c5ee3473f283f0b526362161487b9e6b26 time=2026-10-15T06:35:00Z -->

4. Judge every finding with the same skepticism, and merge duplicates across lenses.
   {{.rejection_rule}}
5. Record the lens review at `{{.disposition_place}}`: one line per finding, with its lens and whether it was fixed or rejected and why.
   Name a fork that returned no report.
