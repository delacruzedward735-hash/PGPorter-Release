# PGPorter v1.0.0 — Windows & Linux

PGPorter 1.0.0 is the first stable release of the recovery-first migration workflow: new standalone backups, the v7 security-audit fix round across the plain-SQL import path, Windows file permissions and tool imports that finally fail closed, smarter connection security and a hash-pinned build supply chain — with the same migration, Explorer, history and diagnostics experience on both platforms.

**Free to download and use for personal and commercial work.** No Python, Rust or build tools required — these are compiled release builds.

## Downloads

| Asset | Platform | Kind |
| --- | --- | --- |
| `PGPorter-Setup-1.0.0-x64.exe` | Windows 10 / 11, x64 | Installer |
| `PGPorter-Windows-v1.0.0-Release.zip` | Windows 10 / 11, x64 | Portable |
| `PGPorter-1.0.0-amd64.deb` | Linux x64 (Zorin OS, Ubuntu, Debian, Linux Mint) | Installer |
| `PGPorter-Linux-v1.0.0-Nuitka.tar.gz` | Linux x64 | Portable |

Each asset has a `.sha256` sidecar in portable `HASH  filename` format — see *Verify your download* below.

---

## 🪟 Windows

**Install (recommended):** run `PGPorter-Setup-1.0.0-x64.exe`. All-users installs go to Program Files; per-user installs to `%LOCALAPPDATA%\Programs\PGPorter`.

**Portable:** extract `PGPorter-Windows-v1.0.0-Release.zip` anywhere and run `PGPorter.exe`. First and later launches use a cached extraction folder, so start-up is faster than previous versions.

**First run — PostgreSQL tools:** PGPorter drives `psql`, `pg_dump` and `pg_restore` and does not bundle them. Use **Install Missing Tools** (Migration, Settings or Diagnostics) — it installs *only* the command-line tools through winget: no database server, no Windows service, no pgAdmin, no default superuser password. Or use **Import Portable ZIP…** to load the EDB `postgresql-*-windows-x64-binaries.zip` with no installer and no administrator rights. Imported ZIPs and folders are now hash-checked and Authenticode-verified *before* anything is executed (see *Security audit fixes* below).

Runtime history is stored in `%LOCALAPPDATA%\PGPorter`, separate from the application folder.

## 🐧 Linux

**Install (.deb, recommended for Zorin OS, Ubuntu, Debian and Linux Mint):**

```bash
sudo apt install ./PGPorter-1.0.0-amd64.deb
```

**Portable:** extract the archive into its own folder and run the executable:

```bash
mkdir PGPorter && tar -xzf PGPorter-Linux-v1.0.0-Nuitka.tar.gz -C PGPorter
./PGPorter/PGPorter
```

**PostgreSQL tools:** install `postgresql-client` from your package manager, or use **Install Missing Tools** in-app. Preflight reports the source server's major version and blocks `pg_dump` older than the source; the in-app installer can target the required client major (for example `postgresql-client-18` for current Render compatibility).

Runtime history is stored in `~/.local/share/pgporter`.

Both platforms share the same migration, backup, explorer, history and diagnostics workflow.

## 🔐 Verify your download

```bash
sha256sum -c PGPorter-Windows-v1.0.0-Release.zip.sha256
sha256sum -c PGPorter-Setup-1.0.0-x64.exe.sha256
sha256sum -c PGPorter-Linux-v1.0.0-Nuitka.tar.gz.sha256
```

On Windows (PowerShell), compare manually:

```powershell
Get-FileHash .\PGPorter-Setup-1.0.0-x64.exe -Algorithm SHA256
```

## ✨ What's new in 1.0.0

### Standalone backups (new)

- **Back Up Source Now** (Backups tab) or the new `--backup` CLI: tests connectivity, checks free disk space, writes an atomic dump, validates the archive with `pg_restore --list`, records SHA-256 (plus a `.sha256` sidecar) and registers the backup in history. `--keep N` keeps only the newest *N* manual backups; `--backup-format plain` produces `.sql` instead of `.dump`.
- The CLI reads the connection URL from an environment variable (`PGPORTER_BACKUP_URL` by default), so passwords never appear in argv or shell history. Exit codes: `0` success, `1` failure, `4` missing URL variable.
- Restoring a saved backup from the Backups tab is transactional when no safety snapshot exists, so a failed restore no longer leaves a half-restored database.

```bash
export PGPORTER_BACKUP_URL='postgresql://user:pass@host:5432/dbname?sslmode=require'
python main.py --backup --backup-dir /srv/pg-backups --keep 14   # add --backup-format plain for .sql
```

### Security audit fixes

- **Plain-SQL import safety (CVE-2025-8714 class):** the psql meta-command scan was bypassable — a `SELECT 1; \! cmd` written mid-line, or a fake `COPY … FROM stdin` hidden inside a string literal, slipped past the old scanner. It is now a real lexer that follows psql's rules (string literals, dollar quotes, nested comments, `standard_conforming_strings`, COPY data blocks) and reports *any* backslash command outside them.
- **A second bypass class, found in the release audit, is closed:** comments and quotes inside a COPY header no longer hide the data block (`COPY t FROM/**/STDIN` and `COPY"t"FROM STDIN` are now recognised), and only a real `COPY [BINARY] <table> [(cols)] FROM STDIN` switches the scanner to data mode — a table named `stdin` no longer makes it skip lines psql would execute.
- Every plain `.sql` restore — GUI, Rust core and Python fallback — is scanned and **refused by default**; only the SQL-import dialog can allow meta-commands, after you confirm the warning. The file is re-scanned immediately before it runs, not just at confirmation time.
- **"Export Source → .sql"** is now atomic and owner-only: the export is published only on success, no partial file survives a failure or cancellation, and the file is no longer world-readable on Linux.
- Quoted / multi-word `password='my secret'` values are now masked in full in all output (Python and Rust).
- The Rust execution core protocol was bumped: arguments are passed as `--key=value`, so a value starting with `--` can no longer be mistaken for a flag. An older core binary is simply unavailable (PGPorter falls back to the Python backend) instead of misparsing arguments.

### Windows file & tool protection

- Owner-only file protection now actually works on Windows: dumps (including the partial file before data is written), `.sha256` sidecars, `pgporter.db` and the backup folders PGPorter creates receive an owner + SYSTEM ACL via `icacls` — previously `chmod` was silently ignored on Windows. Pre-existing user folders are not modified.
- Table CSV export, history CSV export and saved logs are created owner-only *before* any data is written; an existing looser file is tightened instead of keeping its old permissions. Table export streams through a private partial file and leaves nothing behind on failure.
- **CSV formula injection is neutralised:** values starting with `=`, `+`, `-`, `@`, tab or CR get a leading apostrophe so spreadsheet applications cannot execute them. Plain numbers and the `\N` NULL marker are untouched, and a byte-exact export remains available for advanced use.
- **Imported PostgreSQL tools are verified before anything runs:** ZIP or folder imports (and the manual bin-folder picker) are hashed and Authenticode-checked. Files without a valid EnterpriseDB/PostgreSQL signature are refused unless you explicitly accept, with the ZIP and executable SHA-256 shown so you can compare them against the official download page. The check fails closed, and ZIP extraction enforces the real byte budget instead of the spoofable sizes declared in ZIP headers.

### Connection security

- The migration confirmation now warns when a remote host uses `sslmode=require`, `prefer` or `allow` — encrypted, but the certificate is **not** verified.
- The connection dialog gains **verify-full (system CAs)** (`sslmode=verify-full` + `sslrootcert=system`; requires PostgreSQL client tools 16+).
- **"Export Source → .sql"** is now atomic and owner-only: the export is published only on success, no partial file survives a failure or cancellation, and the file is no longer world-readable on Linux.

### Supply chain & build integrity

- GitHub Actions are pinned to full commit SHAs and Inno Setup is pinned to a fixed version; all Python dependencies are hash-pinned in `requirements.lock` / `requirements-build.lock` and installed with `pip install --require-hashes` by CI, the installers and the build scripts. `pip-audit` audits the lock file.
- Fixed the Windows source build failing at the C-compile stage (`cc1.exe: fatal error: module.PIL.XVThumbImagePlugin.c`): Nuitka no longer compiles 39 unused Pillow image-format plugins (PNG, BMP, GIF, ICO, JPEG, PPM, TIFF and others stay), which also shortens the build. One automatic clean retry covers vanished or antivirus-locked generated files, and the build warns when the project sits inside OneDrive.
- Release hygiene: no prebuilt Rust core ships in the source package, `Cargo.lock` is tracked and authoritative, the proprietary `LICENSE` is the single license file, and CI now runs the end-to-end migration tests — including a backup round trip — against a real PostgreSQL 16 server.

## 📋 Notes

- **Requirements:** Windows 10/11 x64, or a x64 Linux desktop with a `.deb`-based distribution. PostgreSQL client tools (`psql`, `pg_dump`, `pg_restore`) are required and not bundled.
- **Your data stays local.** PGPorter connects only to the databases you configure; passwords are never stored on disk or sent anywhere; migration history stays in your user profile.
- **Upgrade-friendly:** previous settings, profiles, history and backups are preserved. Clear History remains available if you want a clean slate.
- **Proprietary notice:** PGPorter is free to use but not open source. PostgreSQL and other third-party components keep their own licenses.

---

If PGPorter saves you time, [support the developer on Ko-fi](https://ko-fi.com/edwarddelacruz) — donations are voluntary and never required.

**Website:** [CodeDev by Edward](https://codedev-by-edward.myportfoliohub.online/) · **MyPortfolioHub:** [myportfoliohub.online](https://www.myportfoliohub.online/)

PGPorter © 2026 John Edward Dela Cruz · CodeDev by Edward. All rights reserved.
