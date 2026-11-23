# minecraft-backup

Backing up a live Minecraft world requires coordination with the server so autosaves do not change world files during the copy. This Bash script handles Minecraft server backups on Linux through screen, tmux, RCON, or Docker RCON, with tar archives or a restic repository as the destination.

## Features

- Back up world directories, with repeated `-i` arguments for multiple worlds.
- Disable world autosaves and request a save before the backup, then re-enable autosaves afterward.
- Write compressed tar archives or send backups to a restic repository.
- Manage old backups with `thin`, `sequential`, or `none` deletion policies. The default thinning policy keeps the last 24 hourly and 30 daily backups, then uses remaining capacity for weekly backups.
- Send backup status messages to Minecraft chat with `-c`, without a server plugin.
- Coordinate with servers through screen, tmux, RCON, or the RCON CLI in an `itzg/docker-minecraft-server` container.

## Installation

From a checkout of this project's source, install the shell script under the command name `minecraft-backup`:

```bash
mkdir -p "$HOME/.local/bin"
install -m 755 backup.sh "$HOME/.local/bin/minecraft-backup"
export PATH="$HOME/.local/bin:$PATH"
```

Add the PATH setting to your shell configuration if that directory is not already on your PATH.

For tar backups, install `tar` and your chosen compression program; the default is `gzip`. RCON also requires `xxd`. Install `screen`, `tmux`, `restic`, or Docker as needed for your chosen connection method and backup backend.

## Usage

Enable RCON on your Minecraft server before using the RCON connection method. Replace the example password and paths with your own:

```bash
minecraft-backup -c -i /home/user/server/world -o /mnt/storage/backups -s localhost:25575:secret -w rcon
```

This writes a backup of the world directory, prints status messages in chat, and uses the default thinning policy. Tar backups use gzip compression by default, with a default maximum of 128 backups.

For a server in a screen session named `minecraft`:

```bash
minecraft-backup -c -i /home/user/server/world -o /mnt/storage/backups -s minecraft
```

For a server in tmux session `0`:

```bash
minecraft-backup -c -i /home/user/server/world -o /mnt/storage/backups -s 0 -w tmux
```

To use restic instead of tar, supply a repository with `-r`:

```bash
export RESTIC_PASSWORD="restic-pass-secret"
minecraft-backup -c -i /home/user/server/world -r /mnt/storage/backups-restic -s localhost:25575:secret -w rcon
```

Keep credentials private. Avoid committing passwords or exposing RCON to untrusted networks.

Use `-d sequential` to delete the oldest backup first, or `-d none` to disable deletion. Run `minecraft-backup -h` for the full option list.

### Schedule backups

Edit your crontab with `crontab -e`. This example runs hourly and enables the optional lock file with `-t`:

```cron
0 * * * * /home/user/.local/bin/minecraft-backup -t -c -i /home/user/server/world -o /mnt/storage/backups -s minecraft
```

Make sure the cron environment can find the required commands and has permission to read the world and write backups.

### Restore and check

**Test a restore before you need one.** Extract a gzip-compressed tar backup into a separate directory:

```bash
mkdir restored-world
cd restored-world
tar -xzvf /path/to/backups/2019-04-09_02-15-01.tar.gz
```

Inspect the extracted paths; the world may be nested under parent directories. Stop the Minecraft server before replacing its world directory. For restic backups, use `restic restore`.

**Live backups can still contain inconsistent player data.** Player saves can continue after `save-off`, so inventory and world contents may not match. The script requests saves and a disk flush, but it cannot eliminate this risk without freezing players. Screen and tmux connections wait five seconds by default after the flush request; `-z` changes that delay.

## License

MIT. This project is based on [nicolaschan/minecraft-backup](https://github.com/nicolaschan/minecraft-backup). Retain the upstream copyright and MIT permission notice in copies or substantial portions of the software.

## Related projects

The supplied related-project list contains these repositories. Their purposes were not provided, so no functionality is claimed here.

- [yashu1230203-hash/dark](https://github.com/yashu1230203-hash/dark): purpose not specified in the supplied material.
- [houdinetsounds/claude-config-editor-eqhg](https://github.com/houdinetsounds/claude-config-editor-eqhg): purpose not specified in the supplied material.
- [maxwildcat/vulcan-sql](https://github.com/maxwildcat/vulcan-sql): purpose not specified in the supplied material.

