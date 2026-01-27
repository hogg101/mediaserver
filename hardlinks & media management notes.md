# Radarr/Sonarr + qBittorrent (Docker) hardlinks + cleanup notes

## Goals
1. Use **hardlinks** so imported media in the library does not duplicate disk usage.
2. Keep **downloads** and **library** separate to avoid Radarr/Sonarr warnings and data loss.
3. Clean up torrents/downloads after seeding, without breaking imports.

---

## Folder layout (host)
Everything on the same filesystem (hardlinks require this):

- `/mnt/ext_ssd/media/downloads`
- `/mnt/ext_ssd/media/movies`
- `/mnt/ext_ssd/media/tv`

Key rule: **downloads folder must not be inside a library root folder** configured in Radarr/Sonarr.

---

## Docker path strategy
To make hardlinks reliable, keep **one shared mount** and consistent paths across containers:

Mount:
- Host: `/mnt/ext_ssd/media`
- Container path (all apps): `/data`

So inside containers:
- Downloads: `/data/downloads`
- Movies: `/data/movies`
- TV: `/data/tv`

This avoids path mismatches and makes hardlinks/atomic moves work consistently.

---

## qBittorrent settings
- Default save path: `/data/downloads`
- Optional incomplete folder: `/data/downloads/incomp`
- Ensure qB is **not moving completed downloads into the library**:
  - No “Move completed downloads to” pointing at `/data/movies` or `/data/tv`
  - No category save paths pointing at `/data/movies` or `/data/tv`
  - Existing torrents should not have Save Path set to the library folders

### Seeding stop settings (minutes)
If you want:
- stop after **ratio 1.2** OR
- stop after **7 days seeding** OR
- stop after **24h inactive (stalled) seeding**

Set:
- Ratio: `1.2`
- Total seeding time: `10080` minutes (7 days)
- Inactive seeding time: `1440` minutes (1 day)
- Action: `Stop torrent`

---

## Radarr/Sonarr settings
### Hardlinks
Enable:
- `Settings -> Media Management -> (Advanced) -> Importing -> Use Hardlinks instead of Copy`

### Root folders
- Radarr root: `/data/movies`
- Sonarr root: `/data/tv`
Avoid adding `/data` or `/data/downloads` as a root folder.

### Download client paths
- qBittorrent should report paths under `/data/downloads/...`
- If paths don’t match between qB and *arr, you may need Remote Path Mapping, but with a shared `/data` mount you typically do not.

---

## Radarr warning: "Downloading into Root Folder"
We saw the warning once after changing paths around, then it disappeared after a stack reboot.
Likely due to stale download client/path state while reconfiguring.

Still, the correct prevention is:
- qB downloads must stay in `/data/downloads`
- Radarr root must be `/data/movies` only
- Ensure no qB setting/category/torrent location points into `/data/movies` or `/data/tv`

---

## How to test whether import used hardlinks (not copies)
Use inode + link count.

Example (host paths):
```bash
stat -c '%i %h %n' \
  "/mnt/ext_ssd/media/downloads/<file>.mkv" \
  "/mnt/ext_ssd/media/movies/<movie-folder>/<file>.mkv"