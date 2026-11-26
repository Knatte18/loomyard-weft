# The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails

`area-apparmor-barrier` · Task · feature, process · draft

Decision record: [Sketch](decision-area-apparmor-barrier.md)

Depends on: [`area-seccomp-core`](design-area-seccomp-core.md)

Filed from the Sketch `sandbox-closed-area-build`.

Takes: the core part's supervisor, launcher, attach client, area file with its barrier key, and the start check's barrier hook; the OSL-1046 probe result of D14 step 1
Delivers: lym area profile rendering the hub's profile from the area file's lists and the Landlock paths, with other hubs' creel sockets as one creel-*.sock pattern and the name a hash of the text, and naming the root load command; the launcher and attach client entering it through aa-exec, each still installing a filter with D4's foreign-ABI, io_uring_setup and clone3 rules, no connect rule, no listener, and TSYNC with TSYNC_ESRCH (Added after Discussion); the start refusing while no loaded profile carries the rendered name; the inside test's apparmor label branch exercised end to end
Exports: areaengine's apparmor barrier and its profile renderer

The part changes these packages:

- internal/areaengine (about 350 changed lines)
- internal/areacli (about 100 changed lines)
