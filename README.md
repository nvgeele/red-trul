# red-trul: RED TRanscode & UpLoad

`red-trul` is a lightweight utility designed to non-interactively conditionally transcode FLAC releases, generating and uploading torrents.

- Detects the edition and media (_edition group_) of a release.
- Transcodes `Lossless` and `24-bit Lossless` to `Lossless`, `MP3 (320)`, and `MP3 (V0)` when the edition group lacks the specific transcode.
- Copies image files from the original, excluding everything else.
- Maintains the original folder structure.
- Rejects releases with bad tagging or incorrect bit-rate (for 24-bit FLAC).

## API access
Requires an API key with _Torrents_ capability, created from the settings page
at RED. No further authorization is needed.

## Install

You need:
- `nodejs` version 16 or later.
- `flac`
- `lame`
- `sox`
- `ffmpeg`
- `perl`

```bash
npm install -g red-trul
```

## Use

```
Usage: red-trul [OPTIONS] flac-dir

Options:
      --version           Show version number                          [boolean]
  -i, --info-hash         Torrent hash. Required unless an origin.yaml exists in
                           flac-dir.
      --torrent-id        Use the given torrent id. Alternative to --info-hash.
      --api-key           'Torrents'-capable API token. env-definable as RED_API
                          _KEY
  -o, --torrent-dir       Where to output torrent files           [default: "."]
  -t, --transcode-dir     Output directory of transcodes
      --dirname           Override the derived output directory name; '[<media>
                          <format>]' suffix is appended per encoding
      --no-flac           Don't transcode into FLAC                    [boolean]
      --no-v0             Don't transcode into V0                      [boolean]
      --no-320            Don't transcode into 320                     [boolean]
      --no-upload         Don't upload anything                        [boolean]
      --always-transcode  Always transcode (if tagged correctly)       [boolean]
  -h, --help              Show help                                    [boolean]
```

### Example

```bash
# RED_API_KEY is set in env.
# --announce=https://.../announce is optional is the 'User' capability is given.

red-trul --info-hash=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX \
    --torrent-dir=/home/lfen/rtorrent/watch \
    '/home/lfen/Music/TNO Project - There Is No Obsession EP (Flac24)'
# [-] using announce: https://flacsfor.me/<redacted>/announce
# [-] fetch torrent info...
# [-] analyze filelist...
# [-] ffprobe 6 files...
# [+] Required tags are present, would transcode this
# [-] permalink: https://redacted.ch/torrents.php?torrentid=4521975
# [-] grouplink: https://redacted.ch/torrents.php?id=2123754
# [-] fetch torrentgroup...
# [-] Transcoding /home/lfen/Music/TNO Project - There's No Obsession (2019) - WEB FLAC
# [-] Transcoding 01. TNO Project - Eradicating Deviants.flac...
# [-] Transcoding 02. TNO Project - Oneirology (Dance 4 Me).flac...
# [-] Transcoding 03. TNO Project - Exitus Letalis.flac...
# [-] Transcoding 04. TNO Project - Mendacium & Spem.flac...
# [-] Transcoding 05. TNO Project - Neural Interrogation.flac...
# [-] Transcoding 06. TNO Project - N05A (Who Are U).flac...
# [-] Transcoding /home/lfen/Music/TNO Project - There's No Obsession (2019) - WEB V0
# >> [3140914] Using 4 transcoding processes.
#
# [-] Transcoding /home/lfen/Music/TNO Project - There's No Obsession (2019) - WEB 320
# >> [3140942] Using 4 transcoding processes.
#
# [-] Uploading...
# [-] Write torrents to /home/lfen/rtorrent/watch/...
# [*] Done!
```

### Overriding the output directory name

RED's tracker rejects torrents containing a file path longer than 180 bytes.
If the derived directory name is too long (and the source directory cannot be
renamed), override it with `--dirname`:

```bash
red-trul --info-hash=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX \
    --dirname 'Shorter Artist - Short Album' flac-dir
```

The `[<media> <format>]` suffix (`[WEB V0]`, `[WEB 320]`, `[WEB FLAC16]`) is
still appended per encoding. Re-running the same release requires the *same*
`--dirname` to stay name-consistent with previously uploaded encodings.

## Advanced: post-download hook and watch directory.

This script runs well non-interactively. Post-download example scripts are
available for [qbittorrent](./qbittorrent-postdl.sh) and
[rtorrent](./rtorrent-postdl.sh). Beside configuring a post-download script for
the torrent client, also configure it to use the torrent-dir of trul as watch
dir for new torrents, and download those to the transcode dir to start seeding.

On Linux, use `red-trul-flock` to avoid running multiple instances of red-trul.
This waits until the current invocation finishes before starting, like a queue.
TODO for macos
