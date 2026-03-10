# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains a single Bash script (`zpool_replace.sh`) for interactively replacing missing or degraded disks in ZFS pools on Linux (Proxmox VE, Ubuntu, Debian).

## Running the Script

```bash
chmod +x zpool_replace.sh
sudo ./zpool_replace.sh
```

The script requires root privileges to access ZFS pool data and disk info.

## Dependencies

- `zpool` (ZFS utilities)
- `smartctl` (smartmontools)
- `lsblk`
- Standard GNU utilities (`awk`, `sed`, `grep`)

## Script Architecture

The script is a single linear flow with 8 sequential steps:

1. **Resilver check** — Exits early if any pool scan shows "resilver in progress" (checked via `zpool status | grep "^  scan:"`)
2. **Degraded pool detection** — Iterates all pools, checks state != `ONLINE`
3. **Pool selection** — If multiple degraded pools, prompts user; otherwise auto-selects
4. **Missing disk detection** — Finds disk lines matching `ata-`/`scsi-` prefix with `REMOVED` status in `zpool status` output
5. **All-pool disk inventory** — Builds exclusion list of all disks in any pool by scanning `zpool status` for `ata-`/`scsi-` identifiers
6. **Candidate discovery** — Scans `/dev/disk/by-id/ata-*` and `/dev/disk/by-id/scsi-*`, excludes any in the pool inventory
7. **Candidate display** — Uses `smartctl` and `lsblk` to show model, serial, size for each candidate
8. **Replacement** — Runs `zpool replace <pool> <missing_id> <new_disk>` after double confirmation

## Key Design Notes

- Disk identifiers use the stable `/dev/disk/by-id/` paths (`ata-*`/`scsi-*` format), not `/dev/sdX` paths
- Partition suffixes (`-part1`, `-part2`, etc.) are stripped via `sed 's/-part.*//'` when comparing disk base IDs
- The script uses emoji output — requires a Nerd Font terminal (e.g., Terminess Nerd Font from nerdfonts.com) for full display
