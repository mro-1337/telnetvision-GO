# Telnetvision for Windows

**Stream a webcam, your screen, a movie, a music folder, IPTV or YouTube, and let anyone watch it as live ANSI art: over telnet, or in a web browser.** No BBS needed.

```
telnet your-server 2323
```

This is a Go rewrite and extension of [hmderdoc/telnetvision](https://github.com/hmderdoc/telnetvision), built for Windows and fully compatible with it. The original BBS door and relay are still included.

> Thanks to the original Telnetvision authors and contributors, to Fabrice Bellard for ffmpeg (and many, many other things), and to Rick Parrish for fTelnet, which powers the web player.

---

## How it works

```
 telnetvision.exe  ── feed (7600) ──►  tvserver.exe  ── telnet (2323) ──►  terminals
  (camera PC)                           (server)      ── web (8080) ────►  browsers
  webcam / screen /                                   ── 7601 ──────────►  door.exe (BBS)
  movies / music /
  IPTV / YouTube
```

| Program | What it does |
|---|---|
| `camera-pc\telnetvision.exe` | Reads the video through ffmpeg, shrinks it for terminals, and sends picture, sound and captions to the server. |
| `server\tvserver.exe` | Viewers connect to it by telnet, or open `http://SERVER:8080`. Each viewer gets the picture fitted to their own window, colour depth and character set. |
| `bbs-door\door.exe`, `service.exe` | The original Telnetvision BBS door and relay, for running it inside a real BBS (optional). |

The camera PC and the server can be the same computer or different ones.

## Features

**Watching**
- Works in SyncTERM, NetRunner, IcyTerm, PuTTY, Windows Terminal, the Windows telnet client, and Mac/Linux terminals.
- 24-bit, 256 or 16 colour, in CP437 or UTF-8. The picture is fitted to each viewer's window.
- **Sixel** and **JPEG XL** pictures (real pixels) for terminals that can show them. JPEG XL has three detail levels, and auto-detail drops a level when a viewer's connection lags.
- **Sound**: SyncTERM sound in telnet, full sound in the web player, with lip sync.
- A **web player** built into tvserver.exe (fTelnet). Nothing to install, and it works on phones and tablets, fullscreen included.
- Captions and subtitles: sidecar .srt/.ass files, embedded tracks, and closed captions.
- A per-viewer **S menu** (colours, speed, sound, detail), remembered for next time.
- **Playlist lightbar (L)**: a viewer watching alone can pick what plays next, either *play now* or *tag for up next*.
- **Group chat (N)**: the picture on the left and a scrolling chat on the right, like a live stream. Names get their own colour, and `/name newname` changes yours.

**Sources** (camera PC)
- Webcams and capture cards. Press D to pick one, and save it as the default.
- One screen or all of them.
- Video files and folders, with jumping around: J to type a time, arrow keys to step, and SEEK_STEP for how far.
- Music, with a visualizer.
- IPTV `.m3u` channel lists.
- YouTube channels, playlists and videos, through yt-dlp, downloaded ahead of time.
- Anything given on the command line: `telnetvision.exe X:\Movies` or a file or URL.

**For slow PCs and slow connections**
- **Re-encode**: if the camera PC can't decode a movie fast enough, ffmpeg makes a lighter temporary copy and playback resumes from the exact same spot. It can do this by itself (`REENCODE=1`), or you can press E for a 720p or half-size copy.
- Each viewer gets the frame rate their connection can take. Sound always goes first.
- Live viewer statistics in the server window (`stats`, `who`).

**Running a server**
- Settings in plain `.ini` files. Old setting names and older files are upgraded automatically.
- **Ban lists**: `ipban.txt` (can't watch) and `chatban.txt` (can watch, can't chat). Both are plain text, accept wildcards (`203.0.113.*`) and ranges (`/24`), and are reloaded live. You can also manage them with the server window commands `ban`, `unban`, `chatban`, `unchatban` and `bans`.
- Camera-PC switches for everyone: O turns the viewers' playlist on or off, W turns the group chat on or off.
- TLS on the feed with a feed token, and a viewer limit.

## Quick start

Download `Telnetvision-X.Y.Z-Win64.zip` (or `-Win32.zip` for 32-bit Windows) from **Releases** and unzip it.

**1. Start the server.** Double-click `server\tvserver.exe`. The first time, it creates `tvserver.ini` with a random feed token and shows it, along with the addresses to connect to. Allow it through the Windows firewall when asked.

**2. Start the camera.** Put `ffmpeg.exe` next to `camera-pc\telnetvision.exe`. You can get it from the [gyan.dev builds](https://www.gyan.dev/ffmpeg/builds/), or run `winget install Gyan.FFmpeg`. Then double-click `telnetvision.exe` and answer its questions:
- the server's address (`127.0.0.1` if it's the same PC)
- the feed port (`7600`)
- the token the server showed you
- what to play

**3. Watch.**
```
telnet SERVER-ADDRESS 2323
```
or open `http://SERVER-ADDRESS:8080` in a browser and click **Watch with sound**.

To let people watch from the internet, forward port 2323 (telnet) and port 8080 (web) on your router to the server PC.

For YouTube, put `yt-dlp.exe` next to `telnetvision.exe`.

## Keys

**Viewers**

| Key | |
|---|---|
| Q / X / Enter | stop watching |
| S | settings menu |
| + / - | faster / slower |
| C | captions |
| T | title bar |
| M | mute |
| L | playlist (when watching alone) |
| N | group chat |

**Camera window** (press H for the full list)

| Key | |
|---|---|
| S / A | play something else / add to the playlist |
| N / B | next / back |
| J, ← → | jump to a time / step back and forward |
| E | re-encode to a lighter copy (1 = 720p, 2 = half size) |
| L | list the playlist |
| D | pick a camera |
| O | viewers' playlist on/off |
| W | group chat on/off for everyone |
| T / X | show a message to viewers / clear it |
| R | shuffle |
| Q | quit |

The release zips include a full user guide (`README.txt`) covering every setting.

## Building from source

You need **Go 1.24 or newer**. No C compiler is needed (`CGO_ENABLED=0`), and the build works fully offline.

```bat
build.bat          :: 32-bit and 64-bit
build.bat 64       :: 64-bit only
build.bat test     :: run the tests, then build
```

On Linux or macOS, cross-compile with `./build.sh` (or `./build.sh test`). The programs land in `bin\Win32` and `bin\Win64`, laid out like the release zips. Builds are reproducible: building the same source gives byte-identical `.exe` files.

| Folder | |
|---|---|
| `producer/` | telnetvision.exe |
| `tvserver/` | tvserver.exe. The picture and sound code from the original door is in `door_*.go`, and the web player is in `web/`. |
| `door/`, `service/` | the original door and relay (the relay has one bug fix) |
| `lib/` | shadecell and termprobe, the original author's libraries |
| `third-party/` | fTelnet (full source), the jSquash licence, and the upstream Telnetvision for reference |
| `docs/` | the user guides, example settings and an ASCII ad for your BBS |

`README-SOURCE.txt` has more detail, including how to change the patched fTelnet.

### Feed protocol

The feed uses the original Telnetvision wire format (length-prefixed messages), with extensions. Programs ignore messages they don't know, so old and new versions work together.

| Type | Message | Direction |
|---|---|---|
| `0x10` / `0x11` | FRAME / AUDIO (upstream) | camera → server |
| `0x12` | FRAME_Z: zlib-packed, delta, 6-bit colour | camera → server |
| `0x13` / `0x14` | FRAME_HI / FRAME_HI_Z: sharper copy for sixel | camera → server |
| `0x15` / `0x16` | JPEG XL frame / JPEG XL frame at a detail level | camera → server |
| `0x17` / `0x18` | PLAYLIST (with names) / PLAYLIST_NOW (position only) | camera → server |
| `0x19` | group chat allowed on/off | camera → server |
| `0x20` | WELCOME: what the server understands | server → camera |
| `0x21` / `0x22` | WANT_HI / WANT_JXL (detail-level mask) | server → camera |
| `0x23` / `0x24` | WANT_PLAYLIST / PLAY_ITEM (now or up next) | server → camera |
| `0x25` | CHAT: messages for the camera window | server → camera |

## Credits and licences

- **Telnetvision** by hmderdoc ([MIT](https://github.com/hmderdoc/telnetvision)): the original door, relay and picture/sound code, plus the shadecell and termprobe libraries.
- **fTelnet** by Rick Parrish, R&M Software (AGPL-3.0): the terminal in the web player. Its full source and the small patch applied to it are in `third-party/fTelnet` and `tvserver/web/patch-ftelnet.py`.
- **jSquash** JPEG XL decoder by Jamie Sinclair and Google (Apache-2.0). It's used unchanged, and only for browsers that can't show JPEG XL themselves.
- **ffmpeg** and **yt-dlp** are run as separate programs and aren't included.

Everything else was written new for Telnetvision for Windows. `CREDITS.txt` has line counts and exactly what came from where, and `docs/LICENSE.txt` has the licence texts.
