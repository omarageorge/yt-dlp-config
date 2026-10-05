# yt-dlp Custom Configuration

My personal [yt-dlp](https://github.com/yt-dlp/yt-dlp) configuration for Windows (PowerShell), so I don't have to set everything up from scratch on a new machine.

It provides short aliases for downloading videos (up to 1080p, MP4), playlists, channels, and audio (MP3, 320k, with cropped square cover art).

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration Setup](#configuration-setup)
- [Create a `dlp` Shortcut](#create-a-dlp-shortcut)
- [YouTube Session (Cookies)](#youtube-session-cookies)
- [Alias Reference](#alias-reference)
- [Config File](#config-file)

## Requirements

- [Node.js](https://nodejs.org/) (LTS)
- [yt-dlp](https://github.com/yt-dlp/yt-dlp)
- [FFmpeg](https://ffmpeg.org/) (needed for merging video/audio, converting to MP3, and embedding thumbnails; the winget yt-dlp package normally installs it as a dependency. Confirm with `ffmpeg -version`.)

## Installation

Run the following in PowerShell:

```powershell
winget install --id OpenJS.NodeJS.LTS
winget install --id Yt-dlp.yt-dlp
```

## Configuration Setup

1. Copy the contents of the [config file](#config-file) below.
2. Open PowerShell and navigate to the yt-dlp configuration directory:

   ```powershell
   cd $HOME\AppData\Roaming\yt-dlp\
   ```

3. Create an empty config file:

   ```powershell
   ni config
   ```

4. Open the new `config` file, paste the copied contents, and save it.

## Create a `dlp` Shortcut

This lets you type `dlp` instead of `yt-dlp`.

1. Open your PowerShell profile in Notepad:

   ```powershell
   notepad $PROFILE
   ```

2. Paste the following into the file:

   ```powershell
   function dlp { yt-dlp $args }
   ```

3. Save and close Notepad.
4. Restart the terminal.

## YouTube Session (Cookies)

The config already includes `--cookies-from-browser firefox`, which lets yt-dlp use your logged-in YouTube session (useful for age-restricted, members-only, or private content).

- Make sure you are logged in to YouTube in Firefox.
- To use a different browser, replace `firefox` in the first line of the config (for example, `--cookies-from-browser edge`).
- Firefox is recommended because yt-dlp can read its cookie data more reliably than from Chromium-based browsers (Chrome, Edge, etc.).

## Alias Reference

| Alias  | Type     | Description                                         | Output location                                       |
| ------ | -------- | --------------------------------------------------- | ----------------------------------------------------- |
| `--v`   | Video    | Single video, up to 1080p MP4                       | `Title.mp4`                                           |
| `--plv` | Video    | Playlist                                            | `PlaylistTitle/Index - Title.mp4`                     |
| `--plvr`| Video    | Playlist in reverse order                           | `PlaylistTitle/Index - Title.mp4`                     |
| `--ch`  | Video    | Channel, oldest first, skips already downloaded     | `Uploader/Number - Title.mp4`                         |
| `--chv` | Video    | Channel, oldest first, skips already downloaded     | `Number - Title.mp4` (current directory)              |
| `--a`   | Audio    | Single track, MP3 320k with embedded cover art      | `Title.mp3`                                           |
| `--al`  | Audio    | Album                                               | `AlbumName/Index - Title.mp3`                         |
| `--pla` | Audio    | Playlist                                            | `PlaylistTitle/Title.mp3`                             |

### Usage Examples

```powershell
dlp --v "<video-url>"
dlp --plv "<playlist-url>"
dlp --ch "<channel-url>"
dlp --a "<video-url>"
dlp --al "<album-playlist-url>"
```

### Notes

- The `--ch` and `--chv` aliases use `--download-archive archive.txt`, so re-running them only downloads new videos.
- Video aliases prefer H.264 (`avc`) video with M4A audio for maximum compatibility, falling back to other codecs if unavailable.
- Audio aliases crop the thumbnail to a square before embedding it, then delete the leftover thumbnail file. The cleanup command (`del`) is Windows-specific.

## Config File

```
# Global Cleanup & Settings
--cookies-from-browser firefox
--embed-metadata
--no-keep-video
--rm-cache-dir

# Video Alias
--alias --v "-f 'bv*[height<=1080][vcodec^=avc]+ba[ext=m4a]/b[height<=1080][vcodec^=avc]/bv*[height<=1080]+ba/b[height<=1080]' --merge-output-format mp4 -o '%(title)s.%(ext)s'"

# Video Playlist Alias (Downloads to PlaylistTitle/)
--alias --plv "-f 'bv*[height<=1080][vcodec^=avc]+ba[ext=m4a]/b[height<=1080][vcodec^=avc]/bv*[height<=1080]+ba/b[height<=1080]' --merge-output-format mp4 -o '%(playlist_title)s/%(playlist_index)s - %(title)s.%(ext)s'"

# Video Reversed Playlist Alias (Downloads to PlaylistTitle/)
--alias --plvr "-f 'bv*[height<=1080][vcodec^=avc]+ba[ext=m4a]/b[height<=1080][vcodec^=avc]/bv*[height<=1080]+ba/b[height<=1080]' --playlist-reverse --merge-output-format mp4 -o '%(playlist_title)s/%(playlist_index)s - %(title)s.%(ext)s'"

# Video Channel Alias (Downloads to Uploader/)
--alias --ch "-f 'bv*[height<=1080][vcodec^=avc]+ba[ext=m4a]/b[height<=1080][vcodec^=avc]/bv*[height<=1080]+ba/b[height<=1080]' --playlist-reverse --download-archive archive.txt --merge-output-format mp4 -o '%(uploader)s/%(autonumber)s - %(title)s.%(ext)s'"

# Video Channel Alias (Downloads to the current directory)
--alias --chv "-f 'bv*[height<=1080][vcodec^=avc]+ba[ext=m4a]/b[height<=1080][vcodec^=avc]/bv*[height<=1080]+ba/b[height<=1080]' --playlist-reverse --download-archive archive.txt --merge-output-format mp4 -o '%(autonumber)s - %(title)s.%(ext)s'"

# Audio Alias
--alias --a "-o '%(title)s.%(ext)s' -f 'ba' -x --audio-format mp3 --audio-quality 320k --write-thumbnail --embed-thumbnail --convert-thumbnails none --ppa \"EmbedThumbnail+ffmpeg_o:-c:v mjpeg -vf \\\"crop='if(gt(ih,iw),iw,ih):if(gt(iw,ih),ih,iw)'\\\"\" --exec \"post_process:del %(thumbnails.-1.filepath)q\""

# Audio Album Alias (Downloads to AlbumName/)
--alias --al "-o '%(album)s/%(playlist_index)s - %(title)s.%(ext)s' -f 'ba' -x --audio-format mp3 --audio-quality 320k --write-thumbnail --embed-thumbnail --convert-thumbnails none --ppa \"EmbedThumbnail+ffmpeg_o:-c:v mjpeg -vf \\\"crop='if(gt(ih,iw),iw,ih):if(gt(iw,ih),ih,iw)'\\\"\" --exec \"post_process:del %(thumbnails.-1.filepath)q\""

# Audio Playlist Alias (Downloads to PlaylistTitle/)
--alias --pla "-o '%(playlist_title)s/%(title)s.%(ext)s' -f 'ba' -x --audio-format mp3 --audio-quality 320k --write-thumbnail --embed-thumbnail --convert-thumbnails none --ppa \"EmbedThumbnail+ffmpeg_o:-c:v mjpeg -vf \\\"crop='if(gt(ih,iw),iw,ih):if(gt(iw,ih),ih,iw)'\\\"\" --exec \"post_process:del %(thumbnails.-1.filepath)q\""
```
