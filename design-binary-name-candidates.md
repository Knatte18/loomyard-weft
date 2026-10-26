# Candidate names for the lyx binary: three letters, clash-checked (research, 2026-10-26)

`binary-name-candidates` · Note · research

# Binary name research, round 2: three letters that sound like `lyx`

Researched 2026-10-26.
The clash rule is narrower than in round 1: only an installable command of the same name in apt (Debian or Ubuntu), Homebrew, winget, Scoop, Chocolatey or AUR counts.
Library packages on npm, PyPI and crates.io, and small GitHub repositories, are ignored, though a few are mentioned for context.
"AUR" is marked on every clash that comes only from AUR, the user-uploaded and uncurated Arch repository.
92 names were checked: the 25 `ly?` names, 14 earlier three-letter names, and 53 new names in the l/r/m/n pattern.

## Ranked shortlist

### 1. `lys`

- Sound: one syllable, "leess" in Norwegian, "liss" or "lease" in English; it opens on soft l, has y and closes on soft s.
- Meaning: Norwegian, Danish and Swedish for "light", "candle" and "bright"; it has no English meaning.
- Weaving link: none; as a LoomYard abbreviation it can read "LoomYard System".
- Command clashes: none in any repository checked, AUR included.
  The nearest names on the web are `lyscripts` (PyPI) and the `ly` LilyPond tool, neither of which is named `lys`.

### 2. `lyd`

- Sound: "lyd" with a soft d, almost "lee" in Norwegian, where the d is nearly silent; "lid" in English.
- Meaning: Norwegian and Danish for "sound"; it has no English meaning.
- Weaving link: none, but it is the cleanest abbreviation of LoomYarD.
- Command clashes: none in any repository checked; it is not in the winget index at all.
  The nearest name on the web is `lydia-cli` (PyPI), a local AI coding agent, which is named differently.
- Note: the word is very common in Norwegian, so it is weak as a search term.

### 3. `lyn`

- Sound: "lyn" in Norwegian, "lin" or "Lynn" in English; one soft syllable.
- Meaning: Norwegian and Danish for "lightning", the Welsh *llyn* "lake", and the English given name Lynn; the lightning sense suits a fast tool.
- Weaving link: none.
- Command clashes: AUR only: [`lyn-git`](https://aur.archlinux.org/packages/lyn-git), a weather CLI using the Norwegian Meteorological Institute ([src.timharek.no/tim/lyn](https://src.timharek.no/tim/lyn)), with 0 votes.
  The Homebrew cask [`lyn`](https://formulae.brew.sh/cask/lyn) is a macOS media viewer app; it installs `Lyn.app` and no command.

### 4. `lom`

- Sound: "lomm" in Norwegian, "lom" in English; one soft syllable with a round vowel.
- Meaning: Norwegian for the loon bird, and a Norwegian municipality; in Slavic languages it means "break" or "scrap", which is harmless here.
- Weaving link: the strongest on this list, since it is *loom* with one o.
- Command clashes: none in apt, Homebrew, winget, Scoop, Chocolatey or AUR.
  It falls outside the rule but is worth knowing: [LackOfMorals/lom](https://github.com/LackOfMorals/lom) is a 0-star Go `lom` binary for Neo4j, shipped as a PyPI package.

### 5. `lur`

- Sound: "lür" in Norwegian, "lure" in English; one soft syllable.
- Meaning: in Norwegian, *lur* means "clever, sly", "a nap", and the long wooden lur horn; the English sound "lure" is neutral.
- Weaving link: none.
- Command clashes: none.
  Debian's `lur-command` package (a Logitech receiver tool) installs `/usr/bin/lur-command`, not `lur` ([file list](https://packages.debian.org/sid/amd64/lur-command/filelist)).

### 6. `lyr`

- Sound: "lyr" in Norwegian, "leer" or "lure" in English.
- Meaning: the Norwegian name of the pollock fish; it echoes "lyre" and "lyric".
- Weaving link: none; as an abbreviation it reads "LoomYaRd".
- Command clashes: AUR only, two of them: [`lyr-bin`](https://aur.archlinux.org/packages/lyr-bin), a Wayland/Windows/macOS status bar ([kryptic-sh/lyr](https://github.com/kryptic-sh/lyr)), and [`lyr-git`](https://aur.archlinux.org/packages/lyr-git), a genius.com lyrics scraper; both have 0 votes.
  A crates.io `lyr` lyrics tool exists too, which the rule ignores.

### 7. `lun`

- Sound: "lün" in Norwegian, "loon" in English.
- Meaning: Norwegian for "cosy, warm, sheltered", and for warm, dry humour; the English sound "loon" (a crazy person) is mild.
- Weaving link: none.
- Command clashes: none.

### 8. `nyl`

- Sound: "nyl" in Norwegian, "nil" or "nile" in English; it opens on soft n.
- Meaning: none in Norwegian; in English it is the start of "nylon".
- Weaving link: nylon, a textile fibre.
- Command clashes: none; it appears in the winget index only inside a longer string, not as a value of its own.

### 9. `lyt`

- Sound: "lyt" in Norwegian, "lit" in English.
- Meaning: Danish *lyt*, Norwegian *lytt*, "listen"; in English it reads as "lit", which is mildly positive slang.
- Weaving link: none.
- Command clashes: none.

### 10. `lod`

- Sound: "lodd" in Norwegian, "lod" in English.
- Meaning: Norwegian *lodd* is a weight or plumb, and a lottery ticket; in English, LOD is "level of detail".
- Weaving link: weak; loom weights, *kljåstein*, hold the warp taut on the upright Norwegian loom.
- Command clashes: none.

### 11. `lym`

- Sound: "lym" in Norwegian, "lim" in English.
- Meaning: none of its own; it echoes "lymph" and the Norwegian *lim* "glue".
- Weaving link: none; as an abbreviation it reads "LoomYard Manager".
- Command clashes: none.

### 12. `rya`

- Sound: "RY-ah"; two syllables in both languages, so a step away from the pattern.
- Meaning: the Scandinavian rya rug; it has no other meaning.
- Weaving link: strong; rya is a knotted-pile weave.
- Command clashes: none (Apache Rya and the PyPI `rya` package are not commands in any repository checked).

### 13. `liv`

- Sound: "leev" in Norwegian, "liv" in English.
- Meaning: Norwegian for "life" and "waist"; also a given name.
- Weaving link: none.
- Command clashes: none.

### 14. `myr`

- Sound: "myr" in Norwegian, "meer" or "myrrh" in English; it opens on soft m.
- Meaning: Norwegian for "bog, marsh", which is slightly bleak; in English it echoes "myrrh".
- Weaving link: none.
- Command clashes: none.

### 15. `ull`

- Sound: "ull" in Norwegian, "ool" or "ull" in English; it opens on a vowel, so it is off the pattern.
- Meaning: Norwegian for "wool".
- Weaving link: a textile word.
- Command clashes: none; it appears in the winget index only inside a longer string.

## The `ly?` names

`lyx` is excluded.
"No clash" means no installable command in apt, Homebrew, winget, Scoop, Chocolatey or AUR.

| Name | How it sounds | Meaning in English and Norwegian | As a LoomYard abbreviation | Command clashes |
| --- | --- | --- | --- | --- |
| `lya` | "LEE-ah", two syllables | none; Lyα is the astronomers' Lyman-alpha line | LoomYard Agent | none (in winget only as a publisher name, not a command) |
| `lyb` | "lib" | none | LoomYard Build | none |
| `lyc` | "lick" or "lyk" | none; it echoes "lycée"; the English reading "lick" is a little awkward | LoomYard CLI or Core | none (in winget only as a publisher name) |
| `lyd` | "lee" or "lid" | Norwegian "sound" | LoomYarD | none |
| `lye` | "lie" | English lye, caustic soda, and a homophone of "lie"; awkward for a tool | LoomYard Engine | none |
| `lyf` | "liff" or "life" | none | LoomYard Flow | none |
| `lyg` | "lyg" | Norwegian "lie!" (from *lyve*) | | none; rejected for its meaning |
| `lyh` | hard to say | none | | none; rejected as unpronounceable |
| `lyi` | hard to say | none | | none; rejected as unpronounceable |
| `lyj` | hard to say | none | | none; rejected as unpronounceable |
| `lyk` | "lick" or "lük" | none; it echoes Norwegian *lykke* "luck" | LoomYard Kit | none |
| `lyl` | "lil" | none; it reads as "lil" | | none; weak sound |
| `lym` | "lim" | none; it echoes "lymph" and Norwegian *lim* "glue" | LoomYard Manager | none |
| `lyn` | "lin" or "Lynn" | Norwegian "lightning"; the given name Lynn | | AUR only: `lyn-git` weather CLI, 0 votes; the Homebrew `lyn` cask is a GUI app with no command |
| `lyo` | "LEE-oh", two syllables | none; it echoes Lyon | | none |
| `lyp` | "lip" | none; it reads as "lip" | | none |
| `lyq` | hard to say | none | | none; rejected as awkward to say and type |
| `lyr` | "leer" or "lure" | Norwegian "pollock" (a fish) | LoomYaRd | AUR only: `lyr-bin` status bar and `lyr-git` lyrics scraper, both 0 votes |
| `lys` | "leess" or "liss" | Norwegian "light", "candle", "bright" | LoomYard System | none |
| `lyt` | "lit" | Danish "listen", Norwegian *lytt* "listen" | LoomYard Tool | none |
| `lyu` | "LEE-oo", two syllables | none | | none |
| `lyv` | "lyv" | Norwegian "lie!" (from *lyve*) | | none; rejected for its meaning |
| `lyw` | hard to say | none | | none; rejected as unpronounceable |
| `lyy` | "lee" | none | | none; it looks like a typo |
| `lyz` | "lies" | none, but it sounds like "lies" | | none; rejected for its sound |

The best two `ly?` names are `lys` and `lyd`: both have no clash at all and one clean syllable, and `lyn` comes third with only an AUR clash.

## Verdicts on every other name

| Name | Verdict | Reason |
| --- | --- | --- |
| `lon` | clear | No clash; no meaning beyond the given name Lon and the LON abbreviation for London; fits the sound but is bland. |
| `lor` | clear | No clash (in winget only as a tag, for *Legends of Runeterra*); it reads "lore". |
| `lox` | clear | No clash; English lox (smoked salmon) and LOX (liquid oxygen) are harmless; a good sound. |
| `lov` | clear | No clash; Norwegian "law" and "permission", and it echoes "love"; a good sound. |
| `lok` | AUR clash | AUR `lok`, a line-of-code counter with 0 votes ([wyhaya/lok](https://github.com/wyhaya/lok)); Norwegian *lokk* means lid. |
| `lub` | clear | No clash; it reads "lub" (lubricant, heartbeat), which is unattractive. |
| `lud` | clear | No clash; English "m'lud" and the echo of "Luddite" are an odd fit for an automation tool. |
| `lux` | clash | Homebrew formula `lux`, Scoop Main `lux`, Chocolatey `lux`, and AUR `lux`. |
| `lum` | clear | No clash (in winget only inside a longer string); no meaning. |
| `lut` | clash | Debian `tracetuner` ships `/usr/bin/lut`; LUT also means "lookup table", and Norwegian *lut* means lye. |
| `luv` | clear | No command clash: the Homebrew `luv` formula is a Lua binding library with no executable; but "luv" is cutesy slang. |
| `lin` | clash | Homebrew `linode-cli` installs a `lin` command; it is a pity, since Norwegian *lin* means flax and linen. |
| `lim` | clear | No clash; Norwegian "glue", English "limit"; it sounds fine. |
| `lix` | AUR clash | AUR `lix`, a Lemmings-like game (4 votes); LIX is also a readability index, and Lix is a Nix fork. |
| `lid` | clash | Debian `id-utils` ships `/usr/bin/lid`, Debian `libuser` ships `/usr/sbin/lid`, and Homebrew `idutils` ships `lid`. |
| `lir` | clear | No clash; no meaning, and bland. |
| `lis` | clash | Homebrew `lisette` installs a `lis` command; AUR `lis` is a solver library. |
| `ryn` | clear | No clash; it echoes Norwegian *rynke* "wrinkle". |
| `rym` | clear | No clash; Swedish *rymma* "to escape"; it sounds a bit flat. |
| `ryk` | clear | No clash; Norwegian *rykk* means "jerk"; a hard sound. |
| `rok` | unclear | It is in the winget index as a value in a moniker-like list; without sqlite I could not confirm whether it is a moniker, a command or a tag, so it is not called clear. |
| `rom` | rejected | ROM is too generic, and *Rom* is the Romani people's own name for themselves, which is best avoided; it is also a winget tag. |
| `ron` | clear | No clash; it is just the given name Ron. |
| `rov` | rejected | Norwegian *rov* means plunder or prey. |
| `rox` | clash | Debian `rox-filer` ships `/usr/bin/rox`; AUR `rox`. |
| `run` | clash | Homebrew `run`, AUR `run-git`; far too generic. |
| `rud` | clear | No clash; it reads close to "rude". |
| `rum` | AUR clash | AUR `rum`, a game library manager (1 vote); English "rum", Danish "room". |
| `rin` | clear | No clash (in winget only inside a longer string); no meaning. |
| `rid` | AUR clash | AUR `rid-git`, a Reddit image downloader (1 vote); English "rid" is neutral. |
| `rim` | clash | Chocolatey `rim` is the R installation manager ([r-lib/rim](https://github.com/r-lib/rim)). |
| `rio` | clash | apt `rio`, Homebrew cask `rio`, Scoop Extras `rio` (the Rio terminal), Chocolatey and Arch `rio`. |
| `myk` | clear | No clash; Norwegian "soft"; it reads "mick". |
| `mur` | clear | No clash; Norwegian "wall"; a dull sound. |
| `mul` | clear | No clash; it reads as "mule" or "multiply". |
| `mun` | clear | No clash; it echoes Norwegian *munn* "mouth". |
| `mox` | clash | Homebrew `mox` and AUR `mox`, a mail server. |
| `mod` | clash | Debian `mono-devel` ships `/usr/bin/mod`, as does Homebrew `mono`; too generic anyway. |
| `moy` | clear | No clash; no meaning. |
| `myn` | clear | No clash; no meaning. |
| `nyx` | clash | apt, Homebrew and Arch `nyx`, the Tor status monitor. |
| `nor` | clear | No clash; the logical NOR, and an abbreviation for Norway. |
| `nok` | clear | No clash; Norwegian "enough", and the currency code NOK; a little comic. |
| `nul` | rejected | NUL is a reserved device name on Windows, so files and binaries with this name misbehave there. |
| `nub` | clash | Homebrew `nub`, AUR `nub`, and Chocolatey `nub`. |
| `nyr` | clear | No clash (in winget only inside a longer string); no meaning. |
| `noy` | clear | No clash; it reads like "annoy". |
| `nur` | clear | No clash; German "only", Arabic "light". |
| `kud` | clear | No clash; but it does not fit the sound pattern ("kood"). |
| `vef` | clear | No clash; but the operator found it unpleasant to say. |
| `wvn` | clear | No clash; but it has no vowel and does not fit the sound pattern. |
| `lmy` | clear | No clash; but it has no vowel sound of its own. |
| `wov` | clear | No clash; off the sound pattern. |
| `hed` | AUR clash | AUR `hed-git`, a hex editor (1 vote); off the sound pattern. |
| `vov` | clear | No command clash (winget lists only "VOV" product names, and Altair VOV is not in any repository); off the sound pattern. |
| `hov` | clear | No clash; off the sound pattern; Norwegian "pagan temple". |
| `bur` | clear | No clash; off the sound pattern; Norwegian "cage". |
| `loi` | clear | No clash; French "law"; it reads as two vowels, "lwah" or "loy", and `loy` was rejected for being too close to `ly`. |
| `vev` | rejected | Ruled out by the operator. |

## How the checks were made

Every name went through the same checks, using a bash loop and `curl`:

- **Local:** `command -v` and `apt-cache search --names-only '^name$'` on this Ubuntu machine.
- **Debian:** the [packages.debian.org contents search](https://packages.debian.org/) for an exact file name in sid, keeping hits under `/bin`, `/sbin`, `/usr/bin`, `/usr/sbin` and `/usr/games`.
  This catches commands shipped by a package of a different name, such as `lid` in `id-utils`.
  The Ubuntu contents search at packages.ubuntu.com returned HTTP 500, so Ubuntu is covered only by the local apt package-name search; a command that only Ubuntu ships under another package name could be missed.
- **Homebrew:** Homebrew's own executables index ([formulae.brew.sh/api/internal/executables.txt](https://formulae.brew.sh/api/internal/executables.txt)), which maps each formula to the commands it installs, plus the formula and cask JSON APIs.
  Cask hits were then read by hand: the `lyn` cask installs only an app.
- **winget:** the official source index (`source.msix` from cdn.winget.microsoft.com, `Public/index.db`), searched byte-wise, since there is no sqlite3 on this machine.
  A name was counted as present when it appears as a whole SQLite text value.
  Which table a value is in (command, moniker, tag, publisher) was inferred from the neighbouring values, so that column is less certain than the others; that is why `rok` is marked unclear.
- **Scoop:** whether a manifest of that name exists in the official Main, Extras, Versions, Java and Games buckets; third-party buckets and manifests whose `bin` differs from their name were not searched.
- **Chocolatey:** the community feed `FindPackagesById`, which matches package IDs only, not the commands a package installs.
- **Arch and AUR:** the Arch package search by name, and the AUR RPC by name and by `provides`.
  Commands in official Arch packages of a different name are not covered, since Arch has no web file search.
- **Web:** searches for "<name> command line tool github" for `lys`, `lyd` and `lur`; they found no command of the same name.

Meanings in English and Norwegian are from my own knowledge, with no native-speaker review.


# Round 1 (any registry hit counted as a clash)


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
