# Pterodactyl Minecraft Multi-Profile Manager

Run many independent Minecraft server folders from **one Pterodactyl server and one allocation**, while only starting one profile at a time.

This project is designed for home servers and small private hosts that keep multiple Minecraft worlds/modpacks but do not want to reserve a separate Pterodactyl allocation for every stored server.

> Not affiliated with or endorsed by Pterodactyl Software.

## Why this exists

A normal Pterodactyl server requires a primary allocation even while the server is stopped. If you keep many old worlds, modpacks, test servers, and friend servers as separate Pterodactyl servers, the allocation list grows with them.

This project instead stores them as profiles inside one Pterodactyl server:

```text
/home/container/
├─ servers/
│  ├─ survival/
│  ├─ modded/
│  ├─ hardcore/
│  └─ friends/
├─ profile-backups/
└─ .multijava/
```

Only the selected profile runs, and every profile uses the same Pterodactyl primary allocation.

## Features

- One Pterodactyl server, one primary allocation
- Multiple independent Minecraft server folders
- `profile.conf` generated automatically when missing
- Server type auto-detection
- Minecraft version detection where possible
- Automatic Java runtime selection
- Per-profile configuration overrides
- Safe handling of invalid or missing profiles
- Per-profile backups when switching
- Primary allocation port automatically applied to `server.properties`
- Optional dynamic `SERVER_PROFILE` dropdown in the Pterodactyl Startup page

## Supported server layouts

The launcher is intended to work with common layouts including:

- Paper
- Purpur
- Folia
- Spigot
- Vanilla
- Fabric
- Quilt
- Forge
- NeoForge
- Mohist
- Magma
- Arclight

Auto-detection is intentionally conservative. If the launcher cannot determine a Minecraft or Java version safely, it stops and asks you to set the value in `profile.conf` instead of guessing.

## Installation

### 1. Import the Egg

Import:

```text
egg/egg-minecraft-multi-profile-manager-v2.1.json
```

from the Pterodactyl admin panel under your Minecraft nest.

Create one server using the Egg and assign a single primary allocation, for example `25565`.

Set `ACCEPT_EULA=true` only if you accept the Minecraft EULA.

### 2. Add profiles

Place each existing Minecraft server inside its own directory:

```text
/home/container/servers/
├─ survival/
│  ├─ server.jar
│  ├─ server.properties
│  ├─ world/
│  └─ plugins/
└─ modded/
   ├─ run.sh
   ├─ libraries/
   ├─ mods/
   └─ world/
```

Then select the folder name as `SERVER_PROFILE` and start the Pterodactyl server.

## `profile.conf`

If a profile does not contain `profile.conf`, the manager creates one automatically.

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

Normally the `auto` values can be left unchanged. Override only what cannot be detected reliably.

## Automatic Java selection

The manager can select a Java runtime based on the detected or configured Minecraft version. Supported runtime slots include Java 8, 11, 16, 17, 21, and 25.

Downloaded runtimes are stored under:

```text
/home/container/.multijava/
```

and reused on later starts.

If the required Java version cannot be determined safely, startup stops instead of guessing. Set `MINECRAFT_VERSION` or `JAVA_VERSION` in that profile's `profile.conf` to resolve it.

## Profile switching and backups

When you change `SERVER_PROFILE`, the manager can back up the previously active profile before starting the new one.

Backups are stored per profile:

```text
/home/container/profile-backups/
├─ survival/
├─ modded/
└─ hardcore/
```

These are profile-level `.tar.gz` archives and are separate from Pterodactyl's built-in Backups tab.

## Dynamic Startup dropdown (optional)

The `panel-patch/` directory contains an optional Pterodactyl Panel frontend patch.

Instead of typing:

```text
SERVER_PROFILE=survival
```

it reads `/servers` using Pterodactyl's existing file-list API and renders the available directories as a dropdown in Startup.

Example:

```text
/servers/
├─ survival/
├─ modded/
├─ hardcore/
└─ friends/
```

becomes:

```text
SERVER_PROFILE
[ survival ▼ ]

survival
modded
hardcore
friends
```

The list refreshes when the Startup page loads and when the dropdown is focused.

### Install the Panel patch

On the machine hosting the Pterodactyl Panel:

```bash
cd panel-patch
sudo bash install.sh /var/www/pterodactyl

cd /var/www/pterodactyl
yarn build:production
```

Then hard-refresh the browser.

The patch only changes the Panel frontend. It does not modify Wings or add a new backend API.

> Pterodactyl Panel updates can overwrite frontend source changes. Re-apply the patch after an update if the dropdown disappears.

## Port behavior

Every selected profile is forced to use the current Pterodactyl primary allocation.

For a primary allocation of `25565`, the manager updates the selected profile's `server.properties` to use that port before launch.

This means you can keep one external forwarding rule such as:

```text
WAN :25565 → Pterodactyl node :25565
```

while storing many server profiles behind it.

## Important limitations

- Only one profile can run at a time.
- All profiles share the resource limits of the single Pterodactyl server.
- Auto-detection cannot identify every custom or heavily modified server layout.
- The dynamic dropdown requires a Pterodactyl Panel source modification and frontend rebuild.
- Profile backups are filesystem archives, not Pterodactyl-native backup records.
- Always keep separate backups of important worlds before testing server or Java version changes.

## Project structure

```text
.
├─ egg/
│  └─ egg-minecraft-multi-profile-manager-v2.1.json
├─ panel-patch/
│  ├─ DynamicProfileSelect.tsx
│  ├─ install.sh
│  ├─ patch_variablebox.py
│  └─ uninstall.sh
├─ docs/
│  └─ MIGRATION-v1-to-v2.md
├─ CONTRIBUTING.md
├─ LICENSE
├─ README.md
└─ SECURITY.md
```

## License

MIT. See [LICENSE](LICENSE).
