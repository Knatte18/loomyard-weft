<!-- This is the twill relaunch part: the host-neutral paragraphs that say what a writer relaunched around its lens spawn call does, for a host that runs no writer fork.
     internal/twilldirective renders it into the twill directive, and the spawning module hands that renderer every host value, so this block names no file, step, verb or section of any one host.
     It declares these required markers:
     {{.relaunch_files}}, {{.format_step}}, {{.self_check_step}}, {{.self_check_command}} and {{.disposition_place}}.
lym-stencil: sha256=cc04048fd52acde9bcbe7516ffa7de7f5ccc15bf0dd11d78dc5b4fc27db6f1a6 build=ef912d4155479098331ad98253e9a38250450e09 time=2026-10-27T07:47:27Z -->

A writer relaunched after its process died before its Agent call still spawns once.
On its next turn it reads back every reviewed file it did not write in this session, so the forks inherit them.
It then finishes writing {{.relaunch_files}} per `{{.format_step}}`, runs `{{.self_check_command}}` per `{{.self_check_step}}` until it passes, and then runs this procedure from its first step.

A writer relaunched after its process died at or after its Agent call has lost every fork it spawned, and never spawns again.
On its next turn it resumes at the first of the steps from saving the fork reports onward that it had not finished, with the reports it already holds.
It records at `{{.disposition_place}}` that the lens review was cut short, naming each fork that had not returned as "no report".
