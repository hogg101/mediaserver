# Media Server Migration Notes

## Goal
Move the media stack config to the new busybee host, add Jellyfin, and ensure hardlinking works between downloads and media on the external SSD.

## Key Findings (Hardlinking)
- Hardlinks require source and destination on the same filesystem.
- All containers must see identical absolute paths for downloads and media.
- Any download "incomplete" directory must also live on the same filesystem as the final media path.
- Permissions must match the container user (PUID/PGID 1000:1000).

## Storage Layout (New Host)
- Media: `/mnt/ext_ssd/media`
- Downloads: `/mnt/ext_ssd/media/downloads`
- Jellyfin cache: `/srv/docker/mediaserver/cache/jellyfin`
- Compose and config repo files: `/home/james/docker`

## Compose Changes Applied
- Replaced `/Volumes/Media` with `/mnt/ext_ssd/media` for all services.
- Replaced `./downloads` with `/mnt/ext_ssd/media/downloads` for all services.
- Kept all `/config` volume mounts under the repo `./config` (git-backed).
- Added Jellyfin (linuxserver) with `/config`, `/cache`, and `/media` mounts.
- Enabled iGPU transcoding for Jellyfin via `/dev/dri` device mapping and `group_add: 1000`.

## Open Items / Next Steps
- Confirm qBittorrent "incomplete" download path is on `/mnt/ext_ssd/media/downloads`.
- Confirm permissions on `/mnt/ext_ssd` and `/srv/docker/mediaserver` are owned by UID/GID 1000.
- If Jellyfin HW transcode fails, add the host user to the `video`/`render` groups or set `group_add` in compose.
