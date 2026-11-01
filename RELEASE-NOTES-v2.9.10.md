<!-- Suggested release title: PGPorter v2.9.10 — Recovery-First Migration Studio (Windows + Linux) -->
<!-- Suggested tag: v2.9.10 -->
<!-- Everything below this line is the release body: paste it into the release description field. Attach every asset listed in the table, including the .sha256 sidecars. -->

# PGPorter v2.9.10 — Windows & Linux

PGPorter 2.9.10 is the production-hardened cumulative release of the v2.9.8–v2.9.10 line: a faster, lighter Database Explorer, safer destructive workflows, hardened history clearing and smarter Windows tool setup — with the same recovery-first migration workflow on both platforms.

**Free to download and use for personal and commercial work.** No Python, Rust or build tools required — these are compiled release builds.

## Downloads

| Asset | Platform | Kind |
| --- | --- | --- |
| `PGPorter-Setup-2.9.10-x64.exe` | Windows 10 / 11, x64 | Installer |
| `PGPorter-Windows-v2.9.10-Release.zip` | Windows 10 / 11, x64 | Portable |
| `PGPorter-2.9.10-amd64.deb` | Linux x64 (Zorin OS, Ubuntu, Debian, Linux Mint) | Installer |
| `PGPorter-Linux-v2.9.10-Nuitka.tar.gz` | Linux x64 | Portable |

Each asset has a `.sha256` sidecar in portable `HASH  filename` format — see *Verify your download* below.

---

## 🪟 Windows

**Install (recommended):** run `PGPorter-Setup-2.9.10-x64.exe`. All-users installs go to Program Files; per-user installs to `%LOCALAPPDATA%\Programs\PGPorter`.

**Portable:** extract `PGPorter-Windows-v2.9.10-Release.zip` anywhere and run `PGPorter.exe`. First and later launches use a cached extraction folder, so start-up is faster than previous versions.

**First run — PostgreSQL tools:** PGPorter drives `psql`, `pg_dump` and `pg_restore` and does not bundle them. Use **Install Missing Tools** (Migration, Settings or Diagnostics) — it now installs *only* the command-line tools through winget: no database server, no Windows service, no pgAdmin, no default superuser password. Or use **Import Portable ZIP…** to load the EDB `postgresql-*-windows-x64-binaries.zip` with no installer and no administrator rights.

Runtime history is stored in `%LOCALAPPDATA%\PGPorter`, separate from the application folder.

## 🐧 Linux

**Install (.deb, recommended for Zorin OS, Ubuntu, Debian and Linux Mint):**

```bash
sudo apt install ./PGPorter-2.9.10-amd64.deb
```

**Portable:** extract the archive into its own folder and run the executable:

```bash
mkdir PGPorter && tar -xzf PGPorter-Linux-v2.9.10-Nuitka.tar.gz -C PGPorter
./PGPorter/PGPorter
```

**PostgreSQL tools:** install `postgresql-client` from your package manager, or use **Install Missing Tools** in-app. Preflight reports the source server's major version and blocks `pg_dump` older than the source; the in-app installer can target the required client major (for example `postgresql-client-18` for current Render compatibility).

Runtime history is stored in `~/.local/share/pgporter`.

Both platforms share the same migration, backup, explorer, history and diagnostics workflow.

## 🔐 Verify your download

```bash
sha256sum -c PGPorter-Windows-v2.9.10-Release.zip.sha256
sha256sum -c PGPorter-Setup-2.9.10-x64.exe.sha256
sha256sum -c PGPorter-Linux-v2.9.10-Nuitka.tar.gz.sha256
```

On Windows (PowerShell), compare manually:

```powershell
Get-FileHash .\PGPorter-Setup-2.9.10-x64.exe -Algorithm SHA256
```

## ✨ What's new in 2.9.10

### Performance & memory

- **Explorer:** fixed browsing tables containing a single value larger than 128 KB ("Could not parse PostgreSQL CSV output"); large pages use up to **~4–5× less RAM** while parsing; pages over 64 MB are refused with an actionable message.
- Showing a huge text/JSON cell no longer copies the whole value three times; formatted cell text and column widths are cached per page, so filtering is instant.
- The on-screen log is bounded (20,000 lines) and drains faster, so very verbose restores no longer grow memory without limit.
- Faster second and later launches for the portable Windows build (cached extraction folder).

### Database Explorer

- Redesigned grid: light/dark themes, zebra rows, content-sized columns with horizontal scroll (headers are no longer cut off), right-aligned numbers.
- **Double-click any cell** to see its full value — long text, Markdown and JSON are no longer truncated. Right-click to copy the cell or the whole row.
- **New Edit mode** (off by default, resets whenever you connect or switch Source/Destination): double-click a cell, change it, save. Every save is one atomic statement that locates the row by primary key — or by every column when the table has none — and only applies if the cell still holds the value you were shown, so someone else's change is never overwritten. Views are never editable, and identical duplicate rows in a table without a primary key are refused. Browsing itself stays read-only.

### Security hardening

- Plain-SQL safety (CVE-2025-8714): plain `.sql` export is blocked when `pg_dump` is older than 13.22 / 14.19 / 15.14 / 16.10 / 17.6; `.sql` import scans for psql meta-commands (`\!`, `\i`, `\o`, …) and requires explicit confirmation.
- Connection URLs now reject `password`, `host`, `hostaddr`, `port`, `dbname`, `user`, `service` and `passfile` query parameters (they could override the displayed target or leak secrets).
- `sslmode=disable/allow/prefer` is rejected for non-local hosts; external connections require TLS.
- Destructive restores fail closed when the same-database identity check cannot be completed; non-rollback restores run with `--single-transaction`.
- Backups, partial dumps and CSV exports are created with owner-only permissions (0600, backup folder 0700).
- Cancellation now escalates to a process-group kill after a 10-second grace period.

### Privacy & history

- **Clear History** is production-hardened: deletion is refused during active recovery or while any migration is still marked running, and GUI, CLI and helper entry points share one authoritative safety gate. Clear History still preserves physical backup files, saved profiles and settings.

### Managed PostgreSQL (Render and similar)

- Preserves the full password-free libpq connection URI for `psql`, `pg_dump` and `pg_restore`, the original hostname for TLS SNI, and forwards every libpq option — including ones PGPorter does not explicitly know about.
- Rejects insecure `sslmode` on detected Render external hostnames and adds Render-specific guidance for SNI, TLS, IP allowlists, authentication and timeout failures.
- Full automatic migration blocks PostgreSQL major-version downgrades (for example 18 → 17) instead of attempting an unsafe restore.

### Fixed

- Crash after every "Export Source SQL" (the UI never returned to idle).
- Migrations could hang when `pg_dump`/`pg_restore` emitted localized or non-UTF-8 output.
- Explorer failing on Windows with `invalid byte sequence for encoding "UTF8": 0x97`; non-ASCII table, schema and column names now work on all platforms.
- Linux/Tk crash when a global mouse-wheel event reported a raw widget-path string (CustomTkinter 5.3.0).

## 📋 Notes

- **Requirements:** Windows 10/11 x64, or a x64 Linux desktop with a `.deb`-based distribution. PostgreSQL client tools (`psql`, `pg_dump`, `pg_restore`) are required and not bundled.
- **Your data stays local.** PGPorter connects only to the databases you configure; passwords are never stored on disk or sent anywhere; migration history stays in your user profile.
- **Upgrade-friendly:** previous settings, profiles, history and backups are preserved. Clear History remains available if you want a clean slate.
- **Proprietary notice:** PGPorter is free to use but not open source. PostgreSQL and other third-party components keep their own licenses.

---

If PGPorter saves you time, [support the developer on Ko-fi](https://ko-fi.com/edwarddelacruz) — donations are voluntary and never required.

**Website:** <https://codedev-by-edward.myportfoliohub.online/> · **MyPortfolioHub:** <https://www.myportfoliohub.online/>

PGPorter © 2026 John Edward Dela Cruz · CodeDev by Edward. All rights reserved.
