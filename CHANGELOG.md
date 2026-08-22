# Changelog

## v2.2.0

- Added Unicode/Korean profile names.
- Added Minecraft version detection from modern Paper `latest.log` lines.
- Added server-family detection fallback from `latest.log`.
- Added `versions/` directory version detection fallback.
- Improved imported servers whose executable jar is named only `server.jar`.
- Fixed `SERVER_MEMORY=0` being converted into a 128 MB maximum Java heap.
- Preserved automatic Java 8/11/16/17/21/25 runtime selection and caching.
- Verified profile-switch backups and `BACKUP_KEEP` in a real Pterodactyl environment.
- Removed the experimental dynamic Startup dropdown / Panel patch.
- Documented Pterodactyl `Launch SFTP` + WinSCP as the recommended large-file workflow.
- Added explicit AI-assistance disclosure.
- Verified real Minecraft client connections and profile round-trips.
