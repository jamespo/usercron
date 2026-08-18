# usercron

Manage user cron entries like `/etc/cron.d`.

`usercron` lets you keep your personal crontab as a directory of separate
`*.cron` files (one per job, or grouped however you like) instead of a
single monolithic file edited via `crontab -e`. Each file is validated
before install, so a typo in one job can't break the rest of your crontab.

> **Note:** usercron itself is hand-written, not AI-generated. This README,
> however, was written with AI assistance.

## How it works

Cron entry files live in `~/.config/usercron/` and must end in `.cron`.
Running `usercron install` (or `usercron edit`, which installs
automatically) does the following:

1. Finds every `*.cron` file in `~/.config/usercron/`.
2. Validates each file with the system's crontab syntax checker.
3. Concatenates the valid files, wrapped in `### USERCRON start ###` /
   `### USERCRON end ###` markers, and installs them as your crontab.
4. Any existing crontab content outside those markers (i.e. entries you
   manage by hand) is preserved; only the previous USERCRON block is
   replaced.

Invalid files are reported and skipped (or, when editing, deleted) so a
bad file never gets loaded into your crontab.

## Usage

```
usercron (help|install|edit <cronfile>)
```

### Create or edit a cron job

```
$ usercron edit backups.cron
```

Opens `~/.config/usercron/backups.cron` in `$EDITOR` (falls back to `vi`).
On save, the file is syntax-checked:

- If valid, it's installed immediately into your crontab.
- If invalid, the error is printed and the file is deleted.

Example contents of `backups.cron`:

```
0 2 * * * /home/james/bin/backup.sh
```

### Install/refresh all cron entries

```
$ usercron install
crontab installed for james with 3 entries from /home/james/.config/usercron
```

Re-scans `~/.config/usercron/*.cron` and rebuilds the crontab. Useful after
manually adding, removing, or editing files in that directory outside of
`usercron edit`.

### Help

```
$ usercron help
Usage: usercron (help|install|edit <cronfile>)
```

## Dependencies

- **bash**
- **crontab** (from a cron implementation providing `crontab -l`/`crontab
  <file>`, e.g. Debian cron, cronie, or Vixie cron). Syntax validation uses
  a distro-specific dry-run flag (`crontab -n` on Debian, `crontab -T` on
  Arch/cronie and RHEL-family systems newer than version 8); on systems
  without a supported flag, validation is skipped.
