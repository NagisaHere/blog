---
layout: post
title:  "Self-Hosting Your Own Spotify on CasaOS"
author: "Ryan Wang"
date:   2026-06-06 00:00:00 +1000
categories: 
tags: homelab casaos music navidrome
---

In peak exam procrastination, I've been putting together a self-hosted music stack on my home server; recommendations, downloading, and a mobile player. Here's how the pieces fit together.

## Overview

| Role | Service |
| --- | --- |
| Music library & streaming | [Navidrome](https://www.navidrome.org/) |
| Music recommendations | [Explo](https://github.com/LumePart/Explo) |
| Downloading (Spotify) | [Downtify](https://github.com/henriquesebastiao/downtify) |
| Downloading (YouTube Music) | [Yubal](https://github.com/guillevc/yubal) |
| Mobile player | [Substreamer](https://substreamer.org/) |

**Navidrome, Explo, Downtify, and Yubal all share the same media folder**. Downloads land in one place, Navidrome indexes them, and Explo writes its discovered tracks into the same library.

```
/path/to/media/music
├── (Downtify downloads)
├── (Yubal downloads)
└── explo/          ← Explo subfolder (recommended)
```

## Shared Media Folder

Before installing anything, pick a single host path for your music library e.g. `/DATA/Media/music` on CasaOS and mount it into every container that reads or writes music files. Doing this via the CasaOS UI was relatively straightforward.


## Music Library — Navidrome

Navidrome is the Subsonic-compatible server that actually streams your collection, which we can install from the app store.

### Setup

1. Install Navidrome from the CasaOS app store.
2. Point the music volume to your shared media folder (e.g. `/DATA/Media/music`).
3. Open the web UI, create an admin account, and let Navidrome scan the library.

Navidrome handles transcoding, playlists, and the Subsonic API that mobile clients like Substreamer talk to.

Before we proceed, its best to create a free listen brainz account [here](https://listenbrainz.org/). You'll need this to submit your listens, so grab your user token [here](https://listenbrainz.org/profile/).

## Music Recommendations — Explo

[Explo](https://github.com/LumePart/Explo) is a self-hosted alternative to Spotify's Discover Weekly. It pulls personalised playlists from [ListenBrainz](https://listenbrainz.org/), downloads missing tracks via YouTube, and creates playlists directly in Navidrome.

I deployed Explo using the base `docker-compose.yaml` from the repo and finished configuration through the web UI.

### Docker Compose

Create a folder (e.g. `explo/`) with the config subfolders and paste the base compose file from the [Explo repo](https://github.com/LumePart/Explo/blob/main/docker-compose.yaml):

```yaml
services:
  explo:
    image: ghcr.io/lumepart/explo:latest
    restart: unless-stopped
    container_name: explo
    ports:
      - "7288:7288"
    volumes:
      - ./explo/.env:/opt/explo/.env
      - ./explo/config:/opt/explo/config
      - /path/to/media/music:/data/
    environment:
      - TZ=Australia/Brisbane
      - WEB_UI=true
      - UI_USERNAME=youruser
      - UI_PASSWORD=yourpassword
```

Replace `/path/to/media/music` with the same shared media folder Navidrome uses. Explo recommends putting its downloads in a subfolder like `/data/explo/`. Make sue to also set your web ui user and password as well.


### Web UI Setup

1. Run `docker compose up -d` and open `http://YOUR_SERVER_IP:7288`.
2. Log in with the credentials from the compose file.
3. Follow the setup wizard:
   - **Music system** — connect to Navidrome (URL, username, password). Note here your url should be http://navidrome:4533.
   - **ListenBrainz** — link your account so Explo can fetch Weekly Exploration / Weekly Jams / Daily Jams playlists.
   - **Downloaders** — enable YouTube via yt-dlp.

### YouTube Data API Key

Explo needs a YouTube Data API key from [Google Cloud Console](https://console.cloud.google.com/):

1. Create a project (or use an existing one).
2. Enable the **YouTube Data API v3**.
3. Create an API key under **Credentials**.
4. Paste the key into Explo's web UI under the YouTube downloader settings.

The free tier (10,000 units/day) is more than enough for a weekly discovery tool.

Once you open the web ui, you should be able to follow its instructions to set it up.

## Connecting Explo and Navidrome
By default, Navidrome and Explo won't be located on the same network. Navidrome comes with its own bridge network you can use, which we can set in the ui.



Go listen to a couple songs, and when a daily recommended playlist is generated, you can manually download through the test here to see if it works.
<!-- image or smth -->

## Music Downloading

For manually grabbing tracks outside of Explo's automated discovery, I use two downloaders; one for Spotify links and one for YouTube Music.

### Downtify (Spotify)

[Downtify](https://github.com/henriquesebastiao/downtify) wraps spotDL in a clean web UI. Paste a Spotify track, album, or playlist URL and it downloads tagged audio sourced from YouTube.

On CasaOS, install from the app store or deploy manually:

```yaml
services:
  downtify:
    container_name: downtify
    image: ghcr.io/henriquesebastiao/downtify:latest
    ports:
      - "8000:8000"
    volumes:
      - /path/to/media/music:/downloads
    restart: unless-stopped
```

Open `http://YOUR_SERVER_IP:8000`, paste a Spotify link, and files land directly in the shared media folder with album art and metadata.

### Yubal (YouTube Music)

[Yubal](https://github.com/guillevc/yubal) does the same for YouTube Music — paste a track, album, or playlist link and get a tagged, organised library.

```yaml
services:
  yubal:
    image: ghcr.io/guillevc/yubal:latest
    container_name: yubal
    ports:
      - "8001:8000"
    environment:
      PUID: 1000
      PGID: 1000
      YUBAL_SCHEDULER_CRON: "0 0 * * *"
      YUBAL_DOWNLOAD_UGC: false
      YUBAL_TZ: Australia/Brisbane
    volumes:
      - /path/to/media/music:/app/data
      - ./yubal/config:/app/config
    restart: unless-stopped
```
You can just use any free ports on your host.

Make sure `PUID`/`PGID` match your CasaOS user so file permissions stay consistent across all containers.

After either downloader finishes, trigger a Navidrome scan (or wait for the next scheduled one) and new tracks show up in the library.
>>>>>>> d529a7f (added local music setup)

## Music Player (Mobile) — Substreamer

[Substreamer](https://substreamer.org/) is a free, open-source Subsonic client for iOS and Android. It connects directly to Navidrome and supports offline playback, playlists, Last.fm scrobbling, and Chromecast.

### Setup

1. Install Substreamer from the [App Store](https://apps.apple.com/us/app/substreamer/id1012991665) or [Google Play](https://play.google.com/store/apps/details?id=com.ghenry22.substream2).
2. Add your Navidrome server URL — use your Tailscale hostname or reverse-proxy URL if connecting remotely.
3. Enter your Navidrome username and password.

### Using with VPN

So long as you have a way to reach your home network (e.g. via tailscale), that's it; your entire self-hosted library is streamable from your phone, including anything Explo, Downtify, or Yubal added.

1. **Download** music with Downtify (Spotify) or Yubal (YouTube Music), or let Explo discover and fetch new tracks automatically.
2. **Stream** everything through Navidrome.
3. **Listen** on your phone with Substreamer.

