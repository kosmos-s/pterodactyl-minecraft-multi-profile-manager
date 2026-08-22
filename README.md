# Pterodactyl Minecraft Multi-Profile Manager v2.2.0

> 한국어 문서: https://github.com/kosmos-s/pterodactyl-minecraft-multi-profile-manager-ko

Store multiple independent Minecraft server folders inside **one Pterodactyl server with one Primary Allocation**, and run exactly one profile at a time.

This is aimed at home servers and small private hosts that keep many worlds, modpacks, test servers, or archived servers without wanting a separate Pterodactyl allocation for every stored instance.

> Not affiliated with or endorsed by Pterodactyl Software.

## Features

- One Pterodactyl server + one Primary Allocation
- Multiple independent Minecraft server folders under `/home/container/servers/`
- Unicode profile names, including Korean
- Automatic `profile.conf` creation
- Common server-family detection for Paper, Purpur, Folia, Spigot, Vanilla, Fabric, Quilt, Forge, NeoForge, and more
- Minecraft version detection using `logs/latest.log`, `versions/`, jar names and common layouts
- Automatic Java 8 / 11 / 16 / 17 / 21 / 25 selection and caching
- Correct Pterodactyl `Memory=0` (Unlimited) handling
- Per-profile configuration overrides
- Automatic backup of the previously active profile when switching
- Backup retention with `BACKUP_KEEP`
- Primary Allocation port automatically applied to `server.properties`
- Conservative validation: ambiguous or invalid profiles stop instead of guessing

## Layout

```text
/home/container/
├─ servers/
│  ├─ default/
│  ├─ survival/
│  ├─ modded/
│  └─ old-server/
├─ profile-backups/
├─ .multijava/
├─ .multi-profile-state/
└─ profile-launcher.sh
```

## Installation

1. Import `egg/egg-minecraft-multi-profile-manager-v2.2.0.json` into a Minecraft Nest in the Pterodactyl admin panel.
2. Create a new server using this Egg.
3. Assign one Primary Allocation, for example `25565`.
4. Set `ACCEPT_EULA=true` only if you accept the Minecraft EULA.
5. Start the server.

## Switching profiles

The normal workflow is intentionally simple:

```text
Stop server
→ Startup
→ change SERVER_PROFILE
→ Start server
```

Example:

```text
SERVER_PROFILE=survival
```

runs:

```text
/home/container/servers/survival/
```

Profile names may contain Unicode letters/numbers plus `.`, `_`, and `-`. Spaces and path-like values such as `/`, `\\`, and `..` are rejected for safety.

## Importing an existing server

Place the complete existing server inside a profile folder:

```text
servers/survival/
├─ server.jar
├─ server.properties
├─ logs/
├─ plugins/
├─ world/
└─ ...
```

If `profile.conf` does not exist, the manager creates it automatically on first launch.

Imported servers are intentionally handled conservatively. Even if the jar has been renamed to `server.jar`, an existing `logs/latest.log` such as:

```text
Loading Paper 1.21.11-69-... for Minecraft 1.21.11
```

can be used to detect:

```text
Server type      : paper
Minecraft version: 1.21.11
Java             : 21
```

If detection is not reliable, set the version manually in `profile.conf` rather than letting the launcher guess.

## `profile.conf`

Example:

```ini
NAME=Survival
SERVER_TYPE=auto
MINECRAFT_VERSION=auto
JAVA_VERSION=auto
LAUNCH_MODE=auto
SERVER_JAR=auto
SERVER_SCRIPT=run.sh
JAVA_ARGS=
XMS_MB=128
MEMORY_PERCENT=95
BACKUP_ON_SWITCH=true
BACKUP_KEEP=5
BACKUP_EXCLUDE_LOGS=true
```

Normally the `auto` values can stay unchanged. Override only values that cannot be detected safely.

Example:

```ini
MINECRAFT_VERSION=1.21.11
JAVA_VERSION=auto
```

## Automatic Java selection

The Egg uses the Pterodactyl Java 25 Yolks image as its base, but a profile may require another Java version.

The manager can select and cache Java 8, 11, 16, 17, 21, or 25 under:

```text
/home/container/.multijava/
```

Example:

```text
Minecraft 1.21.11 → Java 21
Minecraft 26.2    → Java 25
```

Cached runtimes are reused on later starts.

## Memory behavior

Pterodactyl uses `Memory=0` to mean Unlimited.

v2.2.0 handles that correctly:

```text
Memory=0
→ Xms=128M
→ Xmx=JVM auto
```

With an explicit 4096 MB limit and the default `MEMORY_PERCENT=95`:

```text
Memory=4096
→ Xmx≈3891M
```

For production use, an explicit Pterodactyl memory limit is recommended so one Minecraft profile cannot consume memory needed by other services on the host.

## Profile-switch backups

When switching profiles, the previously active profile can be archived automatically.

Example:

```text
mcs → default
```

creates a backup similar to:

```text
/home/container/profile-backups/mcs/
└─ mcs-YYYYMMDD-HHMMSS-1-switch.tar.gz
```

`BACKUP_KEEP=5` keeps the latest five backups per profile.

These are filesystem `.tar.gz` archives and are separate from Pterodactyl's built-in Backups tab.

## Large file uploads: SFTP + WinSCP

Pterodactyl's web File Manager may have a 100 MB upload limit depending on the installation.

For large worlds, server archives, and backups, the recommended Windows workflow is:

```text
Pterodactyl server
→ Settings
→ SFTP Details
→ Launch SFTP
→ WinSCP
```

WinSCP is convenient because it can register itself for `sftp://` links, allowing Pterodactyl's **Launch SFTP** button to open the server directly.

PowerShell `sftp` and `scp` also work if you prefer command-line tools.

## Important limitations

- Only one profile is intended to run at a time.
- All profiles share the same Pterodactyl server resource limits.
- Auto-detection cannot identify every custom or heavily modified server layout.
- Profile backups are separate from Pterodactyl-native backup records.
- Keep independent backups of important worlds before testing server, mod, or Java version changes.

## Tested in a real Pterodactyl environment

The v2.2.0 release was tested with:

- clean Egg startup
- `Memory=0` Unlimited handling
- Unicode/Korean profile names
- automatic `profile.conf` creation
- imported Paper server detection from `latest.log`
- Java 21 ↔ Java 25 automatic switching
- Primary Allocation port enforcement
- profile-switch backups and `BACKUP_KEEP`
- invalid/missing profile rejection
- real Minecraft client connections and profile round-trips

## AI assistance disclosure

This project was developed with assistance from OpenAI ChatGPT for design discussions, implementation, debugging, documentation, and release preparation. Final feature decisions, real Pterodactyl testing, distribution, and repository management are handled by the repository owner.

See [AI_ASSISTANCE.md](AI_ASSISTANCE.md) for additional details.

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

## License

MIT. See [LICENSE](LICENSE).
