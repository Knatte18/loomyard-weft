# Card 4 — hub-directory

**Create:**
- `plan:internal/creelengine#HubDirectory` -> `type HubDirectory interface`
- `plan:internal/creelengine#ErrHubUnresolved` -> `var ErrHubUnresolved error`
- `plan:internal/creelengine#Store.SetHubs` -> `func (s *Store) SetHubs(hubs HubDirectory, goos string)`
- `plan:internal/creelengine#daemon.recordHub` -> `func (d *daemon) recordHub()`
- `plan:internal/creelcli#hubDirectory` -> `type hubDirectory struct`
- `plan:internal/creelcli#newHubDirectory` -> `func newHubDirectory(hubPath, shortname, tempRoot, registryDir string, registryErr error) *hubDirectory`
- `plan:internal/creelcli#daemonHubs` -> `func daemonHubs(hubPath, shortname, tempRoot string) *hubDirectory`
- `plan:internal/creelcli#hubDirectory.Record` -> `func (d *hubDirectory) Record() error`
- `plan:internal/creelcli#hubDirectory.Lookup` -> `func (d *hubDirectory) Lookup(shortname string) (string, error)`
- `plan:internal/creelcli#hubDirectory.Geometry` -> `func (d *hubDirectory) Geometry(hubPath string) (creelengine.Geometry, error)`
- `plan:internal/creelcli#hubDirectory.Spawn` -> `func (d *hubDirectory) Spawn(hubPath, logPath string) (creelengine.Spawn, error)`
- `plan:internal/creelcli#hubDirectory.TrustedHubs` -> `func (d *hubDirectory) TrustedHubs() ([]string, error)`
- `plan:internal/creelcli#TestHubDirectory` -> `func TestHubDirectory(t *testing.T)`
- `plan:internal/testkit/creelfake#Hubs` -> `type Hubs struct`
- `plan:internal/testkit/creelfake#NewHubs` -> `func NewHubs() *Hubs`
- `plan:internal/testkit/creelfake#Hubs.Add` -> `func (h *Hubs) Add(shortname string, geom creelengine.Geometry)`
- `plan:internal/testkit/creelfake#Hubs.SetSpawn` -> `func (h *Hubs) SetSpawn(fn func(hubPath string) (creelengine.Spawn, error))`
- `plan:internal/testkit/creelfake#Hubs.SetTrusted` -> `func (h *Hubs) SetTrusted(shortnames []string, err error)`
- `plan:internal/testkit/creelfake#Hubs.SetRecordError` -> `func (h *Hubs) SetRecordError(err error)`
- `plan:internal/testkit/creelfake#Hubs.SetLookupError` -> `func (h *Hubs) SetLookupError(shortname string, err error)`
- `plan:internal/testkit/creelfake#Hubs.SetGeometryError` -> `func (h *Hubs) SetGeometryError(hubPath string, err error)`
- `plan:internal/testkit/creelfake#Hubs.Records` -> `func (h *Hubs) Records() int`
- `plan:internal/testkit/creelfake#Hubs.Spawns` -> `func (h *Hubs) Spawns(hubPath string) int`
- `plan:internal/testkit/creelfake#Hubs.Record` -> `func (h *Hubs) Record() error`
- `plan:internal/testkit/creelfake#Hubs.Lookup` -> `func (h *Hubs) Lookup(shortname string) (string, error)`
- `plan:internal/testkit/creelfake#Hubs.Geometry` -> `func (h *Hubs) Geometry(hubPath string) (creelengine.Geometry, error)`
- `plan:internal/testkit/creelfake#Hubs.Spawn` -> `func (h *Hubs) Spawn(hubPath, logPath string) (creelengine.Spawn, error)`
- `plan:internal/testkit/creelfake#Hubs.TrustedHubs` -> `func (h *Hubs) TrustedHubs() ([]string, error)`
- `internal/creelcli/hubdirectory_test.go`

**Edit:**
- `internal/creelengine#Options`
- `internal/creelengine#Run`
- `internal/creelengine#Store`
- `internal/creelengine#daemon`
- `internal/creelengine#daemon.serve`
- `internal/creelcli#clientSeam`
- `internal/creelcli#seam`
- `internal/creelcli#serveDaemon`
- `internal/creelcli#newRig`
- `internal/creelengine_test#rig`
- `internal/creelengine_test#rig.options`
- `internal/creelengine_test#TestDaemon_Lifecycle`

**Extends:**
- `internal/creelengine/doc.go`
- `internal/creelcli/doc.go`

**Uses:**
- `internal/hubregistry#Record`
- `internal/hubregistry#Lookup`
- `internal/hubregistry#ErrUnresolved`
- `internal/creelconfig#LoadConfig`
- `internal/hubgeom#CreelGeometry`
- `internal/standalonestate#UserStateDir`
- `internal/fabricengine#BoltDir`

**Intent:** Give the engine the one told seam every cross-hub step goes through, and record the hub when its daemon starts (D1, and `PATTERN-cwd-resolution`: creelengine stays told).

`HubDirectory` has five methods: `Record() error` records this hub; `Lookup(shortname string) (string, error)` is D1's lookup; `Geometry(hubPath string) (Geometry, error)` builds another hub's geometry; `Spawn(hubPath, logPath string) (Spawn, error)` starts that hub's daemon; `TrustedHubs() ([]string, error)` reads D5's list.
A `Lookup` error matching `ErrHubUnresolved` is D1's none or several refusal; any other is a fault of reading the record, which card 7 treats as transient.
`Options` gains `Hubs HubDirectory`.
`Store` gains the seam and the daemon's platform, set once by `SetHubs` before the scheduler starts.
A nil seam means a hub that serves no other hub: a name of another hub stays an unknown receiver, as `TestStore_Mail`'s `zz:orch` row pins, so every hand-built store and rig keeps today's behaviour.
Production always passes a seam.

`Run` calls `recordHub` right after it takes its single-instance lock, and calls `store.SetHubs` once the store is open.
A failed record logs one Warn per episode and the daemon serves on, since a hub that cannot be addressed must not lose its own mail; a daemon that lost the lock returns `ErrDaemonRunning` first and records nothing.
`daemon.serve` also calls `recordHub` after each locally resolved verb.
After a record that succeeded, `recordHub` records at most once per named `hubRecordInterval`, a minute, on the daemon's clock; after one that failed it skips the interval and records at the next such verb.
This makes D1's way forward true for a hub whose daemon already runs: a live daemon whose last record failed records the hub at the next locally resolved verb, and a live daemon whose file was removed after a record that succeeded records it again at the first such verb past `hubRecordInterval`, which is why card 1's corrupt-record way forward waits a minute.
A record is one short file-locked write, and a nil seam skips it.

`hubDirectory` in `internal/creelcli/hubdirectory.go` implements the seam: `Record` and `Lookup` over `internal/hubregistry` in the told `registryDir`, with `seam.clock.Now()`; `Lookup` maps `hubregistry.ErrUnresolved` onto an error with the same text that matches `creelengine.ErrHubUnresolved`; `Geometry` through `hubgeom.CreelGeometry` under `tempRoot`; `Spawn` through `seam.spawn`, so `spawnDaemon` logs and reaps each start; `TrustedHubs` through `creelconfig.LoadConfig` with `configengine.Dirs{Board: fabricengine.BoltDir(hubPath)}`, read at each call, so a changed list applies with no daemon restart.
`clientSeam` gains `registryDir func() (string, error)`, `standalonestate.UserStateDir` in `seam`, and `newRig` points it at `t.TempDir()`.
`daemonHubs` calls `seam.registryDir` and builds the directory, and `serveDaemon` passes it as `Hubs`.
A `registryDir` that fails is kept as `registryErr`: `Record` and `Lookup` then answer `creel: the per-user state directory cannot be read (<cause>); way forward: set XDG_STATE_HOME or HOME in the environment the hub's daemon starts from`, so a home-less hub serves its own strands, a post to another hub is refused with that cause, and a delivery to it is transient.

`creelfake.Hubs` is the shared fake: it holds shortname-to-geometry entries, answers `Lookup` with D1's refusals, matching `ErrHubUnresolved`, for an unknown shortname and for one added twice, counts `Record` and `Spawn`, and answers `TrustedHubs` as told.
`SetLookupError` and `SetGeometryError` make a lookup or geometry fail as told.
With nothing set, `Spawn` answers an error and starts nothing, `Geometry` of an unknown path answers an error, and `TrustedHubs` answers an empty list.
`rig` gains `hubs *creelfake.Hubs` and `goos string`, and `rig.options` passes them when set, so a nil `hubs` leaves every existing daemon test as it was.

`TestDaemon_Lifecycle` rows: a start records once, after the lock; a second daemon that loses the lock records nothing; a failing record still serves; after a failed record the next verb records again with the fake clock unmoved; after a record that succeeded a verb inside the interval does not record, and a verb past it, by the fake clock, does.
`TestHubDirectory` is in `internal/creelcli/hubdirectory_test.go`, which is tier0: its hubs are temporary directories with only a shortname record and a board `creel.yaml`, and `registryDir` is `t.TempDir()`.
Its rows: an unknown shortname's error matches `creelengine.ErrHubUnresolved` and a corrupt record's does not; `Geometry` names the other hub's socket; a rewritten base file reads the new list on the next `TrustedHubs` with no restart; `Spawn` hands `seam.spawn` the hub path and log path; `daemonHubs` with a failing `registryDir` answers `Lookup` and `Record` naming the cause.

**ImpactSummary:** New seam behind a nil-safe option: `Run`, `daemon.serve` and `serveDaemon` gain one call each, `OpenStore`, `NewScheduler` and every test literal of `Options` keep compiling unchanged, and only the one `clientSeam` test literal in `newRig` is touched.
