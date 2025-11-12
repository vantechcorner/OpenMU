## Install OpenMU on Windows (PowerShell, no Visual Studio/Node/Docker)

This guide shows how to build and run the all‑in‑one OpenMU server on Windows 10+ using only the .NET SDK and PostgreSQL.

### Overview
- No Visual Studio required
- No NodeJS required
- No Docker required
- Uses PostgreSQL and .NET SDK 9

### Prerequisites
- PostgreSQL (Windows x64)
  - Download: https://www.enterprisedb.com/downloads/postgres-postgresql-downloads
  - During setup, choose and remember the postgres superuser password (default in repo config is `admin`).
- .NET SDK 9 (x64)
  - Download: https://dotnet.microsoft.com/en-us/download/dotnet/9.0
  - Or install via winget: `winget install Microsoft.DotNet.SDK.9`

### 1) Get the sources
- Download ZIP: https://github.com/MUnique/OpenMU/archive/refs/heads/master.zip
  - Unpack it, e.g., to `D:\Github\OpenMU`
  - Or clone: `git clone https://github.com/MUnique/OpenMU.git`

### 2) Configure database connection
Open and edit:
`src\Persistence\EntityFramework\ConnectionSettings.xml`

- Update only the first two connections (admin superuser) to match your PostgreSQL setup:
  - User Id: `postgres`
  - Password: the one you set during PostgreSQL installation

Example (only change the Password value):
```xml
<ConnectionString>Server=localhost;Port=5432;User Id=postgres;Password=YOUR_PASSWORD;Database=openmu;Command Timeout=120;</ConnectionString>
```

Leave the other users (`config`, `account`, `friend`, `guild`) as-is; the server initializes them on first run.

Optional (instead of editing XML): Use environment variables before starting the server:
- `DB_HOST` – database host name
- `DB_ADMIN_USER` – admin user (postgres)
- `DB_ADMIN_PW` – admin password

### 3) Build (publish) the server
Publishing is recommended to include all required web assets.

PowerShell (from repository root):
```powershell
dotnet publish .\src\Startup\MUnique.OpenMU.Startup.csproj -c Release -o .\ServerOpenMu
```

This creates `.\ServerOpenMu\` with `MUnique.OpenMU.Startup.exe`.

### 4) Run the server
From PowerShell:
```powershell
cd .\ServerOpenMu\
.\MUnique.OpenMU.Startup.exe -autostart -resolveIP:local
```

Notes:
- `-autostart` automatically starts all servers (connect, game, chat).
- `-resolveIP:local` resolves and advertises a local IP (good for LAN/same machine).
- First run may update or initialize the database. To force initialization (optional): add `-reinit`:
  ```powershell
  .\MUnique.OpenMU.Startup.exe -autostart -resolveIP:local -reinit
  ```
- Deprecated: the old `-deamon` parameter is no longer needed; console input behavior can be controlled in the Admin Panel (Configuration → System → Read Console Input).

### 5) Admin Panel
- The server logs the bound Admin Panel URL, typically `http://localhost:5000`.
- If you need a specific binding, set `ASPNETCORE_URLS`, e.g.:
  ```powershell
  $env:ASPNETCORE_URLS = "http://+:5000"
  .\MUnique.OpenMU.Startup.exe -autostart -resolveIP:local
  ```
- If started without `-autostart`, open Admin Panel → Server List to start servers.
- You can also set IP resolver and system options under Configuration → System.

### 6) IP resolver options
Use one of the following with `-resolveIP:`
- `public` – detect public IP automatically (default if nothing specified)
- `local` – choose a local LAN IP or fallback loopback (127.127.127.127)
- `loopback` – force loopback (127.127.127.127), best for same-machine client
- `<IPv4>` – custom IP, e.g. `-resolveIP:192.168.1.2`

Environment alternative: `RESOLVE_IP` (same values as above) if CLI flag omitted.

### 7) Firewall and ports
Ensure these TCP ports are allowed:
- 80 (Admin Panel; configurable)
- 44405–44406 (Connect Servers)
  - 44405: official client default
  - 44406: open-source client default
- 55901–55906 (Game Servers)
- 55980 (Chat Server)

### 8) Test accounts
Created automatically on initialization (Season 6 by default). Passwords equal the usernames.
- `test0` – `test9`: General test accounts, level 1..90 in steps of 10
- Season 6 only:
  - `test300`, `test400`
  - `testgm`, `testgm2`
  - `testunlock`
  - `quest1`, `quest2`, `quest3`
  - `ancient`, `socket`

You can also create accounts in the Admin Panel (Accounts → Create). Create characters in the game client for best results.

### 9) Client setup
- Official launcher binaries (requires .NET 9 runtime):
  - https://github.com/MUnique/OpenMU/releases/download/v0.9.0/MUnique.OpenMU.ClientLauncher_0.9.6.zip
- For same-machine testing, use any 127.x.x.x except 127.0.0.1 (client blocks it), e.g. 127.127.127.127 with `-resolveIP:loopback`.
- Third‑party client binaries/patches are at your own risk.

### Troubleshooting
- Database connection errors
  - Verify PostgreSQL is running and the superuser password matches your `ConnectionSettings.xml` (first two entries) or `DB_ADMIN_PW`.
  - Ensure `DB_HOST` is reachable (default `localhost`).
  - First-time initialization may take a bit; watch the server console/logs.
- Ports already in use
  - Close other servers using the same ports, or change bindings in Admin Panel → Configuration → System (and restart relevant servers).
- Admin Panel not reachable
  - Check the console for “Admin Panel bound to urls: …”
  - If needed, set `ASPNETCORE_URLS` and restart.

### Power users
- Change game version at initialization with `-version:<season6|0.75|0.95d>` together with `-reinit` or by using the Setup page in Admin Panel.
- Headless behavior: toggle “Read Console Input” in Admin Panel (Configuration → System).

You’re set. Publish, run, open the Admin Panel, and enjoy!


## Install OpenMU on Ubuntu (native, non-Docker)

This guide installs and runs the all‑in‑one OpenMU server natively on Ubuntu (no Docker).
Tested on Ubuntu 22.04/24.04. Adjust commands with sudo as needed.

### Overview
- PostgreSQL for persistence
- .NET SDK 9 (framework‑dependent publish; run with `dotnet`)
- Admin Panel (Blazor Server) on HTTP

### 1) Install prerequisites
Update packages:
```bash
sudo apt update
```
Install PostgreSQL:
```bash
sudo apt install -y postgresql postgresql-contrib
```
Install Microsoft package repo for .NET 9:
- Ubuntu 24.04:
  ```bash
  wget https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
  sudo dpkg -i packages-microsoft-prod.deb
  sudo apt update
  ```
- Ubuntu 22.04:
  ```bash
  wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
  sudo dpkg -i packages-microsoft-prod.deb
  sudo apt update
  ```
Install .NET SDK 9:
```bash
sudo apt install -y dotnet-sdk-9.0
```

### 2) Prepare PostgreSQL
Set a password for the `postgres` superuser (replace YOUR_PASSWORD):
```bash
sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'YOUR_PASSWORD';"
```
Ensure the service is running:
```bash
systemctl is-active --quiet postgresql || sudo systemctl start postgresql
```

Default local auth is usually password-capable out of the box (`scram-sha-256`).
If you previously customized `pg_hba.conf`, ensure local TCP connections allow password auth for `postgres` (e.g., `scram-sha-256` or `md5`). Then restart:
```bash
sudo systemctl restart postgresql
```

You do not need to manually create the `openmu` database or extra roles; OpenMU initializes them on first run when connected as `postgres` admin.

### 3) Get the source
```bash
git clone https://github.com/MUnique/OpenMU.git
cd OpenMU
```
Or download and extract the ZIP instead.

### 4) Configure database connection
Option A (recommended): Set environment variables for admin connection at runtime:
```bash
export DB_HOST=localhost
export DB_ADMIN_USER=postgres
export DB_ADMIN_PW=YOUR_PASSWORD
```

Option B (alternative): Edit the first two entries in
`src/Persistence/EntityFramework/ConnectionSettings.xml` and change only the `Password` for `User Id=postgres`.
Leave the other users (`config`, `account`, `friend`, `guild`) unchanged.

### 5) Publish the server
Publishing includes all required web assets.
```bash
dotnet publish ./src/Startup/MUnique.OpenMU.Startup.csproj -c Release -o ./server
```

### 6) Run the server
From the publish directory:
```bash
cd server
# If you used env vars, ensure they’re still exported in this shell
export DB_HOST=${DB_HOST:-localhost}
export DB_ADMIN_USER=${DB_ADMIN_USER:-postgres}
export DB_ADMIN_PW=${DB_ADMIN_PW:-YOUR_PASSWORD}

# Auto-start servers; resolve local IP; optionally reinitialize the DB on first run
dotnet ./MUnique.OpenMU.Startup.dll -autostart -resolveIP:local
# Optional first-time force (re)initialization:
# dotnet ./MUnique.OpenMU.Startup.dll -autostart -resolveIP:local -reinit
```

Notes:
- `-autostart` starts connect/game/chat listeners automatically.
- `-resolveIP:local` advertises a local LAN/loopback IP suitable for same host or LAN clients.
- You can switch the initial game data version with `-version:season6|0.75|0.95d` in combination with `-reinit`, or use the Setup page in the Admin Panel.
- The legacy `-deamon` flag is deprecated; console input can be disabled/enabled under Configuration → System in the Admin Panel.

### 7) Admin Panel (web UI)
On startup, the server prints the Admin Panel URLs (typically `http://localhost:5000`). Open it in your browser and verify servers are running. If you started without `-autostart`, use the panel to start servers.

Bind to a specific URL/port (e.g., 0.0.0.0:5000) by setting:
```bash
export ASPNETCORE_URLS="http://0.0.0.0:5000"
dotnet ./MUnique.OpenMU.Startup.dll -autostart -resolveIP:local
```

### 8) IP resolver options
Choose how the advertised IP is determined (used by clients when connecting):
- `-resolveIP:public` – detect public IP automatically (default if not specified)
- `-resolveIP:local` – pick a local IP or fallback loopback (127.127.127.127)
- `-resolveIP:loopback` – force loopback (127.127.127.127) for same-machine testing
- `-resolveIP:<IPv4>` – specify a fixed IP (e.g., `-resolveIP:192.168.1.10`)

Alternative env var if the flag is omitted: `RESOLVE_IP` (same values).

### 9) Open firewall ports (ufw)
If UFW is enabled, allow:
```bash
sudo ufw allow 80/tcp        # Admin Panel (if bound to 80)
sudo ufw allow 5000/tcp      # Admin Panel default
sudo ufw allow 44405:44406/tcp
sudo ufw allow 55901:55906/tcp
sudo ufw allow 55980/tcp
sudo ufw reload
```

### 10) Test accounts
Created automatically when the database is initialized. Passwords equal usernames.
- `test0`–`test9`: level 1..90 in steps of 10
- Season 6 only: `test300`, `test400`, `testgm`, `testgm2`, `testunlock`, `quest1`, `quest2`, `quest3`, `ancient`, `socket`

Create accounts in Admin Panel (Accounts → Create). Prefer creating characters via the game client.

### Optional: run as a systemd service
Create `/etc/systemd/system/openmu.service` (adjust paths and environment):
```ini
[Unit]
Description=OpenMU Server
After=network.target postgresql.service

[Service]
Type=simple
WorkingDirectory=/opt/OpenMU/server
Environment=ASPNETCORE_URLS=http://0.0.0.0:5000
Environment=DB_HOST=localhost
Environment=DB_ADMIN_USER=postgres
Environment=DB_ADMIN_PW=YOUR_PASSWORD
ExecStart=/usr/bin/dotnet /opt/OpenMU/server/MUnique.OpenMU.Startup.dll -autostart -resolveIP:public
Restart=on-failure
User=www-data
Group=www-data

[Install]
WantedBy=multi-user.target
```
Then:
```bash
sudo systemctl daemon-reload
sudo systemctl enable openmu
sudo systemctl start openmu
sudo systemctl status openmu
```

### Troubleshooting
- Database authentication failed
  - Verify `DB_ADMIN_USER`/`DB_ADMIN_PW`. Ensure `postgres` has a password: `sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD '...';"`
  - Confirm local TCP auth in `pg_hba.conf` allows password (`scram-sha-256`/`md5`), then `sudo systemctl restart postgresql`.
- Admin Panel unreachable
  - Check console logs for bound URLs. If binding to 0.0.0.0:5000, ensure firewall allows 5000.
  - Try setting `ASPNETCORE_URLS="http://0.0.0.0:5000"` before starting.
- Ports already in use
  - Free the ports or change settings in Admin Panel → Configuration → System, then restart affected servers.
- Same-machine client
  - Use `-resolveIP:loopback` to advertise 127.127.127.127 (the client blocks 127.0.0.1).

You’re done. Publish, run, open the Admin Panel, and enjoy!

