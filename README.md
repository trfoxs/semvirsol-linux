# 🛡️ SemvirSol — Heuristical Antivirus Solution | v0.1.0 Linux Beta

**Copyright © 2026 Semu Design. All rights reserved.**

SemvirSol is a proprietary Linux Beta file-scanning tool that combines static heuristic indicators with integrity checks for supported web frameworks. It analyses file contents as data and does not execute the files it scans.

> **Closed-source binary release:** Source files are not distributed in this repository. The compiled Linux binary is published through GitHub Releases. This project is not open source; see [`LICENSE`](LICENSE).

## ✨ Features

- Static heuristic scanning for suspicious PHP, JavaScript and HTML-related code patterns.
- One-file scanning, recursive directory scanning and watch/daemon mode.
- Optional quarantine for one-off scans; watch mode follows the quarantine setting in `config`.
- Project detection and integrity checks for WordPress, Laravel, CodeIgniter, Symfony and OpenCart.
- Per-project detection cache. Each installation is keyed by its own normalised project root.
- Operational logs by default; optional detailed per-file logs with `-lgp`.

## File types covered by the current rule database

- **PHP:** `.php`, `.php5`, `.phtml`, `.inc`, `.phar`
- **JavaScript:** `.js`, `.cjs`, `.mjs`, `.jsx`
- **HTML / related web content:** `.html`, `.htm`, `.shtml`, `.svg`, `.xhtml`

Coverage depends on the database version included in a release. No scanner can guarantee detection of every malicious file.

## Integrity checks

- **WordPress:** core-file checksums are obtained from the official WordPress checksum API when available. A network connection may be required when checksum data is not already cached.
- **Laravel, CodeIgniter, Symfony and OpenCart:** the framework is detected automatically. Files require a trusted reference (for example, a valid `.semvir-integrity.json` SHA-256 manifest) to be reported as an integrity match.
- A trusted `match` skips heuristic scanning for that file. `modified` or `unknown` files proceed to heuristic scanning.
- `.semvir-integrity-cache.json` stores project-detection metadata, not per-file hash results. Entries are separated by project root, so multiple sites/framework installations do not share a project identity accidentally.

## Minimum system requirements

These are provisional targets for Semvir itself, not the complete web-hosting stack.

| Resource | Minimum target | Recommended |
|---|---|---|
| OS / architecture | 64-bit Linux, x86-64 / AMD64 | A Linux distribution compatible with the release binary and its runtime libraries |
| CPU | 1 vCPU | 2 vCPU |
| RAM | 1 GB | 1–2 GB for Semvir alone |
| Free disk | 100 MB plus database/log/quarantine space | Size according to workload and quarantine retention |

Measurements so far were made on Ubuntu under WSL2 and do not prove a guaranteed minimum on all servers. A host running multiple WordPress sites, web server processes, PHP-FPM and a database needs additional memory; size the complete server separately. ARM64 requires a matching build.

## Download and install

1. Open **Releases** and download the archive matching your Linux architecture, for example `semvirsol-linux-amd64-v0.1.0.tar.gz`.
2. Extract it to a dedicated directory, such as `/opt/semvir`.
3. Review `config` before running. Set `watch_paths` to the directories you intend to monitor, check the database path and review quarantine settings.
4. Ensure the account running Semvir can read the database and watched files and write to the log and quarantine directories.
5. Test on a non-production directory first.

Example:

```bash
mkdir -p /opt/semvir
sudo tar -xzf semvirsol-linux-v0.1.0.tar.gz -C /opt/semvir
cd /opt/semvir
chmod +x ./semvir-cli
./semvir-cli help
./semvir-cli version
```

Do not enable automatic quarantine on a production server until you have reviewed the rules and tested the configuration. In watch mode, detections may be quarantined automatically when `quarantine.enabled` is `true`.

## Commands

Run from the directory containing `config`, or pass `--config PATH`.

| Command | Description |
|---|---|
| `./semvir-cli help` | Show available commands. |
| `./semvir-cli version` | Show Semvir and database versions. |
| `./semvir-cli -s /path/to/file.php` | Scan one file and show per-file details. |
| `./semvir-cli -sa /path/to/site` | Recursively scan a directory with animated progress and a summary. |
| `./semvir-cli -s /path/file.php -r` | Scan a file and quarantine detections. `-r` works with both `-s` and `-sa`. |
| `./semvir-cli -sa /var/www/site -r` | Scan a directory and quarantine detections. |
| `sudo ./semvir-cli` | Run watch mode in the foreground using `watch_paths`. |
| `sudo ./semvir-cli -lgp semvir.log` | Run watch mode with detailed per-file logging to the given path. |
| `./semvir-cli -sa /var/www/site -lgp scan.log` | Scan a directory and write detailed scan logs to the given path. |
| `./semvir-cli -lgp` | Show the configured/default log path or initialise its default. |
| `sudo ./semvir-cli enable` | Install and enable the systemd service at boot. |
| `sudo ./semvir-cli start` / `stop` / `status` | Start, stop or inspect the systemd service. |
| `./semvir-cli --config /path/to/config help` | Use a specific configuration file. |

Service-management commands require systemd and normally require root privileges. Command-line details may change during Beta; verify with `./semvir-cli help` for the exact release.

## Whitelist and false positives

Semvir uses heuristic indicators based on suspicious code patterns and combinations of indicators. Legitimate custom code can sometimes trigger a detection. Review a finding before acting on it. If a trusted project or component repeatedly triggers a false positive, add only the reviewed file or directory to `whitelist`.

Each whitelist entry goes on its own line. Directory entries end with a trailing slash:

```text
# Exact trusted file
/var/www/example.test/custom-plugin/trusted-file.php

# Trusted directory (trailing slash)
/var/www/example.test/custom-plugin/approved-code/
```

Whitelisted paths are skipped by scanning. Do not whitelist entire website roots or writable upload directories unless you fully understand the security impact.

## Release contents and repository layout

This public repository contains documentation, examples and the proprietary licence. It does **not** contain Python source code.

```text
semvirsol/
├── README.md
├── LICENSE
├── CHANGELOG.md
├── .gitignore
├── config.example
├── whitelist.example
└── docs/
    └── RELEASE-GUIDE.md
```

Upload compiled distribution packages as assets attached to a GitHub Release rather than committing temporary build output. A release archive should contain:

```text
semvirsol-linux-v0.1.0/
└── db/semvir-rules.smdb   # db
    └── RELEASE-GUIDE.md
├── semvir-cli             # Compiled Linux binary
├── config                 # Generic config; review paths before running1
├── whitelist              # Blank or commented template
├── README.md
└── LICENSE
```

Publish the database only if you own or have permission to redistribute all rules and signatures it contains. Do not put machine-specific paths, logs, `.semvir-integrity-cache.json`, `.quarantine/`, virtual environments or crash reports into the release archive.

## Beta and update notice

SemvirSol is a **Linux Beta** and its heuristic database is still being expanded. Please check the latest release and database version regularly, follow the project for updates, and test new releases before deploying them on important systems. Keep backups and do not rely on SemvirSol as your only security layer.

## Licence

The compiled binary and Semvir rule database are proprietary. Personal non-commercial use is permitted only under the conditions in [`LICENSE`](LICENSE). Commercial use, redistribution, resale, modification and reverse engineering are restricted by that licence, subject to applicable law. For written permission or commercial licensing, contact **info@semu.work**.
