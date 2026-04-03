# Scanner — Port & Exploit Scanner

A lightweight, interactive port scanner and exploit tester for Grey Hack. Scans target IPs, discovers vulnerabilities via metaxploit, classifies exploit results, and provides post-exploitation tools — all from a single script.

---

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Scanning Flow](#scanning-flow)
  - [Port Discovery](#1-port-discovery)
  - [Exploit Scanning](#2-exploit-scanning)
  - [Exploit Types](#3-exploit-types)
- [Post-Exploitation Menu](#post-exploitation-menu)
  - [Crack Passwords](#c-crack-passwords)
  - [Switch User (Su)](#s-switch-user)
  - [Launch Apps](#a-launch-apps)
  - [Local Library Scan](#l-local-library-scan)
  - [LAN Network Scan](#n-lan-network-scan)
  - [File Browser](#f-file-browser)
  - [Download / Upload Files](#d-download--u-upload)
  - [Add Password](#p-add-password)
  - [Rename Files](#r-rename)
  - [Wipe Log](#w-wipe-log)
- [Bouncing](#bouncing)
- [Navigation Reference](#navigation-reference)
- [Examples](#examples)

---

## Requirements

- **metaxploit.so** — Must be in `/lib/` or the current directory. Required for all scanning.
- **crypto.so** — Optional, but needed for password cracking. Looked for in `/lib/` then current directory.

## Installation

1. Copy the source code from the [scanner.src GitHub file](scanner.src)
2. In-game, open **CodeEditor.exe** (found in `/usr/bin/` or launch from desktop)
3. Paste the code into the editor
4. Click **Build** to compile it (optionally save the file first)

The compiled binary will be placed in your home directory. No other dependencies needed — the script self-contains all helpers.

---

## Usage

```
scanner [target] [-r] [-q]
```

| Argument | Description |
|----------|-------------|
| `target` | IP address or domain name to scan |
| `-r`     | Scan a random public IP instead |
| `-q`     | Quick scan — show port table only, skip exploit testing (like `nmap`) |

**Examples:**

```bash
scanner 192.168.0.1          # Scan specific IP
scanner www.somewhere.com    # Resolve domain, then scan
scanner -r                   # Scan a random public IP
scanner 10.0.0.1 -q          # Quick port scan only, like nmap
```

If no arguments are given, scanner launches in **interactive mode** with a main menu:

```
[ip]  Scan target IP or domain
[r]   Scan random IP
[?]   Help
[x]   Exit
```

---

## Scanning Flow

### 1. Port Discovery

Scanner queries the target's router for all used ports (plus port 0 for the router itself), then displays a table:

```
PORT    STATE    SERVICE       VERSION    LAN
0       open     router        1.2.3      192.168.0.1
22      open     ssh           1.0.0      192.168.0.2
80      open     http          2.1.0      192.168.0.3
25      closed   smtp          1.0.0      192.168.0.4
```

From here you can:
- Enter a **port number** to scan just that port
- Enter **`a`** to scan all ports at once
- Enter **`b`** to go back

### 2. Exploit Scanning

For each selected port, scanner automatically tests for known vulnerabilities and classifies what each exploit returns. Results are displayed with counts:

```
[1] • kernel_router.so (Version: 1.2.3) -> 192.168.0.1:0
    |-> Not Patched
    5 exploits found
    Shells:2 - Computers:1 - Files:0 - Bounces:1 - Unknown:1
```

### 3. Exploit Types

| Type | Color | Description |
|------|-------|-------------|
| **shell** | Green | Returns a shell object — full command access |
| **computer** | Yellow | Returns a computer object — file system access |
| **file** | Red | Returns a file object — single file access |
| **bounce** | Purple | Router exploit that can redirect to another public IP |
| **null** | Grey | Returned null — may be a password-change exploit or need a parameter |

When you select an exploit to run, its result opens the **post-exploitation menu**.

---

## Post-Exploitation Menu

After successfully exploiting a target, you get an interactive menu. Available options depend on exploit type:

```
[c] Crack      (3 accounts)
[s] Su         (2 cracked creds)
[a] Apps       (12 available)
[l] Local scan
[n] LAN scan
[f] Browse files
[d] Download file
[r] Rename file
[u] Upload file
[p] Add password to cache
[w] Wipe log
[b] Back
```

### `c` — Crack Passwords

Reads `/etc/passwd`, displays all user accounts with their hashes, then offers:

- Enter a **number** to crack a specific account
- Enter **`a`** to crack all
- Enter **`b`** to skip

Cracked credentials are stored for the session and shared across the network — use them with `s` (su) on any exploited machine.

### `s` — Switch User

If you have cracked credentials and aren't already root, presents a list of cracked username/password pairs. Select one to log in as that user on the remote machine. If successful, your session upgrades to that user's shell.

### `a` — Launch Apps

*(Shell exploits only)*

Lists all executables found in `/usr/bin`, `/bin`, and user home directories. Select an app by number, optionally followed by arguments:

```
[App # + args, or 'b' to go back] > 3 --help
```

### `l` — Local Library Scan

*(Shell exploits only)*

Scans libraries installed on the **remote** machine for local exploits. Works just like the main port scan, but targets local files instead of network ports.

This is how you escalate privileges when you have a non-root shell — find local exploits that give root.

### `n` — LAN Network Scan

*(Shell exploits only)*

Maps the internal network from the compromised machine. Discovers all devices on the LAN (routers, switches, cameras, appliances, etc.) and displays a device table with type, firewall status, and open ports. Select a device to scan its ports for exploits.

Device types detected:

| Type | Description |
|------|-------------|
| Edge Rtr | Edge/gateway router |
| Router | Internal router |
| Switch | Network switch |
| Camera | CCTV (port 37777) |
| Appliance | Smart appliance (port 1883) |
| RShell | Remote shell service (port 1222) |
| ADB | Android Debug Bridge (port 5555) |
| Device | Generic device |

Firewall status is shown as `F` (green = rules but no DENY, red = has DENY rules).

### `f` — File Browser

Interactive file system explorer. Shows permissions, owner, group, size for each file and directory. Interesting files (Bank.txt, Mail.txt, passwd, etc.) are highlighted with a ★.

```
#    PERMS       OWNER     GROUP     SIZE    NAME
1    drwxr-xr-x  root      root      -       bin/
2    -rw-r--r--  root      root      1234    passwd
3    -rw-------  tux       tux       567     Bank.txt ★
```

Controls:
- **Number** — Enter directory or cat file
- **Path** — Navigate directly to a path
- **`b`** — Go up one directory
- **`x`** — Exit browser

### `d` — Download File

Enter a remote file path to download it to your local `~/Downloads/` folder.

### `u` — Upload File

*(Shell exploits only)*

Upload a local file to the remote machine. Prompts for local path and optional destination directory.

### `p` — Add Password

Manually add a root password to the credential cache. Useful when you already know the password (e.g., from a previous session or a password-change exploit) and want to use `s` (su) without cracking.

### `r` — Rename

Rename any file or folder on the remote filesystem (requires write permission).

### `w` — Wipe Log

Wipes `/var/system.log` on the remote machine to remove your traces. Only shown when you have write permission to the log file.

---

## Bouncing

Router exploits (port 0) often return **bounce** or **null** type results. These can be used to reach other public IPs through the target router:

1. Select a null/bounce exploit
2. Enter the public IP you want to reach
3. If successful, you get a shell/computer on the bounce target

Use **`ba`** (bounce all) to automatically try all null exploits against a target IP at once.

---

## Navigation Reference

These controls are consistent across all menus:

| Key | Action |
|-----|--------|
| `#` | Select item by number |
| `a` | Scan/crack all |
| `b` | Go back to previous menu |
| `x` | Exit current section |
| `?` | Show help screen |
| `r` | Refresh (LAN scan only) |
| `ba` | Bounce all null exploits against a target IP |

---

## Examples

### Basic scan → exploit → crack → su → root

```
[target] > 45.33.21.100

PORT    STATE    SERVICE       VERSION    LAN
0       open     router        1.2.3      192.168.0.1
22      open     ssh           1.0.0      192.168.0.2
80      open     http          2.1.0      192.168.0.3

[Port #, 'a' scan all, 'b' back, '?' help] > a

[i] Scanning for exploits...

[1] • kernel_router.so (1.2.3) -> 192.168.0.1:0
    |-> Not Patched
    3 exploits found
[2] • libssh.so (1.0.0) -> 192.168.0.2:22
    |-> Not Patched
    2 exploits found

[Target #, 'b' back] > 2

#    TYPE        USER        ADDRESS       VALUE
1    shell       guest       0x7A3F        lqkSjf
2    shell       tux         0x4B12        rTpVx2

[Exploit #, 'b' back] > 2

[+] Got shell!
    User: tux
    Host: 45.33.21.100

[tux@45.33.21.100 shell] > c

#    USER            HASH
1    root            $1$xyz...
2    tux             $1$abc...

[Crack #/a/b] > 1
root            P@ssw0rd123

[tux@45.33.21.100 shell] > s

  Cracked credentials:
  1) root : P@ssw0rd123

[Connect as #, or 'b' to skip] > 1

[+] Got root shell!
```

### Pivot through LAN

```
[tux@45.33.21.100 shell] > n

[i] Building network map...

#    LAN IP            TYPE        FW   PORTS
1    192.168.0.1       Edge Rtr    F    0,22,80
2    192.168.0.2       Device           22
3    192.168.0.3       Device           80
4    192.168.0.4       Camera           37777
5    192.168.0.5       Appliance        1883

[Device # to scan, 'r' refresh, 'b' back] > 4
```

### Quick port scan

```bash
scanner 10.0.0.1 -q
```

Outputs the port table and exits immediately — basically `nmap`. Useful for quick recon without triggering exploits.
