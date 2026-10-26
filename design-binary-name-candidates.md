# Candidate names for the lyx binary: weaving-rooted, clash-checked (research, 2026-10-26)

`binary-name-candidates` · Note · research

# Binary name research: replacing `lyx`

Researched 2026-10-26.
About 85 candidates were checked; 11 are classed as clear.
"Clear" means the checks below found no packaged binary of that name, no GitHub repository of that name above roughly 50 stars, no file extension and no awkward meaning in English or Norwegian.
It does not mean the name is unused anywhere: every short string has a few zero-star repositories.

## Ranked shortlist (clear)

### 1. `vef`

- Derivation: Old Norse *vefr* and Icelandic *vefur* (web, woven cloth), from *vefa* (to weave); it shares a root with English *weave*, *weft* and *web*.
- Reading and typing: reads "vef", like "Bev" with a v; it is three letters typed with the left hand alone on QWERTY (v-e-f), which is fast. Hub suffix `-VEFHUB`.
- Clash check: no local command, no apt, Homebrew formula or cask, Repology, AUR, Scoop, Chocolatey, crates.io, PyPI or Go module hit.
  npm has an unrelated `vef` package, and JSR has a `@vef` scope ([@vef/easy-cli](https://jsr.io/@vef/easy-cli/doc/~/EasyCli)), a Deno library namespace, not a binary.
  The largest GitHub repository of that name is [ChuanyiZ/vef](https://github.com/ChuanyiZ/vef) (5 stars, a VCF variant filter).
  No file extension on [fileinfo.com](https://fileinfo.com/extension/vef).
  It means nothing in English or Norwegian; VEF is also a historical Latvian electronics maker, which does no harm.

### 2. `veft`

- Derivation: Norwegian, Danish and Swedish *veft* (weft); Norwegian textile writing gives *veft* or *innslag* for weft ([snl.no: veving](https://snl.no/veving)).
- Reading and typing: reads "veft"; four letters, all left hand. Hub suffix `-VEFTHUB`.
- Clash check: no hit in any registry checked, including npm and PyPI; the GitHub repositories of that name have 0 stars; no file extension; the web search found no tool ([search notes](#how-the-checks-were-made)).
- Caveat: the records repository is already named `loomyard-weft`, so `veft` next to `weft` invites confusion between the binary and the records repository.

### 3. `wvn`

- Derivation: *woven*, with the vowels dropped.
- Reading and typing: spoken letter by letter, "double-u vee en", or "woven"; easy to type, with w and v on the left hand and n on the right. Hub suffix `-WVNHUB`.
- Clash check: no hit in any registry checked, npm and PyPI included; only 0-star GitHub repositories; no file extension; the web search found no tool of that name.
- Caveat: with no vowel, people may misspell it as `wnv` or `vwn`.

### 4. `kett`

- Derivation: German *Kette* (warp); *Kettfaden* is a warp thread.
- Reading and typing: reads "ket"; four letters, double t. Hub suffix `-KETTHUB`.
- Clash check: no hit on npm, PyPI, crates.io, Homebrew, Repology, AUR, Scoop, Chocolatey or pkg.go.dev, and no GitHub repository of that name was found; no file extension.
  The web search turned up only near names (`kitt`, `kettle`, and the PyPI `ket` Bitbucket client).
- Caveat: `ket` (Dirac notation, an AUR `ket-git` exposure tracker) is close.

### 5. `kud`

- Derivation: Finnish *kude* (weft).
- Reading and typing: reads "kood"; three letters, typed well across both hands. Hub suffix `-KUDHUB`.
- Clash check: no apt, Homebrew, Repology, AUR, crates.io, PyPI, Scoop or Chocolatey hit.
  npm `kud` is an obscure "kmdjs bundler"; pkg.go.dev lists two obscure modules, `github.com/GoLangStackDev/kud` and `github.com/wangyanglinux/kud`, with no notable stars; no file extension.
  The web search for "kud command line tool" returned only Apache Kudu's `kudu` CLI, which is a near name, not the same one.
- Caveat: the Finnish derivation is the weakest weaving tie on this list for a Norwegian team.

### 6. `vevr`

- Derivation: Norwegian *vever* (weaver), from *vev* (loom, web).
- Reading and typing: reads "vever"; four letters, left hand plus r. Hub suffix `-VEVRHUB`.
- Clash check: no hit in any registry; only 0-star GitHub repositories; no file extension; the web search found no tool of that name.
- Caveat: it reads as a typo of `vev`, which has a minor clash (see the table).

### 7. `wov`

- Derivation: *woven*, and *woof*, the old English word for weft.
- Reading and typing: reads "wove"; three letters. Hub suffix `-WOVHUB`.
- Clash check: no apt, Homebrew, Repology, AUR, crates.io, PyPI, Scoop or Chocolatey hit; npm has an unrelated `wov` package; the GitHub repositories of that name have 1 star or fewer; no file extension.
  The web search found `wovo` ([@gowovo/wovo](https://socket.dev/npm/package/@gowovo/wovo)), an HTML deploy CLI and MCP server, as the nearest name.
- Caveat: it looks like a typo for "wow".

### 8. `vefa`

- Derivation: Old Norse and Icelandic *vefa* (to weave).
- Reading and typing: reads "VEH-fa"; four letters, left hand. Hub suffix `-VEFAHUB`.
- Clash check: no hit in any registry except an unrelated npm `vefa` package; only 0-star GitHub repositories; no file extension.
  The web search's nearest name was Volcengine's `vefaas` serverless CLI.
- Caveat: *Vefa* is a Turkish given name and an Istanbul district, meaning "loyalty"; that is harmless, but the name is not unique to us.

### 9. `skje`

- Derivation: Norwegian *skje* or *vevskje* (the reed) ([snl.no: vevskje](https://snl.no/vevskje)).
- Reading and typing: reads "sheh" in Norwegian; an English reader will not know how to say it, and the j is an unusual key in the middle of a word. Hub suffix `-SKJEHUB`.
- Clash check: no hit in any registry; only 0-star GitHub repositories; no file extension; the web search found no tool of that name.
- Caveat: everyday Norwegian *skje* means "spoon" and "happen", so the weaving sense is not what a Norwegian reader thinks of first.

### 10. `rokk`

- Derivation: Norwegian *rokk* (spinning wheel), a textile word but from spinning, not weaving.
- Reading and typing: reads "rock"; four letters. Hub suffix `-ROKKHUB`.
- Clash check: no hit in any registry, no GitHub repository of that name and no file extension.
- Caveat: the near name `rok` is taken by two Rust CLIs, the Rok framework's `rok-cli` ([docs.rs](https://docs.rs/rok-cli)) and a JSON task runner, so `rokk` will be confused with them.

### 11. `lmy`

- Derivation: an abbreviation of *LooMYard*; it has no weaving meaning of its own.
- Reading and typing: spoken letter by letter; three letters. Hub suffix `-LMYHUB`.
- Clash check: no hit in any registry except an unrelated npm `lmy` package; GitHub repositories of that name have 4 stars or fewer; no file extension.
- Caveat: the name carries no image and is the least memorable on this list.

## All other candidates

| Name | Derivation | Verdict | Reason |
| --- | --- | --- | --- |
| `vev` | Norwegian *vev* (loom, web) | minor clash | AUR `vev-bin` is a terminal multiplexer called `vev` ([git.bnema.dev/vev](https://git.bnema.dev/vev), 0 votes), which is awkward next to our tmux use; npm and PyPI also have small `vev` libraries. Otherwise the best derivation of all; worth a second look if the AUR tool stays obscure. |
| `lom` | *loom*; Norwegian *lom* is the loon bird and a municipality | minor clash | [LackOfMorals/lom](https://github.com/LackOfMorals/lom) is a Go `lom` binary, a Neo4j CLI "used by people and by AI agents", published on PyPI (`pip install lom`) and updated in 2026; it has 0 stars but is a direct binary-name clash. In Russian *лом* means crowbar or scrap. |
| `hed` | heddle | minor clash | [credmp/hed](https://github.com/credmp/hed) is a hosts-file editor installed as `cargo install hed` (13 stars, crates.io); AUR `hed-git` is a hex editor; [s9xie/hed](https://github.com/s9xie/hed) is HED edge detection (1.9k stars); HED is also a neuroscience annotation standard (`hedtools`). In Danish *hed* means hot. |
| `rya` | Scandinavian *rya* rug weave | minor clash | [apache/rya](https://github.com/apache/rya) is an Apache RDF store (115 stars); PyPI `rya` is a Python app bootstrapper. |
| `vov` | Danish *vov* (a dog's bark), punning on *woof* (weft) | minor clash | Altair Accelerator, an EDA job scheduler, is known as VOV and has a `vov*` command family ([Altair docs](https://help.altair.com/accelerator/html/vov/topics/vov/other_files.htm)); [gen2brain/vov](https://github.com/gen2brain/vov) is a game (30 stars). |
| `pirn` | pirn, the weft bobbin in a shuttle | minor clash | PyPI `pirn` and `pirn-core` are an active "pipeline framework" library (2026), which is close to our domain. |
| `hov` | Norwegian *hovel* (heddle) ([snl.no](https://snl.no/hovel)) | minor clash | AUR `hov-qt` is a music client, PyPI `hov` is a set of AI app libraries; Norwegian *hov* is a pagan temple. |
| `bur` | burler; Norwegian *bur* is a cage | minor clash | npm `bur` is a React/GraphQL framework; obscure Go modules use the name; the Norwegian "cage" sense is a little unwelcome. |
| `loi` | Finnish *loimi* (warp) | minor clash | French *loi* means law; obscure Go modules use the name. |
| `ikat` | ikat dyed weave | minor clash | TREC iKAT benchmark ([irlabamsterdam/iKAT](https://github.com/irlabamsterdam/iKAT)); PyPI and crates.io `ikat` packages exist; ikat is a dyeing technique more than a weave. |
| `skn` | skein | minor clash | `.skn` is a skin-file extension ([fileinfo](https://fileinfo.com/extension/skn)); it reads as "skin". |
| `wft` | weft | minor clash | It reads as a typo of "wtf"; PyPI `wft` is a Whisper fine-tuning tool. |
| `lyr` | the "ly" prefix plus r | minor clash | crates.io `lyr` lyrics tool, AUR `lyr-bin` desktop status bar ([kryptic-sh/lyr](https://github.com/kryptic-sh/lyr)); Norwegian *lyr* is the pollock fish. |
| `twil` | twill | minor clash | npm `twil` is a Twitter CLI; it is too close to Twilio's `twilio` CLI. |
| `ket` | German *Kette* (warp) | minor clash | PyPI `ket` is a Bitbucket CLI, AUR `ket-git` is an exposure tracker, and ket is a term in Dirac notation. |
| `trm` | thrum (warp waste ends) | minor clash | crates.io `trm` is a "temporary rm" CLI. |
| `krk` | Norwegian *krokbragd* weave | minor clash | crates.io `krk` CLI helper (3k downloads); Krk is a Croatian island. |
| `tvl` | twill | minor clash | [Max-Fu/tvl](https://github.com/Max-Fu/tvl) (103 stars); TVL means "total value locked" in crypto. |
| `vft` | veft | minor clash | [jtomori/vft](https://github.com/jtomori/vft) VFX fractal toolkit (157 stars). |
| `varp` | Norwegian and Swedish *varp* (warp) | minor clash | [tech-beeline/varp](https://github.com/tech-beeline/varp) is a VS Code C4 extension (40 stars); PyPI `varp` exists. |
| `vaev` | Danish *væv* (loom) | minor clash | [skift-org/vaev](https://github.com/skift-org/vaev) is an HTML/CSS engine (309 stars). |
| `wvr` | weaver | minor clash | npm `wvr` is a CLI for wvr.io; [marchingband/wvr](https://github.com/marchingband/wvr) (98 stars). |
| `skt` | Norwegian *skyttel* (shuttle) | minor clash | crates.io `skt` Kubernetes template CLI; npm `skt`; "skt" is a common abbreviation for Sanskrit. |
| `shu` | shuttle | minor clash | [dooshu/shu](https://github.com/dooshu/shu) (415 stars); crates.io and PyPI `shu`. |
| `thr` | thread | minor clash | Generic abbreviation; npm and PyPI `thr` packages. |
| `ull` | Norwegian *ull* (wool) | minor clash | crates.io `ull` emulator core; a textile word, not a weaving one. |
| `tela` | Latin *tela* (loom, web) | minor clash | Tela icon theme; crates.io `tela` web framework; GitHub repositories of that name with 70 to 86 stars. |
| `hata` | Japanese *hata* (loom) | minor clash | Turkish *hata* means "error"; [HuyaneMatsu/hata](https://github.com/HuyaneMatsu/hata) Discord library (187 stars); npm `hata`. |
| `shd` | shed | minor clash | [alttch/shd](https://github.com/alttch/shd) is a disk-list CLI in AUR and on crates.io (50 stars). |
| `wrp` | warp | minor clash | [tenox7/wrp](https://github.com/tenox7/wrp) Web Rendering Proxy (1.3k stars). |
| `drn` | darn | minor clash | [fyu/drn](https://github.com/fyu/drn) dilated residual networks (1.1k stars). |
| `twl` | twill | minor clash | [openlibrecommunity/twl](https://github.com/openlibrecommunity/twl) (409 stars); npm `twl` Tailwind helper; TWL is a Scrabble word list. |
| `dby` | dobby | minor clash | [ulfox/dby](https://github.com/ulfox/dby) YAML DB (64 stars); it reads poorly. |
| `hld` | heald (heddle) | minor clash | crates.io `hld` hard-link deduplicator CLI; HLD means "high-level design". |
| `trd` | treadle | minor clash | crates.io and npm `trd`; [louieworth/trd](https://github.com/louieworth/trd) (139 stars). |
| `lmd` | LooMyarD | minor clash | [azproduction/lmd](https://github.com/azproduction/lmd) JS module loader (445 stars); LMD is Linux Malware Detect. |
| `lmr` | LooMyaRd | minor clash | GitHub repositories of that name with up to 50 stars; no weaving tie. |
| `lmx` | loom plus x | minor clash | Several small repositories; LMX is a market-data protocol name. |
| `sked`, `skd` | Swedish *sked* (reed) | minor clash | "sked" is slang for schedule; crates.io and npm `sked`; [brjathu/SKD](https://github.com/brjathu/SKD) (102 stars). |
| `solv` | Swedish *solv* (heddles) | minor clash | AUR `solv` is a Visual Studio solution validator; libsolv's `solv` tools exist. |
| `spol` | Norwegian *spole* (bobbin) | minor clash | AUR `spol` exists; PyPI `spol`. |
| `brik` | Norwegian *brikkevev* (tablet weaving) | minor clash | `brik` is CriscoStack's site-management CLI on PyPI; crates.io `brik`. |
| `garn` | Norwegian *garn* (yarn) | rejected | [garnix-io/garn](https://github.com/garnix-io/garn) is a `garn` build tool CLI (344 stars). |
| `rok` | Norwegian *rokk* (spinning wheel) | rejected | Two Rust CLIs named `rok` ([docs.rs rok-cli](https://docs.rs/rok-cli)). |
| `vvr`, `vvx`, `wvz`, `vevd`, `skft`, `hevl` | assorted vowel-dropped forms of *vev*, *weave*, *skaft*, *hovel* | clear but poor | No clash was found, but each is hard to read or remember. |
| `heald`, `dreil` | heald (heddle), Norwegian *dreiel* (a twill damask) | rejected | Five letters; no clash found, but too long. |
| `vevx` | *vev* plus x | rejected | The x suffix echoes `lyx` and adds nothing; not fully checked. |
| `hvl` | *hovel* | rejected | HVL is Høgskulen på Vestlandet, a Norwegian university. |
| `dnt` | dent (a reed slot) | rejected | DNT is Den Norske Turistforening, and "Do Not Track". |
| `rad` | raddle | rejected | Radicle's CLI is `rad`; Homebrew formula `rad`; [chaitin/rad](https://github.com/chaitin/rad) (1.5k stars). |
| `red` | reed | rejected | `/usr/bin/red` (restricted ed) is on this machine; Red is a programming language. |
| `bob` | bobbin | rejected | Homebrew formula `bob` (Neovim version manager). |
| `bat` | batten | rejected | `bat` is a very popular `cat` clone; not tool-checked, a known clash. |
| `btn` | batten | rejected | It reads as "button". |
| `crl` | creel | rejected | `.crl` is the certificate revocation list extension. |
| `paw` | pawl | rejected | Ubuntu package `paw` (CERN Physics Analysis Workstation). |
| `bom` | Norwegian *bom* (beam) | rejected | BOM means bill of materials and byte order mark; not tool-checked, rejected on meaning. |
| `sol` | Swedish *solv* | rejected | Homebrew cask `sol`, FreeBSD package; Solana and Solidity usage. |
| `ori` | Japanese *ori* (weaving) | rejected | Ubuntu and pkgsrc package `ori`. |
| `hst` | Greek *histos* (loom) | rejected | Chocolatey package `hst`. |
| `pik` | pick (a weft pass) | rejected | Packaged in Arch, Nix and Chocolatey; vulgar in Danish and Norwegian. |
| `jaq` | jacquard | rejected | `jaq` is a widely packaged jq clone. |
| `ply` | ply | rejected | PLY (Python Lex-Yacc), the `ply` tracer and the `.ply` 3D format. |
| `nap` | nap of cloth | rejected | Homebrew formula `nap` (Charm snippet manager). |
| `inn` | Norwegian *innslag* (weft) | rejected | Debian package `inn` (InterNetNews server). |
| `nis` | Finnish *niisi* (heddle) | rejected | Debian package `nis` (Network Information Service). |
| `wev` | weave | rejected | Debian package `wev` (Wayland event debugger). |
| `wef` | weft | rejected | [D3Ext/WEF](https://github.com/D3Ext/WEF) Wi-Fi exploitation framework (3.2k stars); World Economic Forum. |
| `wvx` | weave | rejected | `.wvx` is a Windows Media redirector extension. |
| `rnn` | Norwegian *renning* (warp) | rejected | Recurrent neural network. |
| `yrn` | yarn | rejected | It reads as a typo of the `yarn` package manager. |
| `ell` | ell (a cloth measure) | rejected | [MadcowD/ell](https://github.com/MadcowD/ell) LLM prompt library (5.8k stars), widely packaged. |
| `vad` | Norwegian *vadmål* cloth | rejected | VAD means voice activity detection ([ricky0123/vad](https://github.com/ricky0123/vad), 2k stars); Swedish *vad* means "what". |
| `sley` | to sley a reed | rejected | [indaco/sley](https://github.com/indaco/sley) is a semver CLI in AUR. |
| `slv` | selvedge | rejected | [ValidatorsDAO/slv](https://github.com/ValidatorsDAO/slv) is an AI agent kit for Solana (96 stars). |
| `selv` | selvedge | rejected | Norwegian *selv* means "self"; not tool-checked. |
| `wv` | weave | rejected | `.wv` is the WavPack extension; not tool-checked. |
| `tex` | textile | rejected | TeX; not tool-checked, a known clash. |
| `loo` | loom | rejected | British slang for toilet; not tool-checked. |
| `ly`, `loy`, `lyd`, `*lyx*` | | rejected | Excluded by the operator. |

## How the checks were made

Every candidate got the local check: `command -v <name>` and `apt-cache search --names-only '^<name>$'` on this Ubuntu machine.
Every candidate also got a GitHub check through `gh search repos <name> --match name --sort stars`, filtered to exact repository names.
GitHub's name search is fuzzy and ranks by stars, so for short common strings an exact-name repository can fall outside the first 100 results; "no repository" on GitHub is therefore weaker evidence than "no package".

The shortlist and most minor-clash names were also checked, through their public JSON APIs, against:

- the Repology project API, which covers Debian, Ubuntu, Arch, AUR, Alpine, Nix, FreeBSD, Homebrew and others;
- the Homebrew formula and cask APIs (formulae.brew.sh);
- the AUR RPC, searching for packages named `<name>` or `<name>-*`;
- crates.io, npm and PyPI;
- Scoop's Main and Extras buckets, and the Chocolatey community feed;
- pkg.go.dev search, filtered to module paths ending in `/<name>` (scraped from HTML, so a module could be missed);
- fileinfo.com, for file extensions.

Most shortlist names also got a WebSearch for "<name> command line tool", "<name> cli github" or "<name> software"; the query budget did not cover all three phrasings for every name.

Known gaps:

- winget is checked only weakly.
  GitHub code search over `microsoft/winget-pkgs` returned 0 even for known monikers such as `jq`, so it was useless; the unofficial winget.run API found no exact match for the shortlist, but that index may be stale.
- Chocolatey was checked only through the community feed, and Scoop only through the Main and Extras buckets.
- The checks for unfortunate meanings in English and Norwegian come from my own knowledge plus the search results, with no native-speaker review; Danish, Swedish, Finnish and Turkish meanings were noted where I knew them, but not systematically.
- Trademarks were not searched.
