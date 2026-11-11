# Creel repeats the same notice many times for an unchanged state

`creel-repeat-notices` · Note · bug, creel, dobby, fabric

## Seen (orch, 2026-11-11)

During a network outage on OSL-1046 (`Could not resolve host: github.com`, `EAI_AGAIN`), the orch's mail held about 25 dobby notices that read the same for `testmap-verb` and `arras-recipe-perch`: same state, same `since` time, same history length.
FABRIC sent the same push failure for the same branch several times, for example the bolt branch `main` and the records branch of each pair.

## Idea

A notice whose text and run state have not changed since the last one sent to the same strand is not sent again, or is sent again only after a stated interval.
Read first where dobby and fabric decide to send, and whether a repeat is deliberate, for instance as a reminder.
