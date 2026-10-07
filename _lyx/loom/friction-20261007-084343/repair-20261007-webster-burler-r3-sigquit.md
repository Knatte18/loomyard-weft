# Repair record: Webster-Burler round 3 step interrupted by SIGQUIT

- Run: `bugfix-webster`, `current_producer` Webster-Burler, `history_length` 36 (step 9 of 14, Webster-Review).
- Failure: `lyx shed step` trace `19887614b34dfcd8` (pid 438372) never returned an envelope.
  The round 3 agent finished at 10:23 but the step never took its Stop (#411); `ly:orch` sent SIGQUIT at 10:34 and the process exited with code 2.
- Trace lines acted on: the trace holds only `shed: step` at 2026-10-07T10:15:54.620+02:00 and no `shed: step done`; no `fabric: mutation` records, so nothing half-finished to undo.
- Status read after the exit: unchanged key, `state: running`, `last_step.finished: false`, `interrupt_policy: reinvoke`.
- Action: re-invoke `lyx shed step bugfix-webster` (counts toward the repair cap; first repair of this key), as `ly:orch` also instructed.
- The goroutine dump follows. `goroutine 1` is blocked in `verifyrun.run` -> `os/exec.Cmd.Wait`, reached from `loomshed.NewVerifyGate` via `burlerengine.repairReportBeforeGate` and `shuttleengine.evaluateGate`.

## Goroutine dump

```
SIGQUIT: quit
PC=0x494dc1 m=0 sigcode=0

goroutine 0 gp=0x17fc8c0 m=0 mp=0x17fdea0 [idle]:
runtime.futex(0x17fdff8, 0x80, 0x0, 0x0, 0x0, 0x0)
	/usr/lib/go-1.26/src/runtime/sys_linux_amd64.s:569 +0x21 fp=0x7ffd9ce96b98 sp=0x7ffd9ce96b90 pc=0x494dc1
runtime.futexsleep(0x17fdea0?, 0x9ce96c10?, 0x465208?)
	/usr/lib/go-1.26/src/runtime/os_linux.go:73 +0x30 fp=0x7ffd9ce96be8 sp=0x7ffd9ce96b98 pc=0x44feb0
runtime.notesleep(0x17fdff8)
	/usr/lib/go-1.26/src/runtime/lock_futex.go:47 +0x87 fp=0x7ffd9ce96c20 sp=0x7ffd9ce96be8 pc=0x425dc7
runtime.mPark(...)
	/usr/lib/go-1.26/src/runtime/proc.go:1967
runtime.stopm()
	/usr/lib/go-1.26/src/runtime/proc.go:3008 +0x8c fp=0x7ffd9ce96c50 sp=0x7ffd9ce96c20 pc=0x45b44c
runtime.findRunnable()
	/usr/lib/go-1.26/src/runtime/proc.go:3796 +0xec8 fp=0x7ffd9ce96e20 sp=0x7ffd9ce96c50 pc=0x45d008
runtime.schedule()
	/usr/lib/go-1.26/src/runtime/proc.go:4164 +0xb1 fp=0x7ffd9ce96e60 sp=0x7ffd9ce96e20 pc=0x45e151
runtime.park_m(0x23c93120e3c0)
	/usr/lib/go-1.26/src/runtime/proc.go:4304 +0x285 fp=0x7ffd9ce96ec0 sp=0x7ffd9ce96e60 pc=0x45e5e5
runtime.mcall()
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:496 +0x55 fp=0x7ffd9ce96ed8 sp=0x7ffd9ce96ec0 pc=0x491655

goroutine 1 gp=0x23c930c661e0 m=7 mp=0x23c930f00008 [syscall]:
syscall.Syscall6(0xf7, 0x3, 0xa, 0x23c93128c268, 0x4, 0x23c93124e2d0, 0x0)
	/usr/lib/go-1.26/src/syscall/syscall_linux.go:96 +0x39 fp=0x23c93128c1e0 sp=0x23c93128c180 pc=0x4b0459
internal/syscall/unix.Waitid(0x5994ad?, 0x23c9310884c0?, 0xb?, 0x23c931083980?, 0x3?)
	/usr/lib/go-1.26/src/internal/syscall/unix/waitid_linux.go:18 +0x39 fp=0x23c93128c238 sp=0x23c93128c1e0 pc=0x502899
os.(*Process).pidfdWait.func1(...)
	/usr/lib/go-1.26/src/os/pidfd_linux.go:108
os.ignoringEINTR(...)
	/usr/lib/go-1.26/src/os/file_posix.go:256
os.(*Process).pidfdWait(0x23c930fe2580)
	/usr/lib/go-1.26/src/os/pidfd_linux.go:107 +0x1c7 fp=0x23c93128c348 sp=0x23c93128c238 pc=0x514b27
os.(*Process).wait(0x23c931190680?)
	/usr/lib/go-1.26/src/os/exec_unix.go:25 +0x1a fp=0x23c93128c360 sp=0x23c93128c348 pc=0x50feba
os.(*Process).Wait(...)
	/usr/lib/go-1.26/src/os/exec.go:346
os/exec.(*Cmd).Wait(0x23c931190680)
	/usr/lib/go-1.26/src/os/exec/exec.go:930 +0x45 fp=0x23c93128c3c0 sp=0x23c93128c360 pc=0x599e45
os/exec.(*Cmd).Run(0x23c931190680)
	/usr/lib/go-1.26/src/os/exec/exec.go:632 +0x2d fp=0x23c93128c3d8 sp=0x23c93128c3c0 pc=0x598b6d
github.com/Knatte18/loomyard/internal/verifyrun.run({0xfeaf50, 0x181df20}, {0x23c930ee0280, 0x263}, {0x23c930da82a0, 0x30}, {0xfe3560, 0x23c930f02200}, 0x2540be400)
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/verifyrun/run.go:53 +0x295 fp=0x23c93128c520 sp=0x23c93128c3d8 pc=0xaf7f35
github.com/Knatte18/loomyard/internal/verifyrun.Run(...)
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/verifyrun/run.go:34
github.com/Knatte18/loomyard/internal/verifytree.Verify({0xfeaf50, 0x181df20}, {{0x23c930da82a0, 0x30}, {0x23c930ffd6d0, 0x4f}, {0x23c930ffd720, 0x49}, {0x23c930ffd770, 0x47}}, ...)
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/verifytree/verify.go:160 +0x648 fp=0x23c93128c728 sp=0x23c93128c520 pc=0xaf9ca8
github.com/Knatte18/loomyard/internal/loomshed.NewVerifyGate.func1()
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/loomshed/verifygate.go:44 +0x265 fp=0x23c93128cac0 sp=0x23c93128c728 pc=0xb794a5
github.com/Knatte18/loomyard/internal/burlerengine.(*Engine).Run.repairReportBeforeGate.func1()
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/burlerengine/engine.go:277 +0x70 fp=0x23c93128cc10 sp=0x23c93128cac0 pc=0xa86cd0
github.com/Knatte18/loomyard/internal/shuttleengine.(*Run).evaluateGate(0x23c930e24a88, 0x0)
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/shuttleengine/wait.go:996 +0x4e8 fp=0x23c93128ce48 sp=0x23c93128cc10 pc=0xa80608
github.com/Knatte18/loomyard/internal/shuttleengine.(*Run).handleGatedBoundary(0x23c930e24a88)
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/shuttleengine/wait.go:329 +0x79 fp=0x23c93128d0f8 sp=0x23c93128ce48 pc=0xa7ba99
github.com/Knatte18/loomyard/internal/shuttleengine.(*Run).Wait(0x23c930e24a88)
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/shuttleengine/wait.go:259 +0x43f fp=0x23c93128d468 sp=0x23c93128d0f8 pc=0xa7a8ff
github.com/Knatte18/loomyard/internal/shuttleengine.(*Runner).RunGated(_, {{0x23c930e15500, 0xa39}, {0x23c93103bb20, 0x2, 0x2}, {0x23c930c76d50, 0x6}, {0x23c930e69107, 0x4}, ...}, ...)
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/shuttleengine/run.go:643 +0x290 fp=0x23c93128d758 sp=0x23c93128d468 pc=0xa74270
github.com/Knatte18/loomyard/internal/burlerengine.(*Engine).Run(_, {{{0x181df20, 0x0, 0x0}, {0x23c930d7cc60, 0x11f}}, {{0x23c930f41860, 0x1, 0x1}, {0x23c931066000, ...}}, ...}, ...)
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/burlerengine/engine.go:197 +0xea2 fp=0x23c93128de60 sp=0x23c93128d758 pc=0xa86362
github.com/Knatte18/loomyard/internal/shedadapters.(*BurlerProducer).Call(0x23c931193340, {0xfeb0a0, 0x23c930ebecf0})
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/shedadapters/burler.go:419 +0x1424 fp=0x23c93128ee88 sp=0x23c93128de60 pc=0xb5a124
github.com/Knatte18/loomyard/internal/shedengine.(*Shed).stepLocked(0x23c930e6b000, {0xfeb0a0, 0x23c930ebecf0})
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/shedengine/run.go:232 +0xa32 fp=0x23c93128f500 sp=0x23c93128ee88 pc=0x6b7fb2
github.com/Knatte18/loomyard/internal/shedengine.(*Shed).Step(0x23c930e6b000, {0xfeb0a0, 0x23c930ebecf0})
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/shedengine/run.go:471 +0x28e fp=0x23c93128f6d8 sp=0x23c93128f500 pc=0x6bb06e
github.com/Knatte18/loomyard/internal/shedverbs.stepCmd.func1(0x23c930ed2008, {0x23c930ec4690?, 0x4?, 0xf3e2d5?})
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/shedverbs/step.go:240 +0x4a5 fp=0x23c93128fc28 sp=0x23c93128f6d8 pc=0xbdac05
github.com/spf13/cobra.(*Command).execute(0x23c930ed2008, {0x23c930ec4660, 0x1, 0x1})
	/home/knatte/go/pkg/mod/github.com/spf13/cobra@v1.10.2/command.go:1015 +0xb14 fp=0x23c93128fdd0 sp=0x23c93128fc28 pc=0x6a2234
github.com/spf13/cobra.(*Command).ExecuteC(0x23c930e1a308)
	/home/knatte/go/pkg/mod/github.com/spf13/cobra@v1.10.2/command.go:1148 +0x465 fp=0x23c93128fec0 sp=0x23c93128fdd0 pc=0x6a2b65
github.com/spf13/cobra.(*Command).Execute(...)
	/home/knatte/go/pkg/mod/github.com/spf13/cobra@v1.10.2/command.go:1071
github.com/spf13/cobra.(*Command).ExecuteContext(...)
	/home/knatte/go/pkg/mod/github.com/spf13/cobra@v1.10.2/command.go:1064
github.com/Knatte18/loomyard/internal/clihelp.RunRootCtx({0xfeaf50, 0x181df20}, 0x23c930e1a308, {0xfe3560, 0x23c930c64050})
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/clihelp/exec.go:175 +0xa9 fp=0x23c93128ff08 sp=0x23c93128fec0 pc=0xbd4d89
github.com/Knatte18/loomyard/internal/clihelp.RunRoot(...)
	/home/knatte/Code/loomyard-LYXHUB/loomyard/internal/clihelp/exec.go:191
main.main()
	/home/knatte/Code/loomyard-LYXHUB/loomyard/cmd/lyx/main.go:46 +0xa5 fp=0x23c93128ff48 sp=0x23c93128ff08 pc=0xcbc845
runtime.main()
	/usr/lib/go-1.26/src/runtime/proc.go:290 +0x2d5 fp=0x23c93128ffe0 sp=0x23c93128ff48 pc=0x456635
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c93128ffe8 sp=0x23c93128ffe0 pc=0x4932a1

goroutine 2 gp=0x23c930c66d20 m=nil [force gc (idle), 17 minutes]:
runtime.gopark(0x0?, 0x0?, 0x0?, 0x0?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf6fa8 sp=0x23c930cf6f88 pc=0x48ba8e
runtime.goparkunlock(...)
	/usr/lib/go-1.26/src/runtime/proc.go:468
runtime.forcegchelper()
	/usr/lib/go-1.26/src/runtime/proc.go:375 +0xb3 fp=0x23c930cf6fe0 sp=0x23c930cf6fa8 pc=0x456953
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf6fe8 sp=0x23c930cf6fe0 pc=0x4932a1
created by runtime.init.7 in goroutine 1
	/usr/lib/go-1.26/src/runtime/proc.go:363 +0x1a

goroutine 3 gp=0x23c930c672c0 m=nil [GC sweep wait]:
runtime.gopark(0x17fb501?, 0x0?, 0x0?, 0x0?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf7788 sp=0x23c930cf7768 pc=0x48ba8e
runtime.goparkunlock(...)
	/usr/lib/go-1.26/src/runtime/proc.go:468
runtime.bgsweep(0x23c930ca6180)
	/usr/lib/go-1.26/src/runtime/mgcsweep.go:324 +0x151 fp=0x23c930cf77c8 sp=0x23c930cf7788 pc=0x440611
runtime.gcenable.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:214 +0x17 fp=0x23c930cf77e0 sp=0x23c930cf77c8 pc=0x4319b7
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf77e8 sp=0x23c930cf77e0 pc=0x4932a1
created by runtime.gcenable in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:214 +0x66

goroutine 4 gp=0x23c930c674a0 m=nil [GC scavenge wait]:
runtime.gopark(0x10000?, 0x8942b1?, 0x0?, 0x0?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf7f78 sp=0x23c930cf7f58 pc=0x48ba8e
runtime.goparkunlock(...)
	/usr/lib/go-1.26/src/runtime/proc.go:468
runtime.(*scavengerState).park(0x17fc1e0)
	/usr/lib/go-1.26/src/runtime/mgcscavenge.go:425 +0x49 fp=0x23c930cf7fa8 sp=0x23c930cf7f78 pc=0x43e089
runtime.bgscavenge(0x23c930ca6180)
	/usr/lib/go-1.26/src/runtime/mgcscavenge.go:658 +0x59 fp=0x23c930cf7fc8 sp=0x23c930cf7fa8 pc=0x43e619
runtime.gcenable.gowrap2()
	/usr/lib/go-1.26/src/runtime/mgc.go:215 +0x17 fp=0x23c930cf7fe0 sp=0x23c930cf7fc8 pc=0x431977
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf7fe8 sp=0x23c930cf7fe0 pc=0x4932a1
created by runtime.gcenable in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:215 +0xa5

goroutine 5 gp=0x23c930c67a40 m=nil [GOMAXPROCS updater (idle), 17 minutes]:
runtime.gopark(0x0?, 0x0?, 0x0?, 0x0?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf8788 sp=0x23c930cf8768 pc=0x48ba8e
runtime.goparkunlock(...)
	/usr/lib/go-1.26/src/runtime/proc.go:468
runtime.updateMaxProcsGoroutine()
	/usr/lib/go-1.26/src/runtime/proc.go:7095 +0xe7 fp=0x23c930cf87e0 sp=0x23c930cf8788 pc=0x464ec7
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf87e8 sp=0x23c930cf87e0 pc=0x4932a1
created by runtime.defaultGOMAXPROCSUpdateEnable in goroutine 1
	/usr/lib/go-1.26/src/runtime/proc.go:7083 +0x37

goroutine 6 gp=0x23c930d321e0 m=nil [finalizer wait, 17 minutes]:
runtime.gopark(0x0?, 0xfd7eb0?, 0x0?, 0x2?, 0x2000000020?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf6620 sp=0x23c930cf6600 pc=0x48ba8e
runtime.runFinalizers()
	/usr/lib/go-1.26/src/runtime/mfinal.go:210 +0x107 fp=0x23c930cf67e0 sp=0x23c930cf6620 pc=0x430927
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf67e8 sp=0x23c930cf67e0 pc=0x4932a1
created by runtime.createfing in goroutine 1
	/usr/lib/go-1.26/src/runtime/mfinal.go:172 +0x3d

goroutine 7 gp=0x23c930d32780 m=nil [cleanup wait, 17 minutes]:
runtime.gopark(0x0?, 0x0?, 0x0?, 0x0?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf8f68 sp=0x23c930cf8f48 pc=0x48ba8e
runtime.goparkunlock(...)
	/usr/lib/go-1.26/src/runtime/proc.go:468
runtime.(*cleanupQueue).dequeue(0x17fc560)
	/usr/lib/go-1.26/src/runtime/mcleanup.go:522 +0xd4 fp=0x23c930cf8fa0 sp=0x23c930cf8f68 pc=0x42d774
runtime.runCleanups()
	/usr/lib/go-1.26/src/runtime/mcleanup.go:718 +0x45 fp=0x23c930cf8fe0 sp=0x23c930cf8fa0 pc=0x42dde5
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf8fe8 sp=0x23c930cf8fe0 pc=0x4932a1
created by runtime.(*cleanupQueue).createGs in goroutine 1
	/usr/lib/go-1.26/src/runtime/mcleanup.go:672 +0xa5

goroutine 10 gp=0x23c930d32f00 m=nil [GC worker (idle), 17 minutes]:
runtime.gopark(0x0?, 0x0?, 0x0?, 0x0?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf2740 sp=0x23c930cf2720 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c930cf27c8 sp=0x23c930cf2740 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c930cf27e0 sp=0x23c930cf27c8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf27e8 sp=0x23c930cf27e0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 11 gp=0x23c930d330e0 m=nil [GC worker (idle), 17 minutes]:
runtime.gopark(0x0?, 0x0?, 0x0?, 0x0?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf2f40 sp=0x23c930cf2f20 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c930cf2fc8 sp=0x23c930cf2f40 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c930cf2fe0 sp=0x23c930cf2fc8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf2fe8 sp=0x23c930cf2fe0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 12 gp=0x23c930d332c0 m=nil [GC worker (idle), 17 minutes]:
runtime.gopark(0x0?, 0x0?, 0x0?, 0x0?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf3740 sp=0x23c930cf3720 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c930cf37c8 sp=0x23c930cf3740 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c930cf37e0 sp=0x23c930cf37c8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf37e8 sp=0x23c930cf37e0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 13 gp=0x23c930d334a0 m=nil [GC worker (idle)]:
runtime.gopark(0xbb486cce775?, 0x3?, 0x88?, 0x25?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf3f40 sp=0x23c930cf3f20 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c930cf3fc8 sp=0x23c930cf3f40 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c930cf3fe0 sp=0x23c930cf3fc8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf3fe8 sp=0x23c930cf3fe0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 14 gp=0x23c930d33680 m=nil [GC worker (idle)]:
runtime.gopark(0xbb486cba125?, 0x3?, 0x6f?, 0x7b?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf4740 sp=0x23c930cf4720 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c930cf47c8 sp=0x23c930cf4740 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c930cf47e0 sp=0x23c930cf47c8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf47e8 sp=0x23c930cf47e0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 15 gp=0x23c930d33860 m=nil [GC worker (idle), 1 minutes]:
runtime.gopark(0xbb486cba5f5?, 0x1?, 0x22?, 0xd3?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf4f40 sp=0x23c930cf4f20 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c930cf4fc8 sp=0x23c930cf4f40 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c930cf4fe0 sp=0x23c930cf4fc8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf4fe8 sp=0x23c930cf4fe0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 16 gp=0x23c930d33a40 m=nil [GC worker (idle)]:
runtime.gopark(0xbb486ccd393?, 0x1?, 0x1d?, 0x2?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf5740 sp=0x23c930cf5720 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c930cf57c8 sp=0x23c930cf5740 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c930cf57e0 sp=0x23c930cf57c8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf57e8 sp=0x23c930cf57e0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 18 gp=0x23c930d33c20 m=nil [GC worker (idle)]:
runtime.gopark(0xbb486cba21f?, 0x3?, 0x9b?, 0x3b?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c930cf5f40 sp=0x23c930cf5f20 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c930cf5fc8 sp=0x23c930cf5f40 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c930cf5fe0 sp=0x23c930cf5fc8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c930cf5fe8 sp=0x23c930cf5fe0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 19 gp=0x23c93120e000 m=nil [GC worker (idle)]:
runtime.gopark(0xbb486ccbe03?, 0x3?, 0xdc?, 0x31?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c931214740 sp=0x23c931214720 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c9312147c8 sp=0x23c931214740 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c9312147e0 sp=0x23c9312147c8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c9312147e8 sp=0x23c9312147e0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 20 gp=0x23c93120e1e0 m=nil [GC worker (idle)]:
runtime.gopark(0xbb486cba953?, 0x3?, 0x72?, 0x78?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c931214f40 sp=0x23c931214f20 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c931214fc8 sp=0x23c931214f40 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c931214fe0 sp=0x23c931214fc8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c931214fe8 sp=0x23c931214fe0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 21 gp=0x23c93120e3c0 m=nil [GC worker (idle)]:
runtime.gopark(0xbb486cc1850?, 0x1?, 0x68?, 0x16?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c931215740 sp=0x23c931215720 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c9312157c8 sp=0x23c931215740 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c9312157e0 sp=0x23c9312157c8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c9312157e8 sp=0x23c9312157e0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

goroutine 22 gp=0x23c93120e5a0 m=nil [GC worker (idle), 2 minutes]:
runtime.gopark(0xb9bfee6fecf?, 0x3?, 0x1d?, 0xb?, 0x0?)
	/usr/lib/go-1.26/src/runtime/proc.go:462 +0xce fp=0x23c931215f40 sp=0x23c931215f20 pc=0x48ba8e
runtime.gcBgMarkWorker(0x23c930ca3730)
	/usr/lib/go-1.26/src/runtime/mgc.go:1791 +0xeb fp=0x23c931215fc8 sp=0x23c931215f40 pc=0x43448b
runtime.gcBgMarkStartWorkers.gowrap1()
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x17 fp=0x23c931215fe0 sp=0x23c931215fc8 pc=0x434377
runtime.goexit({})
	/usr/lib/go-1.26/src/runtime/asm_amd64.s:1771 +0x1 fp=0x23c931215fe8 sp=0x23c931215fe0 pc=0x4932a1
created by runtime.gcBgMarkStartWorkers in goroutine 1
	/usr/lib/go-1.26/src/runtime/mgc.go:1695 +0x105

rax    0xca
rbx    0x0
rcx    0x494dc3
rdx    0x0
rdi    0x17fdff8
rsi    0x80
rbp    0x7ffd9ce96bd8
rsp    0x7ffd9ce96b90
r8     0x0
r9     0x0
r10    0x0
r11    0x286
r12    0x23c930ce6008
r13    0x1
r14    0x17fc8c0
r15    0x5
rip    0x494dc1
rflags 0x286
cs     0x33
fs     0x0
gs     0x0

[exited with code 2]
```
