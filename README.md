<!-- <p align="center">
  <img src="DBDReshadeManager/Images/App/logo.png" width="128" alt="DBD Reshade Manager logo">
</p> -->

<h1 align="center">DBD Reshade Manager</h1>

<p align="center">
  Reads which map Dead by Daylight just loaded and switches your ReShade preset to match — with a map overlay and a killer-side hook tracker on top.
</p>

<p align="center">
  <img alt="Windows 10 or newer" src="https://img.shields.io/badge/Windows-10%20%2F%2011-0078D4?logo=windows&logoColor=white">
  <img alt=".NET Framework 4.8" src="https://img.shields.io/badge/.NET%20Framework-4.8-512BD4?logo=dotnet&logoColor=white">
  <img alt="Built with GitHub Actions" src="https://img.shields.io/badge/builds-GitHub%20Actions-2088FF?logo=githubactions&logoColor=white">
  <img alt="Apache-2.0" src="https://img.shields.io/badge/license-Apache--2.0-green">
</p>

---

## What it does

Every DBD map has its own lighting, and a ReShade preset that looks great on Ormond makes Léry's unplayable. DBD Reshade Manager watches the loading screen, reads the map name with on-device OCR, and copies the preset you've assigned to that map into ReShade's slot. Press ReShade's reload key and you're done. When the match ends and you're back in a lobby, it puts your base preset back.

It never touches the game process. It takes a screenshot of a small strip of your monitor, runs Tesseract on it, and writes `.ini` files inside your own ReShade folder. That's the whole trick.

## Features

**ReShade switching**
- 35 presets come with the app, one per realm plus alternatives. On the first run it offers to copy them into your ReShade folder and assign them to maps, so switching works without setting anything up; presets you already have with the same name are kept unless you say otherwise, and any realm you've already assigned is left alone. *Install included presets* on the ReShade tab does the same thing later.
- Assign a preset per realm, then expand any entry (+) to give a single map or map variant its own preset that overrides it. Mappings are stored by name, so adding presets or new maps never scrambles them.
- One generated "overlay" preset is what ReShade points at; the app rewrites it per map.
- **Sharpening on every preset**: a switch on the ReShade tab adds NVIDIA's sharpening (`MartysMods_NvidiaSharpen`) to whatever preset is applied. Most presets never enable it and it costs little, so this is a cheap lift for a washed-out one. Your own `.ini` files are never edited — it's added to the copy written into the overlay slot, so turning it off just stops adding it. Presets that already run it keep their own settings, and it needs `MartysMods_NVSHARPEN.fx` in your shaders (ReShade skips it silently if it isn't there).
- A base preset is restored when you reach a lobby, when the game closes, or on a hotkey (`Ctrl+B`). Generating the overlay preset also writes `0_no_effects.ini` (every effect off) next to your presets, and that is the base preset unless you pick one of your own on the ReShade tab.
- Shows the active preset on the ReShade tab, in the tray tooltip, and as a short on-screen toast that names the reload key you bound in ReShade.

**Map detection**
- Auto mode reads the loading screen; a hotkey (`Ctrl+R`) reads the map name from the Esc menu for a manual check.
- Recognition runs a fixed-threshold sweep and then adaptive (Otsu) passes, so it still works with a dark, tinted or washed-out preset already active.
- Lobby detection reads the PLAY / READY button and the score screen's CONTINUE button, in the game's language or words you add.
- Recognition tolerates a misread letter or two: the closest map name wins when it's unambiguous.
- Works with most DBD display languages; language packs are fetched from the official [tesseract-ocr/tessdata](https://github.com/tesseract-ocr/tessdata) repository on demand.

**Overlays**
- Map overlay with configurable position, size and opacity; cycle map variants with `Shift+[` / `Shift+]`. Replace any map picture with your own PNG in the Maps folder (*Export built-in images* shows the names).
- Match timer overlay and an end-of-match toast (map, duration, hooks); every match is appended to `matches.csv` next to your settings.
- **Match stats**: the saved scoreboards are read into `matches-stats.csv`, one line per match with your own numbers — role, character, outcome and bloodpoints from your row, plus how the board stood when it was captured: how many were dead (skull), escaped (running figure in a doorway) and still in the trial (standing silhouette), and the Match stats tab shows matches by role, escape rate, kill rate, average and best bloodpoints, most-played characters, your current streak and the recent matches. Only the full scoreboard is saved and read (the bloodpoints card never counted and is ignored), so page to it after each match. Your row is found by account name (read from the lobby's top-right corner and shown on the Stats tab, where you can correct it), with the highlighted row as the fallback. 2v8 and 1v4 are kept apart (the tab opens on the mode you played last, 1v4 before anything is on file). Both the 2v8 and the 1v4 board are read (the 1v4 board has no mode label, so it is recognised by its rows). *Re-scan* reads a folder of older screenshots. The file is tamper-evident: every line is signed and chained to the one before, each scoreboard's hash is stored with it, matches are marked captured (taken by the app at the end of a match it watched) or scanned (read from files), and impossible numbers are flagged — a line edited outside the app, or whose screenshot changed (*Check screenshots*), is listed but not counted. This stops a text editor, not a programmer with the source; the totals are only as honest as the machine they were made on.
- **Score-screen screenshots**: when the end-of-trial CONTINUE screen is read, the app keeps watching the screen and saves the scoreboard the first time it shows — as a PNG of the whole game monitor, once the score count-up has finished — in a folder you choose (default `%AppData%\DBD Reshade Manager\Scoreboards`), named by time and map. Stat sites that watch the score screen themselves give up once a preset changes the colours; this keeps the frames so nothing is lost. The switch lives on the Match stats tab.
- Killer overlay: hook counter per survivor and a post-unhook endurance timer, as a HUD overlay or a side panel. Works at any monitor resolution and with a preset active, and resets itself when you're back in a lobby, when the next match loads, or when the game closes. Hook totals ("7 / 12") above the counter. Optional secondary timer marks after the endurance window (off by default; see the tab for why). The loading screen's "Playing as a killer in game mode: 2v8" line — or a 2v8 survivor's class line, "The Escapist class" — is read before each match, and the 2v8 lobby roster before that (survivors never see the killers' names there, so a killer name in the roster means you're a killer), so the overlay switches between 2v8 and 1v4 by itself (a switch on the Killer overlay tab turns that off) and knows your role. In 2v8 the survivor list is drawn differently for a spectator, the killer and a survivor; the role picks the list up front, all three are still read every tick, and the overlay moves its rows to whichever is on screen (unless you've dragged it somewhere yourself). A hook icon that comes back within seconds of going away is treated as the same hook, not a second one. The overlay's own timer and count sit clear of the portraits, because the app captures its overlay along with the HUD and a badge over the icon spoils the match — if you've dragged the overlay onto the list, *Reset* on the Position card puts it back.
- Survivor overlay: a team hook strip — one big colour-coded cell per survivor, read from the same HUD icons — and map pins you drop on the map overlay (main building, shack, hatch spawns…) with labels and colours, remembered per map and variant and exportable as a file to share.
- **Streamproof** toggle: overlays stay on your monitor but are left out of OBS / Discord display capture, Game Bar and screenshots (Windows 10 2004+).
- Overlays can hide automatically whenever the game isn't the focused window.
- Switches that depend on another one (the match timer needs Auto mode, hook totals need the hook counter…) offer to turn the other one on, with a "don't ask again" option.

**Quality of life**
- Tray icon, minimize to tray, start minimized, start with Windows (per-user, opt-in).
- **Updates**: the About tab checks for a newer build at startup and every six hours (switchable), downloads it, verifies its checksum and installs it the next time you start the app — or right away with *Restart and install*. Builds come from a public releases repository that holds only zips and checksums, so no token is needed. A fine-grained token with *Contents: read* can be pasted on the About tab as a fallback for reading the private source repository's own releases; it is stored encrypted for your Windows account, never in the build.
- **Keys** (optional, off in the source): a build can require a signed key naming who it was issued to, when it runs out and — if you want — which PC it belongs to. See *Keys* under Building.
- One settings file at `%AppData%\DBD Reshade Manager\settings.ini` that survives updates and re-downloads, with *Save copy…* / *Load…* to back it up or move it to another PC.
- Diagnostics tab: *Check my setup* runs through monitor, game window, OCR language, ReShade folder and presets, capture and what every region reads right now; *Show capture regions* draws them over the game; plus the Esc menu / loading screen / lobby test buttons and the log.
- Single instance — launching it again just brings the running one to the front.

## Getting started

### 1. Install

Two ways, both from the [latest release](https://github.com/q32/dbd-reshade-manager-releases/releases/latest). Each file ships a `.sha256` beside it if you want to verify the download.

**Installer** — `dbd-reshade-manager.msi`. Installs for your user only, so there's no UAC prompt and no admin rights needed. Adds a Start Menu entry and an uninstall entry, and updates replace the install properly so the version in Add/Remove Programs stays right.

**Zip** — `dbd-reshade-manager.zip`. Unpack it anywhere and run `DBDReshadeManager.exe`. Nothing is registered with Windows; updates copy the new build over the folder in place. Fine if you'd rather keep it portable.

Either way, Windows SmartScreen will warn about an unsigned app the first time — *More info* → *Run anyway*. Some antivirus does the same; the app reads the screen, draws overlays and updates itself, which looks unusual to a behavioural scanner even though it's ordinary here.

Settings, stats and screenshots live in `%AppData%\DBD Reshade Manager` in both cases, so switching between them, or uninstalling, never takes your setup with it.

### 2. Set DBD to Windowed Fullscreen

Overlays can only sit on top of the game in this mode: DBD *Options → Graphics → Full screen mode → Windowed Fullscreen*.

### 3. Point it at ReShade

On the **ReShade** tab:

1. **Preset folder** — pick the folder where your ReShade `.ini` presets live (usually next to `DeadByDaylight-Win64-Shipping.exe`).
2. **Overlay preset** — give it a name and click *Generate*. In ReShade, select this preset and leave it selected; the app writes the current map's preset into it.
3. **Assign presets to maps** — pick a preset per realm; expand an entry with + to override it for one map or variant (variants apply when you cycle them with `Shift+[` / `Shift+]`). Between matches the app reverts to the *base* preset: `0_no_effects` by default, or any preset you choose under *Base preset*.
4. In ReShade's *Settings* tab, bind an **Effect reload key**. ReShade only re-reads a preset when told to, so press that key once the toast appears. Tell the app which key it is so the toast can remind you.

Then turn on **Auto mode** on the Map overlay tab and play.

### Default hotkeys

| Action | Default |
| --- | --- |
| Read the map from the Esc menu | `Ctrl+R` |
| Apply the base preset | `Ctrl+B` |
| Previous / next map variant | `Shift+[` / `Shift+]` |
| Hide / show all overlays | `Ctrl+H` |
| Auto mode on / off | `Ctrl+K` |
| Save diagnostic screenshots | `Ctrl+M` |

All of them can be changed on the Hotkeys tab. Pick something other than the key you gave ReShade.

## Settings and backups

Everything you configure is written to `%AppData%\DBD Reshade Manager\settings.ini` — a plain `Key=Value` file you can read, edit or copy. Because it lives outside the app folder, you can delete the app, unpack a new version anywhere, and your setup is still there. The Settings tab has *Open folder*, *Save copy…* and *Load…* (loading restarts the app). The first launch of this version imports settings from an older build automatically.

## Troubleshooting

- **The preset doesn't change in game.** ReShade needs its reload key pressed; check the key you bound in ReShade's Settings tab matches what the toast shows.
- **No map detected.** Open Diagnostics and click *Check my setup* first. Then confirm the right monitor is selected, then use *Test: loading screen* while the map name is on screen and look at the captured strip and OCR text. The game language on the Settings tab must match DBD's.
- **Hooks aren't counted.** Use *Save diagnostic screenshots* (`Ctrl+M`) during a match and check `survivors_N.png` (`survivors_2v8_N.png`, `survivors_2v8_killer_N.png` and `survivors_2v8_survivor_N.png` in 2v8) in the screenshots folder; the *Icon match threshold* slider on the Killer overlay tab adjusts sensitivity.
- **Something crashed.** Diagnostics → *Open log*. The log rolls over at 2 MB so it never grows unbounded.

## Is this safe to use?

The app reads pixels from your screen and writes files in your ReShade folder. It does not read game memory, inject anything, hook the game, or send input to it. Its network access is limited to GitHub: language packs from the tesseract-ocr project when you pick a new language, and its own releases when it checks for updates.

Updates come only from the two repositories compiled into the build — the public releases repository, then the private source one — and never from a URL taken from a server's answer. A new release is downloaded over HTTPS, checked against the `.sha256` published beside it, and unpacked into `%AppData%\DBD Reshade Manager\Update` — entries that would land outside that folder are refused. Nothing is installed while the app runs: at the next start the staged build is run once with `--apply-update` so it can copy itself over the program folder and start it again. No shell, no script, and the only downloaded code that ever runs is the new build itself. If any step fails the old build is left exactly as it was.

That said, ReShade itself is "use at your own risk" as far as Behaviour is concerned, and Dead by Daylight has been trialling Denuvo Anti-Cheat on Steam since September 2026. An overlay that shows information the game's own HUD already shows (hook states) is not something the anti-cheat can see, but whether it counts as an "unfair advantage" is Behaviour's call, not this project's. Use judgement.

## Building

Open `DBDReshadeManager.sln` in Visual Studio 2022 with the *.NET desktop development* workload and build Release, or:

```
msbuild DBDReshadeManager.sln /p:Configuration=Release
```

The output is `DBDReshadeManager/bin/Release/net48/DBDReshadeManager.exe`. Pushing a tag like `v1.5.0` makes GitHub Actions build the zip, compute its SHA-256 and attach both to a release. All third-party actions in the workflow are pinned to commit SHAs.

### Keys

Builds can be locked to people you've given a key to. It's off in the source: `LicenceManager.PublicKey` is empty, and an empty public key means the app never asks for anything. To turn it on:

1. `php artisan licence:signing-key` in the licence server, on your own machine. It prints two strings: `LICENCE_PRIVATE_KEY` for your **local** `.env` only, and `LICENCE_PUBLIC_KEY` for the deployed environment.
2. Paste that public half into `LicenceManager.PublicKey`, rebuild, and release. That constant is what turns key checking on.
3. Per friend: `php artisan licence:issue "Bob" --days=365 --seats=2` — optionally `--machine=XXXX-XXXX-XXXX` to tie the key to one PC (they read that ID off the app's About tab or the window it shows at startup). Send them the line it prints; they paste it into that window.
4. To withdraw one: `php artisan licence:revoke <id>` (and `--undo` to put it back). `php artisan licence:list` shows who has what and which PCs are using it.

Keys are signed on your machine, never on the server: the deployed side only ever holds the public half, so breaking into it gets someone the list of who has a key and nothing else. The app you hand out cannot issue keys either — there is no issuer in it.

Keys are ECDSA P-256 signatures over "who, until when, which PC", so the build can tell a real key from an invented one but can't make keys itself — lifting the public key out of the exe gets you nothing. Without the licence server, withdrawing one means putting its id on its own line in `revoked.txt` at the root of the public releases repository; apps pick it up within a day and refuse it at the next start (never mid-match), keeping the last list they fetched if they can't reach GitHub. With the server, `licence:revoke` does it at their next launch and `revoked.txt` is the belt-and-braces version.

A key, and the server's permission that goes with it, are stored in `settings.ini` alongside every other setting — in `%AppData%`, outside the app folder — so an update never asks anyone to enter theirs again. The updater only ever writes to the program folder.

This keeps the circle to people you handed a key to. It cannot stop someone who edits the program itself — nothing that runs on someone else's PC can — so treat it as a lock on the door, not a vault.

#### Limiting a key to a few PCs

Signatures alone can't count how many PCs a key is on, and withdrawing one means publishing a new `revoked.txt`. If you want either, `server/` is a small Laravel app that does it — deploy it, fill in `ProjectInfo.LicenceServerUrl` and `LicenceManager.EntitlementPublicKey`, and each key gets a seat count you can change and a revoke you can do from a command line. Leave both empty and none of it exists: the app never calls out and behaves exactly as above. The app asks the server at every start, before it opens and before it installs an update, so a key you withdraw stops working the next time that PC opens the app. If the server can't be reached it runs on the signed permission it was last given — two days by default — so being offline, or the server being down, doesn't strand anyone mid-week. `server/README.md` has the setup and the day-to-day commands.

### Publishing a release

The app updates itself from a **public** repository that holds only the built zips — never the source — so nobody needs a token to stay up to date. Set that up once:

1. Create the public repository `q32/dbd-reshade-manager-releases` **with at least one commit** — `gh repo create q32/dbd-reshade-manager-releases --public --add-readme`. A release has to tag something, so publishing into a repository with no commits fails.
2. Make a fine-grained token with **Contents: write** on that repository *only*, and add it to this repository as the secret `RELEASES_TOKEN` (Settings → Secrets and variables → Actions). Make the repository first: a token can only be scoped to repositories that already exist, so one made earlier has to be edited to include it.
3. Cut the release: `.\scripts\release.ps1 -Bump minor` (or `-Version 2.0.0`).

The workflow then attaches the zip and its checksum to a release here *and* to the same tag in the public repository, and every installed copy picks it up within six hours, or at its next start. Until the secret exists the public step is skipped with a warning and the private release is still made, which the app can read with a personal token entered on its About tab.

`scripts/release.ps1` does the bookkeeping that is easy to get wrong: it bumps `AssemblyVersion` and `FileVersion` in `DBDReshadeManager/DBDReshadeManager.csproj`, commits, tags and pushes, then watches the build and checks the release actually reached the public repository. The app compares the tag against the version compiled into the running build, so the two must agree — a tag that isn't higher is ignored, and a tag higher than the build it contains installs and then rolls itself back, forever. The script refuses both, along with a dirty tree, a tag that already exists and a branch that's behind the remote. `-DryRun` shows what it would do; `-Status` just prints the current and last-published versions. Tagging by hand still works if you'd rather: `git tag v1.5.0 && git push origin v1.5.0`.

## Licence

Licensed under the [Apache License 2.0](LICENSE). Derived from [H4RDC0RN/dbd-overlay](https://github.com/H4RDC0RN/dbd-overlay), also Apache-2.0 — [NOTICE](NOTICE) records what changed. Not affiliated with Behaviour Interactive or ReShade.
