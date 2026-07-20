<h1 align="center"><strong><em>dmusic</em></strong></h1>

A highly efficient, minimalist, and keyboard-centric music playback and discovery workflow. While it strongly adheres to the [Suckless](https://suckless.org/) philosophy of simplicity and clarity, **dmusic is an independent project**.

![dmusic demo](assets/dmusic-demo.gif)

Instead of relying on heavy ncurses clients or GUIs, `dmusic` leverages simple, POSIX-compliant shell scripts and native utilities to manage music playback (`mpc` / `mpd`), interactive directory/playlist navigation (via `dmenu`), and music discovery (`curl`, `jq`, `mpv`).

> [!NOTE]
> **Modularity & Optional Features:** `dmusic` is composed of three standalone scripts. You can use the main player (`dmpc`) independently without the discovery (`smd`) or sorting (`msort`) tools. Furthermore, the `dmenu` vim-keys patch and stream preview dependencies are entirely **optional**.

## Components

1. **`dmpc` (ncmpcpp replacement)**
   A POSIX shell script providing a dynamic, vim-like interface to `mpd` via `dmenu`. 
   Features include:
   - Playback control (Toggle, Next, Prev, Shuffle, Repeat).
   - Queue management.
   - Interactive library browsing using `dmenu` directory traversal.
   - Built-in MPD database update option (`[Update DB]`).
   - Play entire directories/albums at once (`[Play Directory Now]`).
   - Filesystem-based playlist management (using symlinks).
   - Local playback history (`~/.cache/dmusic/history.txt`).

2. **`smd` (Simple Music Discoverer)**
   A POSIX shell script utilizing the Last.fm API to discover new music.
   Features include:
   - Find similar artists or tracks based on search or what is currently playing in MPD.
   - Browse top tracks by genre or global charts.
   - **Stream Preview**: Instantly stream discovered tracks in the background using `mpv` and `yt-dlp` without downloading.
   - Copy track names to clipboard for use in Soulseek/Nicotine+.
   - Add discoveries to a local wishlist file.

3. **`msort` (Music Sorter)**
   A POSIX shell script that reads embedded audio metadata via `ffprobe` and automatically organizes downloaded files into the library.
   Features include:
   - Reads FLAC, MP3, OPUS, OGG, AAC and M4A tags (artist, album, title, date).
   - Sorts files into `Artist/(YEAR) Album [FORMAT]/NN - Title.ext`.
   - Sequential track numbering based on existing files in the album directory.
   - Detects singles automatically (album tag equals title tag).
   - Dry-run mode (`-n`) to preview without moving.
   - Integrates with Nicotine+ as an `afterfinish` hook for fully automatic sorting.
   - Also accessible from the `dmpc` menu (icon `󰉒`).

4. **`dmenu-navkeys-5.4.diff`**
   A custom patch for `dmenu` (version 5.4) that introduces a vim-like navigation mode (`-vk`). 
   - Starts in normal mode: use `j`, `k` to navigate.
   - Overloads `h` (Left) and `l` (Right) to return custom exit codes (`10` and `11` respectively), enabling interactive shell scripts to traverse directory trees.
   - Automatically centers the items on the screen when used in horizontal layout.
   - Press `/` to enter standard `dmenu` search/insert mode.
   - Press `Escape` to clear search and return to normal mode.

## Dependencies

This workflow relies on the following packages.

### Core (Required for `dmpc`)
- **`mpd`**: Music Player Daemon (the core audio backend for your local library).
- **`mpc`**: Command-line client for MPD (used by `dmpc` to control playback and queues).
- **`dmenu`**: The menu interface. Works out of the box with stock `dmenu`, or you can use the optional patch for vim-like navigation.
- **Fonts**: The scripts use Nerd Font icons (specifically `nf-md` icons) for the interface. Ensure you have a compatible font installed (e.g., `nerd-fonts`).

### Optional (Required for `smd` discovery)
- **`curl`**: Network request utility to communicate with the Last.fm API.
- **`jq`**: Lightweight JSON processor to parse Last.fm data.
- **`mpv`**: Minimalist media player used to play background stream previews.
- **`yt-dlp`**: Video/audio downloader used as the streaming backend by `mpv`.
- **`xclip`**: Command line interface to the X11 clipboard to copy track names.

### Optional (Required for `msort` sorting)
- **`ffmpeg`**: Multimedia framework (provides `ffprobe` to read embedded audio tags).
- **`jq`**: Used to parse `ffprobe` metadata output.

On Void Linux, you can install the complete suite via `xbps-install`:

```bash
sudo xbps-install -S mpd mpc mpv yt-dlp ffmpeg curl jq xclip
```

## Installation

### 1. Patch and Build dmenu (Optional but Recommended)
For a better experience, you can build `dmenu` with the provided patch to enable vim-like directory traversal (`h`/`l` to go back and forth) and avoid the `[..] Go Back` artificial menu item.

```bash
git clone https://git.suckless.org/dmenu
cd dmenu
patch -p1 -i /path/to/dmusic/dmenu-navkeys-5.4.diff
sudo make clean install
```

If you apply this patch, you must explicitly enable it in your config file (see section 3). Otherwise, the scripts will use a stock (vanilla) `dmenu`.

### 2. Install Scripts
Place the scripts somewhere in your `$PATH` (e.g., `~/.local/bin` or `/opt/scripts`):

```bash
cp dmpc ~/.local/bin/
cp smd ~/.local/bin/
cp msort ~/.local/bin/
chmod +x ~/.local/bin/dmpc ~/.local/bin/smd ~/.local/bin/msort
```

**Nicotine+ hook (optional):** To automatically sort files after each Soulseek download, set the `afterfinish` field in `~/.config/nicotine/config`:
```
afterfinish = /path/to/msort "$"
```

### 3. Configuration
**Last.fm API Key:** `smd` requires a Last.fm API key to discover music. Get your free API key at [Last.fm](https://www.last.fm/api/account/create) and export it in your environment (e.g., `~/.zshenv` or `~/.bashrc`):
```bash
export LASTFM_API_KEY="your_api_key_here"
```

**Global Paths & Toggles:** The scripts respect the `XDG_CONFIG_HOME` and `XDG_CACHE_HOME` environment variables. You can override the default paths and enable the `dmenu` patch by creating a config file at `~/.config/dmusic/config`:
```bash
# ~/.config/dmusic/config
SONGS_DIR="/path/to/your/custom/songs"

# Set to 1 if you installed the dmenu-navkeys-5.4.diff patch
DMUSIC_DMENU_PATCHED=1
```

By default, if no config is provided, the scripts assume:
- Music Directory: `~/songs`
- Playlists Directory: `~/songs/playlists`
- Wishlist: `~/songs/wishlist.txt`

Make sure `mpd` is running and properly configured to use your music directory.

### 4. Directory Structure Example
Here is an example based on a typical user configuration (and the default paths used by the scripts):

```text
~/
├── .cache/
│   └── dmusic/
│       └── history.txt          # Automatically generated by dmpc
├── .config/
│   └── dmusic/
│       └── config               # Optional configuration file
├── .local/
│   └── bin/
│       ├── smd                  # The SMD script executable
│       └── msort                # The msort script executable
├── opt/
│   └── scripts/
│       └── Dmenu/
│           └── dmpc             # The dmpc script executable
└── songs/                       # Your main MPD music directory
    ├── temp/                    # Nicotine+ download dir (msort reads from here)
    ├── playlists/               # Folder where dmpc creates symlink playlists
    │   ├── rock/
    │   └── synthwave/
    ├── wishlist.txt             # Wishlist generated by smd
    ├── Artist A/
    │   └── (2024) Album Name [FLAC]/
    │       ├── 01 - Track.flac
    │       └── 02 - Track.flac
    └── Artist B/
```
## Usage

Simply run `dmpc` or `smd` from your terminal, or bind them to a hotkey in your window manager (like `dwm` or `sxhkd`).

- In the `dmenu` interface, use `j`/`k` to move up and down.
- Use `l` to select/enter a directory.
- Use `h` to go back a directory.
- Select `[Update DB]` in the main Library root to refresh MPD after manually adding files.
- When inside an artist/album directory, select `[Play Directory Now]` to instantly queue and play all its contents.
- Press `/` to search. Press `Escape` to cancel the search and return to normal navigation.
