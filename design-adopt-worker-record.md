# Adopt runs as a detached, step-recorded worker that pauses every adopted run and writes the owner before the deploy

`adopt-worker-record` · Task · bug, batten, tender, dobby, orch, fabric, shortcut · running · Seed-Child

Decision record: [Sketch](decision-adopt-worker-record.md)

Filed from the Sketch `adopt-robust`.

Takes: today's adoptcli steps, holds.json hold and green end; testmap-dobby-refresh's landed changes to dobbyengine, beamcli, orchcli and runproc
Delivers: lym adopt as a hidden detached worker the caller follows; the adopt record adopt.json and adopt.lock under fabricengine.AdoptScratchDir with resume and --discard; the D3 step order with the owner write before the deploy; the adopt pause entry replacing the hold and holds.json; --no-orch and its orch-session default; a green check that only reports; the adopt section of lym beam status
Exports: internal/adoptrecord (the record type, its path and a lock-free read, which neither engine import of adoptcli needs), pauserecord.AdoptCaller, the orch-skip field of orchcli.BeamUpOptions, a runproc adopt role

The part changes these packages:

- internal/adoptcli (about 1500 changed lines)
- internal/adoptrecord (about 250 changed lines)
- internal/pauserecord (about 60 changed lines)
- internal/tenderengine (about 20 changed lines)
- internal/orchcli (about 40 changed lines)
- internal/dobbyengine (about 60 changed lines)
- internal/beamcli (about 200 changed lines)
- internal/runproc (about 10 changed lines)
- pattern (about 30 changed lines)
