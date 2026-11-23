TELNETVISION FOR WINDOWS - SOURCE CODE
======================================

This archive is the complete source for every program in the Telnetvision
for Windows packages, plus the libraries they use and the web viewer. It
builds offline: nothing is downloaded.

Who wrote what, and how much came from the original Telnetvision and
from fTelnet, is in CREDITS.txt.


WHAT'S HERE
-----------

  build.bat          Builds everything on Windows (see BUILDING below).
  build.sh           The same, for building on Linux or macOS.
  README-SOURCE.txt  This file.
  CREDITS.txt        Where the code came from, line counts, licenses.
  CHANGES.txt        What's new in each version.

  producer\          telnetvision.exe - the camera-PC program. Plays files,
                     folders, cameras, capture cards, one screen or all
                     (screens.go), streams, IPTV channel lists (m3u.go),
                     YouTube (youtube.go, via yt-dlp.exe; downloaded ahead
                     by ytcache.go, sound fetched by webstream.go) and music
                     with a visualizer (viz.go, vizsource.go) through ffmpeg,
                     shrinks the picture for terminals and sends picture,
                     sound and captions to the server, and JPEG XL
                     pictures (jxl.go; fetches an ffmpeg that can make
                     them if needed, ffmpegget.go), jumping around in
                     a video (controls.go, player.go), lighter copies of
                     movies the PC can't decode in time (reencode.go)
                     and the playlist for viewers (playlistfeed.go).
                     Plain Go, no libraries.

  tvserver\          tvserver.exe - the server viewers connect to:
                       - telnet viewers (SyncTERM, IcyTerm, NetRunner, ...)
                       - the web viewer (built-in web server, fTelnet)
                       - the viewers' playlist lightbar (playlist.go)
                       - SyncTERM sound, sixel and JPEG XL pictures, the per-viewer
                         S menu, remembered viewer settings, door_port for
                         a real BBS
                     The picture and sound code comes from the original
                     Telnetvision door (the door_*.go files).
    tvserver\web\    The web viewer. It's built into tvserver.exe at
                     compile time, so changes here need a rebuild.
                       index.html        the page
                       tvaudio.js        sound, sixel and JPEG XL for browsers
                       jxl\              jSquash JPEG XL decoder (WebAssembly),
                                         unchanged, for browsers without one
                       ftelnet\          fTelnet (patched), fonts, keyboard
                       patch-ftelnet.py  the changes made to fTelnet

  door\              door.exe - the original Telnetvision BBS door, as
                     published upstream. Unchanged.

  service\           service.exe - the original Telnetvision relay, as
                     published upstream, with one bug fixed (CREDITS.txt).

  lib\shadecell\     Two small libraries by the original Telnetvision
  lib\termprobe\     author, used by tvserver and door. Unchanged.

  third-party\fTelnet\
                     The complete, unmodified fTelnet source (TypeScript)
                     and its release files. tvserver\web\ftelnet\ftelnet.js
                     is release\ftelnet.norip.noxfer.js with
                     patch-ftelnet.py applied.

  third-party\jSquash\
                     The license of the jSquash JPEG XL decoder in
                     tvserver\web\jxl (npm package @jsquash/jxl 1.3.0).

  third-party\telnetvision-upstream\
                     The original Telnetvision repository as it was used
                     (Python producer, Go door and service, docs), kept for
                     reference and comparison.

  docs\              User documentation and example settings:
                       README-USER-Win32.txt / README-USER-Win64.txt
                                         the README from the release zips
                       LICENSE.txt       the license notes in the release zips
                       telnetvision.ini.example
                       door.ini.example, BBS-DOOR-INSTALL.md
                                         for the BBS door
                       telnetvision-ad.txt  an ASCII ad for your BBS


BUILDING
--------

1. Install Go 1.24 or newer from https://go.dev/dl/ (the Windows .msi).
   Open a NEW command prompt afterwards so it can find "go".

2. Unzip this archive, open a command prompt in the folder and run:

       build.bat            32-bit and 64-bit
       build.bat 64         64-bit only
       build.bat 32         32-bit only
       build.bat test       run the automatic tests, then build both

3. The programs land in bin\Win32 and bin\Win64, laid out like the
   release zips:

       bin\Win64\camera-pc\telnetvision.exe   + telnetvision.ini.example
       bin\Win64\server\tvserver.exe
       bin\Win64\bbs-door\door.exe            + door.ini, INSTALL.md
       bin\Win64\bbs-door\service.exe
       bin\Win64\README.txt

No C compiler is needed (everything builds with CGO_ENABLED=0). The
32-bit programs run on 32-bit and 64-bit Windows; the 64-bit ones are a
little faster on 64-bit Windows.

ffmpeg isn't built from here. telnetvision.exe runs ffmpeg.exe to read
video, so the camera PC needs a copy (see the user README), and
yt-dlp.exe for YouTube. The server doesn't need either.

Building one program by hand works too, for example:

       cd tvserver
       set GOOS=windows
       set GOARCH=amd64
       set CGO_ENABLED=0
       go build -trimpath -ldflags="-s -w" -o tvserver.exe .


CHANGING THE WEB VIEWER'S fTELNET
---------------------------------

tvserver\web\ftelnet\ftelnet.js is generated, so make lasting changes in
patch-ftelnet.py and regenerate it (needs Python 3):

    python patch-ftelnet.py ..\..\third-party\fTelnet\release\ftelnet.norip.noxfer.js ftelnet\ftelnet.js

(run in tvserver\web). Then rebuild tvserver.exe.

To rebuild fTelnet itself from its TypeScript, see
third-party\fTelnet\README.md (it uses Node.js and build.cmd).


HOW THE PIECES TALK
-------------------

  telnetvision.exe  --feed (port 7600)-->  tvserver.exe  --telnet-->  viewers
   (camera PC)                              (server)     --web---->  browsers
                                                         --7601--->  door.exe
                                                                     (BBS)

The feed uses the original Telnetvision wire format (third-party\
telnetvision-upstream\wire.py), with these additions. Older and newer
programs ignore messages they don't know.

  0x10 FRAME, 0x11 AUDIO    as upstream
  0x12 FRAME_Z              zlib-packed frame, optional delta, 6-bit color
  0x13 FRAME_HI             sharper copy of the picture, for sixel viewers
  0x14 FRAME_HI_Z           the same, zlib-packed
  0x20 WELCOME              server -> producer: what the server understands
  0x21 WANT_HI              server -> producer: start/stop FRAME_HI

service.exe and door.exe use the plain upstream protocol, so the original
BBS setup in docs\BBS-DOOR-INSTALL.md still works unchanged.
