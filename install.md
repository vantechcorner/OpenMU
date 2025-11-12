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


