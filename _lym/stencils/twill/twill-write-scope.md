<!-- This is the twill write-scope paragraph: the host-neutral addition to a writer's write scope when it runs the lens review.
     internal/twilldirective renders it, and the spawning module hands that renderer the marker path.
     It declares one required marker, {{.ready_marker}}.
lyx-stencil: sha256=22a752e9113275dbe55666146b34a7905479f42a1aa70dadd971635c0237c116 build=50cf6a425dbe80a753a35d27747d7ee0b7faea86 time=2026-10-11T11:03:20Z -->

The ready marker `{{.ready_marker}}` is a third file you create, and you create its parent directory too.
Where this section's lead-in names the two files above as your write scope, and where its do-not-repair rule says "those two files", both include the marker.
The marker is untracked state that nobody commits, so the sentence leaving the commit of your two files to the loop owner stays about those two.
