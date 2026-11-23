This is a go lang implementation of 
https://github.com/hmderdoc/telnetvision

+ Thanks to the original authors and contributers.
+ Thanks to the ffmpeg author (and many many other things) Fabrice Bellard
+ Also, thanks to rick parrish for ftelnet which is used for the web player

This is real easy to setup.  don't be scared.  
make sure you have ffmpeg installed

TELNETVISION FOR WINDOWS 
=================================

Stream your webcam, your screen, a video file or a live stream, and let
anyone telnet in - or open a web page - to watch it as live ANSI art.
No BBS needed.

Two programs, which can run on the same PC or on different ones:

  camera-pc\telnetvision.exe   Captures the video and sends it to the server.
  server\tvserver.exe          Receives it. People telnet here, or visit
                               http://SERVER-ADDRESS:8080, to watch.

Requires 64-bit Windows 10 or 11. (For 32-bit Windows, use the Win32 package.)

Based on https://github.com/hmderdoc/telnetvision and fully compatible with
it: the classic BBS door and relay are included too (bbs-door folder).


1. START THE SERVER
-------------------

Double-click  server\tvserver.exe

The first time, it creates tvserver.ini with a random token and shows:

    Viewers telnet to : port 2323
                        telnet 192.168.1.50 2323
    Feed comes in on  : port 7600
    Feed token        : 3f9a...

Leave that window open. Windows may ask to allow it through the firewall -
say yes (tick Private networks; also Public if people connect from the
internet).


2. START THE CAMERA
-------------------

On the PC with the camera:

  a) Put ffmpeg.exe next to telnetvision.exe. Get the "essentials" zip
     from  https://www.gyan.dev/ffmpeg/builds/  and copy bin\ffmpeg.exe
     (or run:  winget install Gyan.FFmpeg  - then it's found automatically)

  b) Double-click  camera-pc\telnetvision.exe  and answer the questions:
       Server host name or IP  -> the server PC's address
                                  (127.0.0.1 if it's this same PC)
       Server feed port        -> 7600  (feed_port)
       Token                   -> the token tvserver.exe showed
       TLS                     -> either answer works with tvserver.exe
     Then pick a camera, your screen, a video file/URL, or the test pattern.

  The server window will say "feed ... is live".


3. WATCH
--------

From any computer:

    telnet SERVER-ADDRESS 2323

Works with SyncTERM, NetRunner, PuTTY, Windows Terminal, the Windows
telnet client, Mac/Linux terminals, and other BBS terminals. Each viewer
gets the picture fitted to their own window size, in 24-bit color where
the terminal supports it, else 16 colors (viewers can also pick 256),
in the right character set (CP437 for BBS terminals, UTF-8 for
PuTTY/Mac/Linux).

Press Q, X or ENTER to stop watching. C turns captions on/off,
T hides or shows the title bar, M mutes sound, S opens the settings menu,
+ and - make the picture faster or slower (frames per second).

VIEWER SETTINGS MENU (S): each viewer can lower the quality of their own
feed to suit their connection, without changing anyone else's:

  1  Colors       24-bit (best) -> 256 colors -> 16 colors (least data)
                  -> Sixel graphics (real pixels, for terminals that
                  show sixel: SyncTERM, IcyTerm, ...; see SIXEL below)
  2  Speed        frames per second, 1 up to max_fps (30). Also + / -
                  at any time, menu open or not; the new speed shows
                  along the top for a moment.
  3  Sound        Best (as broadcast) -> Good (22 kHz mono)
                  -> Low (11 kHz mono) -> Off
  4  Compression  ADPCM (1/4 size) -> mu-law (1/2 size) -> None
  5  Lip-sync     On (picture waits for its sound) / Off (least delay)
  6  Volume       -12 dB ... 0 (full) ... +12 dB (boosted by the server)
  7  Sixel detail High (sharpest) / Medium / Low (least data) - when
                  Colors is Sixel

Each viewer's choices are remembered (by their address and terminal
program, in server\viewers.json) and used again next time they connect.

The menu also shows how much data the viewer is using right now. Press
the number again to step to the next choice; S, ESC or ENTER closes it.
Turn the menu off with  viewer_settings = off  in tvserver.ini.

The bottom row is a title bar with the movie's name (from its file name:
The.Matrix.1999.mkv shows as "The Matrix 1999") and the banner from
tvserver.ini on the right. Subtitles appear just above it.

If the camera stops, viewers see "Broadcast paused" and the picture comes
back by itself when it resumes.


SIXEL GRAPHICS
--------------

Terminals that can show sixel graphics (SyncTERM, IcyTerm, and others)
can show the picture as real pixels instead of characters: viewers press
S and step Colors (1) to "Sixel graphics". It uses about as much data as
24-bit color (3-5 Mbit/s at 15 frames a second, full screen).

For a sharper sixel picture the camera PC sends a 160x96 copy as well,
but only while somebody is watching in sixel (SIXEL_WIDTH in
telnetvision.ini; 0 = never). Without it, sixel shows the normal picture.

Sixel detail (S, option 7) trades sharpness for data:
  High    the sharp 160x96 copy, sixel_colors colours      2.5-6.5 Mbit/s
  Medium  120x72, half the colours (the default)           2-4 Mbit/s
  Low     the normal 80x48 picture, a quarter of the colours 1.5-2 Mbit/s
(at 15 frames a second; busy, colourful video is at the high end)
Lowering the speed with - saves the same proportion again.

tvserver.ini:
  sixel = on          offer it in the menu (off = never)
  sixel_detail = medium   where viewers start (high | medium | low)
  sixel_colors = 64   colours per picture (more = nicer, more data)
  sixel_cell = 8x16   one character's size in pixels on the viewer's
                      screen (SyncTERM's normal font is 8x16). If the
                      picture comes out the wrong size, change this.

If a terminal shows garbage instead of a picture, it can't do sixel:
press 1 again to go back to characters.

Sixel works on the web page too: the page draws the sixel pictures
itself (fTelnet can't), scaled to the browser's font size.


WATCHING IN A WEB BROWSER
-------------------------

tvserver.exe also has a web page, so people can watch without installing
anything - picture AND sound, on a PC, Mac, tablet or phone:

    http://SERVER-ADDRESS:8080

They click "Watch with sound" and it plays. The page uses fTelnet (an
HTML5 BBS terminal) built into tvserver.exe, so browser viewers get
exactly what telnet viewers get: the title bar, captions, the S settings
menu, + and - for speed, M to mute, and the same connection pacing. The
page also has Mute, Full screen, Settings, Slower and Faster buttons for
touch screens. They count toward max_callers and show in  stats  as
"web browser (fTelnet)".

Settings in tvserver.ini:
  web_port   8080 (default). 0 turns the web page off.
  web_color  truecolor (default) | 256 | 16 - what browsers start with.
             Each viewer can change it with S.

To let people watch from the internet, forward the web port (8080) on
your router as well. The page is plain http; if you want https, put it
behind a web server that handles certificates (such as Caddy or nginx)
and pass /ws through to tvserver as a WebSocket.

Sound in browsers needs that one click (browsers block sound until you
interact with the page). Chrome, Edge, Firefox and Safari all work.


SOUND (SyncTERM viewers)
------------------------

If a SyncTERM viewer gets no sound, look for the "sound debug" lines in
the tvserver window (or tvserver.log). They show how the terminal answered
four questions and say what's wrong in plain words - for example that
SyncTERM is ignoring all its sound commands, which it does when it can't
create its cache folder for the connection (an entry Name containing any
of  : \ / * ? " < > |  or a cache folder it can't write to). A second line
10 seconds later says whether SyncTERM is playing the clips it gets.
sound_debug = on in tvserver.ini writes these lines for every caller.

Viewers using SyncTERM 1.10a or newer hear the sound too. Everyone else
gets the picture only - nothing to configure per viewer; the server asks
each terminal when it connects.

  1. In camera-pc\telnetvision.ini set AUDIO=1 (or answer yes in setup).
     AUDIO_SOURCE=auto uses a video file/stream's own sound, otherwise
     your first microphone; mic:<name> picks a specific input.
  2. In server\tvserver.ini leave  audio = auto.

Sound is compressed with IMA ADPCM for SyncTERM (a quarter of the size of
uncompressed sound) and with mu-law for other terminals that play
SyncTERM's sound commands, such as IcyTerm, which can't play ADPCM
(sound_compression = auto, the default). Viewers can change it with S,
and it's remembered for them. If a viewer's terminal takes the sound but
never plays it (some terminals, or some SyncTERM audio-output settings,
can't play every format), the server notices within a few seconds and
switches that viewer to a simpler format - ADPCM, then mu-law, then
uncompressed - and remembers it. To make sound louder or quieter, viewers
use Volume (S, 6); audio_volume_db in tvserver.ini sets where everyone
starts (0 = full; above 0 the server boosts it).
Bandwidth each listener needs:

                                       ADPCM (default)  mu-law    none
  AUDIO_RATE=11025  AUDIO_CHANNELS=1   0.08 Mbit/s      0.14      0.3
  AUDIO_RATE=22050  AUDIO_CHANNELS=1   0.14 Mbit/s      0.25      0.5
  AUDIO_RATE=44100  AUDIO_CHANNELS=2   0.5  Mbit/s      0.95      1.9

AUDIO_CODEC in telnetvision.ini picks what the camera PC sends:
adpcm (default), ulaw, or pcm (uncompressed). sound_compression in
tvserver.ini picks what viewers get by default (adpcm); the server
converts per viewer, and each viewer can change it with S. If a
viewer's sound ever sounds wrong, they can pick Compression: None.
(FLAC isn't used: SyncTERM can only play it when built with libFLAC,
and ADPCM is smaller anyway.)

A good setup for most: AUDIO_RATE=44100 AUDIO_CHANNELS=2 on the camera
PC (0.5 Mbit/s with ADPCM), and viewers on slow links pick Good or Low.

Sound always goes out ahead of the picture, so on a slow connection the
picture slows down while the sound keeps playing smoothly, and lip-sync
is kept by skipping or briefly holding frames.

Viewers press M to mute or unmute. To fix lip-sync, change
AUDIO_OFFSET_MS in telnetvision.ini (positive = sound earlier) or
sync_slack_ms in tvserver.ini. audio_volume_db sets the loudness.


SUBTITLES AND CLOSED CAPTIONS
-----------------------------

On by default (SUBTITLES=auto in telnetvision.ini). When the video has
captions, viewers see them in a white-on-blue bar under the picture, up to
three lines, with long lines wrapped. Each viewer can press C to hide or
show them; set  captions = off  in tvserver.ini to start them hidden.

Where they come from - auto uses the first one it finds:

  1. A subtitle file next to the video with the same name:
       C:\Videos\movie.mp4  +  C:\Videos\movie.srt
     (also movie.en.srt, movie.vtt). English is preferred if there are
     several languages.
  2. A subtitle track inside the file or stream (mkv, mp4, ts).
     Pick one with SUBTITLE_TRACK=1 (second track) or SUBTITLE_TRACK=spa.
  3. Closed captions (CC) carried in the video - US broadcast TV, an
     HDHomeRun tuner URL, recorded .ts files. They appear the moment the
     broadcast shows them.

Other settings:

  SUBTITLES=off                    no subtitles
  SUBTITLES=cc                     always use closed captions
  SUBTITLES=track                  always use a subtitle track
  SUBTITLES=C:\Subs\movie.srt      use this file with the video

The camera window says what it found, e.g. "subtitles: closed captions"
or "subtitles: subtitle file movie.srt (812 lines)". Press C there to
stop sending captions to everyone.

Not supported: picture-based subtitles from DVD and Blu-ray rips (PGS,
VobSub) - they're images, not text. Put a .srt file next to the video
instead (many are available online, or convert with Subtitle Edit).

LIVE CAPTIONS: CAPTION_FILE=C:\Temp\caption.txt shows the last line of
that text file, ahead of the subtitles above. Use it for speech-to-text
on a live camera (see the upstream README's whisper.cpp setup) or to type
announcements: anything that writes to the file appears within a frame.


SERVER SETTINGS (server\tvserver.ini)
-------------------------------------

Edit in Notepad, then restart tvserver.exe.

  telnet_port   Port people telnet to. Default 2323. Use 23 for the
                standard telnet port (then viewers can just type
                "telnet SERVER-ADDRESS").
  feed_port     Port telnetvision.exe sends to. Default 7600.
  token         Feed password. Must match TOKEN in telnetvision.ini.
  channel       Which feed viewers watch (CHANNEL in telnetvision.ini).
  listen        Network address to listen on. 0.0.0.0 = all.
  max_callers   Most viewers at once. Default 50.
  banner        Title on the "waiting for the broadcast" screen.
  encoding      auto | cp437 | utf8   (force one if auto guesses wrong)
  color         auto | truecolor | 256 | 16   (where viewers start;
                they can change it with S)
  fit           letterbox (keep shape) | stretch (fill the window)
  fps           Frames per second viewers start at. Default 15.
  max_fps       The most a viewer can raise it to with +. Default 30.
                A viewer can't get more than the camera PC sends (FPS in
                telnetvision.ini), so set FPS=30 there to allow 30.
  tls_cert/tls_key
                Your own certificate files for the feed. Blank = an
                automatic one (keep INSECURE=1 in telnetvision.ini).
  captions      on | off - whether viewers start with captions showing.
  title_bar     on | off - whether viewers start with the title bar.
  door_port     Also serve the classic BBS door.exe (e.g. 7601). 0 = off.
  web_port      Web page for watching in a browser. Default 8080. 0 = off.
  web_color     truecolor | 256 | 16 - colors browser viewers start with.
  audio         auto (send sound to SyncTERM viewers) | off.
  audio_volume_db   Starting volume in dB: 0 = full (default), -6 = half,
                    +6 = boosted. Viewers can change their own with S.
  lip_sync          on | off - picture waits for its sound (see PICTURE
                    AND SOUND STAYING TOGETHER). Viewers can switch it.
  max_sync_delay_ms Longest the picture waits for its sound. Default 300.
  sync_slack_ms     Lip-sync tolerance in ms. Default 120.
  sound_compression adpcm (default) | ulaw | none - what viewers start with.
  sound_quality     best | good (22 kHz mono) | low (11 kHz mono) | off
                    - what viewers start with.
  viewer_settings   on | off - whether viewers can press S for the menu.

Ports can also be set on the command line for one run:
    tvserver.exe --telnet-port 23 --feed-port 7600 --web-port 8080

Messages are written to tvserver.log next to tvserver.exe.


PICTURE AND SOUND STAYING TOGETHER
----------------------------------

For a video file or stream, telnetvision.exe takes the picture and the
sound from one ffmpeg and stamps both with their time in the movie, so
the server can keep them together however slow or busy the PC is. (In
tests the picture now lands within 10 ms of its sound; before, it ran
about 0.2 s behind, sometimes up to half a second.)

The same goes for a camera with a microphone on Windows - e.g. the
SplitCam camera with AUDIO_SOURCE=mic:SplitCam - which are captured
together as one source.

Keeping them together has a cost: the picture is held back to match the
sound, which itself sits in a small buffer at the viewer (usually 0.2 to
0.5 s). If the picture feels laggy:

  lip_sync = off           in tvserver.ini: the picture is shown the
                           moment it arrives (least delay; the sound may
                           be a little behind). Viewers can also switch
                           it per viewer with S, option 5.
  max_sync_delay_ms = 300  in tvserver.ini: the longest the picture ever
                           waits for its sound. Lower = less delay.
  SYNC_CAPTURE=0           in telnetvision.ini: capture picture and sound
                           separately, as versions before the lip-sync
                           change did (use AUDIO_OFFSET_MS to line up).

If tvserver says "the picture arrives N ms behind its sound", the
picture is held up on its way from the camera PC - nearly always its
upload. telnetvision.exe now packs pictures before sending them to
tvserver (FEED_COMPRESS=1, on by default): about a quarter to half the
size, e.g. 1.4 -> 0.4-0.6 Mbit/s at 80x24, 15 fps. FEED_COLOR_BITS=6
(default) rounds colours to 262,144, which looks the same at this size;
8 = exact, 5 = smallest. Both programs must be this version or newer.

To see how long the picture is being held, type  stats  in the tvserver
window: each viewer line says "picture held N ms for lip-sync".

Picture and sound are still separate (so can drift) when the sound is
a different source from the picture: a microphone over a video file,
or sound with the screen as the picture. Use AUDIO_OFFSET_MS to line
those up by hand.


SPEED, LAG AND VIEWER STATISTICS
--------------------------------

Each viewer needs roughly this much (80x25 at 15 frames/s):

  24-bit color + 22 kHz mono sound (ADPCM)     about 3.7 Mbit/s
  24-bit color + 44 kHz stereo sound (ADPCM)   about 4 Mbit/s
  256 colors   + 22 kHz mono sound (ADPCM)     about 0.6-1 Mbit/s
  16 colors    + 22 kHz mono sound (ADPCM)     about 0.4 Mbit/s

(The picture depends on the video: busy, noisy scenes need more.)

The server's upload has to carry that for every viewer at once.

tvserver.exe measures each viewer's connection and sends the picture at a
little under what it can take, so a slow viewer gets fewer frames per
second instead of falling further and further behind. Sound always goes
first.

To see what each viewer is getting, type  stats  in the tvserver.exe
window and press Enter (type it again to stop). Every 10 seconds it shows
the feed arriving from the camera PC and, for each viewer: frames per
second shown, Mbit/s sent (picture + sound), whether their connection is
the limit, sound status, and a verdict such as

  BOTTLENECK: this viewer's connection is too slow for this picture
  BOTTLENECK: the camera PC only sends 8.0 fps

Other commands:  stats 5  (every 5 seconds),  who  (one report now),
help.  To have it on from the start, set  stats = on  in tvserver.ini,
or run  tvserver.exe --stats.  Reports also go to tvserver.log.

Ways to lighten the load, biggest first (viewers can do the first
three for themselves with S):
  color = 256 or 16         in tvserver.ini   (picture 3-4 Mbit/s -> 0.2-1)
  fps = 10                  in tvserver.ini
  sound_quality = good      in tvserver.ini, or AUDIO_RATE=22050 and
                            AUDIO_CHANNELS=1 in telnetvision.ini
  AUDIO_CODEC=adpcm         (the default) in telnetvision.ini
  AUDIO=0                   if you don't need sound

Sound buffer (tvserver.ini): audio_prebuffer and audio_max_queued, in
80 ms chunks. Raise both if sound is choppy (adds delay); lower them to
cut delay. With sound on, the picture is held back to match the sound,
which adds about a quarter to half a second.


LETTING PEOPLE WATCH FROM THE INTERNET
--------------------------------------

On your router, forward the telnet port (2323, or 23) to the server PC,
and the web port (8080) for people watching in a browser.
If the camera PC is somewhere else on the internet, forward the feed port
(7600) too. If both programs are on your home network, you only need to
forward the telnet port.


CAMERA SETTINGS (camera-pc\telnetvision.ini)
--------------------------------------------

Run  telnetvision.exe --setup  to answer the questions again, or edit it
in Notepad. The most useful:

  SOURCE    camera | camera:<name> | test |
            screen (all screens) | screen1 | screen2 ... | window:<title> |
            C:\Videos\clip.mp4 | C:\Music\song.mp3 | C:\Movies (a folder) |
            rtsp://... | http://... | an IPTV list (.m3u) |
            a YouTube channel, playlist or video | visualizer
            (see SCREENS, MUSIC, IPTV CHANNEL LISTS and YOUTUBE below)
  SHUFFLE   1 = play a playlist in random order (default 0: in order).
  LOOP      0 = stop at the end of a playlist (default 1: start over).
  MESSAGE_SECONDS
            How long a message typed with T stays up (0 = until cleared).
  SHOW_TITLE 1 = send the movie's name for viewers' title bar (default).
  TITLE     Name shown in the title bar for a camera, the screen or a
            stream, which have no file name. Blank = no bar for those.
  VISUALIZER auto (default) = a different look now and then, or the name
            of one to start on (see MUSIC below).
  VIZ_SECONDS How often auto changes the look (default 25; 0 = only V).
  COLS/ROWS Picture size in characters (80x24 = classic BBS screen).
  FPS       Frames per second sent to the server (the most any viewer
            can get; each viewer picks up to max_fps with + and -).
  SUBTITLES auto | off | cc | track | a .srt/.vtt file (see below).
  CAPTION_FILE
            A text file whose last line is shown as a live caption.
  AUDIO     1 = send sound (see SOUND below).
  AUDIO_CODEC adpcm (default, 1/4 size) | ulaw (1/2) | pcm (uncompressed).
  HWACCEL   auto (default) = let the graphics card decode movies, much less
            work for the processor; off = never.
  FAST_DECODE 1 (default) = skip decoding work invisible at this size.
  FEED_COMPRESS 1 = pack pictures for the trip to tvserver (default).
  FEED_COLOR_BITS 6 (default) | 8 exact | 5 smallest.
  SYNC_CAPTURE 1 = picture and sound from one ffmpeg (default, in sync);
            0 = separately, like older versions.

To list camera and microphone names:  telnetvision.exe --list-devices

Keys in the camera window (press H to see them on screen):
  S    play something else: type a file, folder, URL, IPTV list,
       YouTube link, camera:<name>, screen or test, then Enter (Esc
       cancels). What's playing keeps going until the new one is ready.
       Or just drag a file or folder from Explorer onto the window: it's
       filled in on the Play line - press Enter to play it, Esc to cancel.
       Pasting works too: Ctrl+V, Shift+Insert or a right-click puts
       what you copied (a YouTube link, a path...) on the Play line, or
       into whatever line you're typing.
  A    add a file, folder, IPTV list or YouTube link to the end of the
       playlist (the current video keeps playing)
  N    next video or channel        B    back one
  L    list the playlist or the channels: type a number, or part of a
       name (words in any order) and Enter plays the first match. With
       the preview on, the matches show as you type; without it, Tab
       prints them.
  V    next visualizer look (while music plays)
  R    shuffle on/off: the rest of the playlist plays in random order
       (the current video keeps playing). SHUFFLE=1 in telnetvision.ini
       starts with shuffle on.
  T    type a message for viewers, then Enter. It shows in yellow above
       any subtitles - even for viewers who turned captions off - for
       MESSAGE_SECONDS (15 by default). The status line says MESSAGE ON
       while it's up. Enter on an empty line clears it.
  X    clear the message
  + -  saturation   [ ]  contrast   < >  brightness   0  reset
  M    half-block / text style      G  text glyphs
  C    captions on/off              P  preview on/off
  Q    quit             Esc  close the help, or cancel typing
                           (Esc never quits, so a stray press can't
                           stop your broadcast)

While telnetvision.exe runs, the window's Quick Edit mode is switched off
(it comes back when the program closes). To copy text out of the window
(a log line, say), right-click the title bar > Edit > Mark, select it
with the mouse and press Enter. With Quick Edit on, clicking in
the window starts a text selection that freezes the window and swallows
Esc. The stream to viewers now keeps going even if the window freezes
for any other reason.

Changes made with S and A last until telnetvision.exe is closed; the
next start uses SOURCE from telnetvision.ini again. If something you
switch to won't play (a camera that isn't plugged in, say), it goes back
to what was playing.


FOLDERS OF VIDEOS
-----------------

SOURCE=C:\Movies plays every video in that folder and all its
subfolders, one after another, in natural order (Episode 2 before
Episode 10, each folder's files together). At the end it starts over,
and picks up any files you've added in the meantime. Each video's own
sound and subtitles (a .srt next to it, or a subtitle track) follow it.
Files that won't play are skipped. Hidden folders and the Recycle Bin
are ignored. Video types: mp4 mkv avi mov wmv flv webm mpg ts m2ts vob
m4v 3gp ogv divx rmvb and more.


SCREENS
-------

SOURCE=screen captures all your monitors as one wide picture. To send
just one:

    SOURCE=screen1      the main monitor
    SOURCE=screen2      the next one (then screen3 ...), left to right

telnetvision.exe --list-devices lists them with their sizes and
positions, and --setup offers each one. Also:

    SOURCE=window:Notepad            one window, by (part of) its title (keep it
                                     on screen: not minimized or covered)
    SOURCE=screen:1920,0,1280x720    any area: X,Y from the main monitor's
                                     top left, then width x height

If a screen comes out cropped or shifted (it can happen with display
scaling set above 100% on some monitors), use screen:X,Y,WxH with the
numbers from --list-devices.


MUSIC
-----

Songs play with a music visualizer as the picture, in the spirit of
Milkdrop: tunnels, spirals, kaleidoscopes, spectrum bars, plasma and
more, moving with the bass, mids, treble and beats. Viewers hear the
song and see it in sync.

  - A song: SOURCE=C:\Music\song.mp3 (mp3, flac, ogg, opus, m4a, aac,
    wav, wma, aiff, ...)
  - A folder of music (or music and videos mixed): every song becomes
    part of the playlist - N, B, R (shuffle) and L work as usual. Songs
    with their artist/title tags show them in viewers' title bar.
  - A .m3u playlist of songs, or an internet radio stream (an Icecast /
    Shoutcast link, or radio stations in an IPTV list).
  - SOURCE=visualizer: the visualizer for your microphone or line-in
    (AUDIO_SOURCE picks which; for a DJ mixer or a record player).

The look changes by itself every VIZ_SECONDS (25), blending smoothly on
a beat. Press V for the next one. VISUALIZER= picks the first look
(and keeps it until V): Tunnel, Spiral Galaxy, Kaleidoscope, Plasma
Ocean, Spectrum City, Starburst, Vortex, Fire, Mirror Scope, Pulse
Rings, Rain, Neon Flower.

Songs with album art still get the visualizer (the cover is a still
picture). Sixel viewers get the visualizer at full detail.


IPTV CHANNEL LISTS
------------------

SOURCE (or S) can be an IPTV channel list: a .m3u / .m3u8 file, or the
link your provider gives you, for example

    SOURCE=C:\TV\channels.m3u
    SOURCE=http://provider.example/get.php?username=me&password=pw&type=m3u_plus

Every channel in it becomes an entry in the playlist, named as in the
list (with its group, like News or Sports, in brackets on the L list).

  - A channel plays until you change it: N next, B back, L to pick one
    by number or name. If it drops, it reconnects by itself.
  - A channel that won't open (dead link, provider down) is skipped
    after a few tries, with a note in the window.
  - Movies and shows in the list (anything with an end) play once at
    normal speed, then the list goes on to the next entry.
  - User-agent and referrer lines in the list (#EXTVLCOPT, or
    "link|User-Agent=..." Kodi style) are passed on to ffmpeg.
  - A .m3u8 that is a single stream (HLS), not a list, just plays as
    that stream.
  - A list of files on this PC works too (paths relative to the list).

Web streams are read at their own speed, so live HLS channels (most IPTV)
play smoothly instead of in once-a-second jumps, and a web video (an .mp4
link) plays at normal speed instead of racing through as it downloads.

Big provider lists (thousands of channels) take a few seconds to load;
L with a few letters of the name is the quick way to find a channel.


YOUTUBE
-------

YouTube needs one more free program, yt-dlp.exe, from

    https://github.com/yt-dlp/yt-dlp/releases/latest

Download yt-dlp.exe (on 32-bit Windows, yt-dlp_x86.exe) and put it in
the camera-pc folder next to telnetvision.exe. Update it now and then
(run  yt-dlp.exe -U ): YouTube changes often, and an old yt-dlp is the
usual reason a video stops working. If yt-dlp says it needs a
JavaScript runtime, install Deno (winget install DenoLand.Deno).

Then SOURCE (or S) can be:

  a channel     https://www.youtube.com/@channelname
                While the channel is live, its live stream plays.
                Otherwise its 50 most recent videos play as a playlist,
                newest first (N, B, R and L work). The channel is checked
                every 5 minutes: when it goes live, the live stream takes
                over, and when the stream ends the video it interrupted
                plays again. At the end of the videos the list is fetched
                again, so new uploads show up.
  a playlist    https://www.youtube.com/playlist?list=...
  a video       https://www.youtube.com/watch?v=...  (or a youtu.be link,
                or a live stream's link)

Videos are fetched at 360p-480p, plenty for a terminal picture, and play
at normal speed with their sound and title.

Each video is downloaded completely before it plays (the status line
shows "downloading 45%"), and while it plays the next one in the
playlist downloads - one at a time - so it's ready when its turn comes
and the network can't interrupt it. Downloads go to a
telnetvision-youtube folder in your temp folder and are deleted once
played and when telnetvision closes. Live streams can't be downloaded
ahead, so they're streamed. YOUTUBE_PREFETCH=0 streams every video
instead.


IF MOVIES PLAY SLOWLY ON THE CAMERA PC
--------------------------------------

Decoding a 1080p or 4K movie is the heaviest job in the whole setup, and
it all happens on the camera PC. If its window shows fewer frames a second
than FPS (and it will say "this PC can't decode ... fast enough"):

  - Use the 64-bit package and a 64-bit ffmpeg if Windows is 64-bit
    (Settings > System > About > System type). 32-bit ffmpeg is slower.
  - Keep HWACCEL=auto and FAST_DECODE=1 (the defaults).
  - PREVIEW=0: drawing the live picture in the window costs CPU too.
  - A lower FPS (10 instead of 15).
  - SYNC_CAPTURE=0: picture and sound come from separate ffmpegs again,
    so the sound stays smooth even when the picture can't keep up.
  - 720p versions of movies decode about twice as fast as 1080p.


UPGRADING / YOUR SETTINGS FILES
-------------------------------

To upgrade, replace the .exe files and keep your .ini files.

Every time tvserver.exe and telnetvision.exe start, they tidy their .ini:

  - Every setting is put back in its proper section, under its current
    explanation, in the standard order.
  - Settings a new version added are filled in at their defaults, with
    their explanations, so nothing behaves differently until you change
    them.
  - Your values are kept exactly as you wrote them. If a setting appears
    twice, the one that was in effect (the last one) is kept.
  - Settings you switched off with a # (like  # web_port = 9000) stay
    switched off, moved to their proper place.
  - Old setting names are updated (HOST= becomes BBS_HOST=, MIRROR= becomes
    PREVIEW=).
  - Settings the program doesn't know (a typo, or one from another
    program) are kept at the bottom under a note, so nothing is lost.
  - Your own comments are replaced by the standard explanations.

They say what they changed when they start. The old file is always copied
to tvserver.ini.bak / telnetvision.ini.bak first, and a file that's
already tidy is left alone.

telnetvision.exe --setup edits your existing file in place: only the
answers you change are rewritten, and pressing Enter keeps the current
setting (it recognises partial device names like mic:SplitCam).


RUNNING AUTOMATICALLY AT STARTUP
--------------------------------

Task Scheduler -> Create Task -> Actions -> Start a program -> the .exe,
Trigger "At log on" (or "At startup"). Both programs reconnect and
recover on their own.


USING A REAL BBS INSTEAD (OPTIONAL)
-----------------------------------

The bbs-door folder has the original relay (service.exe) and door
(door.exe) from the upstream Telnetvision project, with its INSTALL.md.
Or set door_port = 7601 in tvserver.ini and point door.exe at it - the
same server then feeds telnet viewers and your BBS at once.


SOURCE CODE
-----------

The full source for all four programs (telnetvision.exe, tvserver.exe,
door.exe, service.exe), the libraries they use and the web viewer is in
Telnetvision-Source.zip. Unzip it and run build.bat (needs Go 1.24 or
newer from https://go.dev/dl/; no internet connection needed). See
README-SOURCE.txt and CREDITS.txt inside it.


