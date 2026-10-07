# Volatility 3 Cheatsheet (Memory Forensics / CTF)

Quick reference for analyzing a Windows memory dump (`chall.mem`) with Volatility 3.

**Command format:**

```bash
vol -f chall.mem <plugin> 2>/dev/null
```

- Save output to a file: add `> name.txt`
- `2>/dev/null` hides the harmless `python-magic` error at the end
- Dump output folder: `-o out/` (placed **before** the plugin name)
- Plugin names can change between versions (e.g. `windows.malfind` is now `windows.malware.malfind`). List them with `vol -h | grep windows`

---

## 1. Quick Workflow

```
info -> pstree/psscan -> netscan -> cmdline -> malfind -> filescan -> dump -> hash -> VirusTotal
```

---

## 2. Basics

```bash
windows.info                  # OS version, dump time (check the file is valid)
```

---

## 3. Processes

```bash
windows.pslist                # active processes (PID, PPID, create time)
windows.psscan                # also finds exited / hidden processes
windows.pstree                # parent-child tree view
windows.cmdline               # command line arguments
windows.cmdline --pid 1234    # one process only
windows.envars --pid 1234     # environment variables
windows.getsids --pid 1234    # owner (user) of the process
```

**Tips:**
- In `pslist`/`psscan` output, **PPID is column 2**: `awk '$2==1234'`
- If a process shows in `psscan` but not `pslist`, it exited or is hidden
- In `pstree`, more `*` means deeper child

---

## 4. Network

```bash
windows.netscan               # all connections + sockets (most complete)
windows.netstat               # active connections
```

Look for: `ESTABLISHED` to external IPs, odd ports (4444, 8080), and the owning PID.

```bash
windows.netscan | grep -E "ESTABLISHED|LISTENING"
windows.netscan | grep 1234
```

---

## 5. Injection and Memory

```bash
windows.malware.malfind                 # RWX regions + MZ header
windows.malware.malfind --pid 1234
windows.vadinfo --pid 1234              # memory regions + protection
windows.dlllist --pid 1234              # loaded DLLs
windows.ldrmodules --pid 1234           # hidden DLLs
windows.handles --pid 1234              # handles (files, registry, mutex)
```

Suspicious protection: `PAGE_EXECUTE_READWRITE`.
Check the **Notes** column: `MZ header` means an injected PE file.

`malfind` columns: `PID | Process | Start VPN | End VPN | Tag | Protection | CommitCharge | PrivateMemory | File output | Notes`

---

## 6. Files

```bash
windows.filescan | grep -i name         # find a file, note the offset
windows.dumpfiles --virtaddr 0xOFFSET   # dump file by offset (or --physaddr)
windows.dumpfiles --pid 1234            # all files of a process
windows.mftscan.MFTScan                 # MFT entries
```

```bash
vol -f chall.mem -o out/ windows.dumpfiles --virtaddr 0xOFFSET
```

---

## 7. Dump Process and Memory

```bash
-o out/ windows.pslist --pid 1234 --dump            # dump the process exe
-o out/ windows.memmap --pid 1234 --dump            # dump all process memory
-o out/ windows.malware.malfind --pid 1234 --dump   # dump injected regions
```

Then:

```bash
sha256sum out/*                  # hash -> check on VirusTotal
strings -a out/file | less       # ASCII strings
strings -a -el out/file | less   # Unicode strings (Windows)
```

---

## 8. Registry and Persistence

```bash
windows.registry.hivelist
windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"
windows.registry.userassist              # programs run by the user
windows.svcscan                          # services (persistence)
```

Common persistence: `Run`, `RunOnce`, new services, Scheduled Tasks.

---

## 9. Credentials and Users

```bash
windows.hashdump                 # NTLM user hashes
windows.lsadump                  # LSA secrets
windows.cachedump                # cached domain credentials
```

---

## 10. Command History

```bash
windows.cmdscan                  # cmd.exe history
windows.consoles                 # console input/output (more complete)
```

---

## 11. Other

```bash
windows.timeliner                # event timeline
windows.modscan                  # kernel modules
windows.driverscan               # drivers (rootkit check)
windows.vadyarascan --yara-rules "string"   # search a pattern in process memory
```

---

## 12. How to Spot a Suspicious Process

1. **Name:** misspelled or unknown (`svch0st`, `oneetx`)
2. **Path:** `Temp`, `AppData`, `Downloads`, `Public`, random folder names
3. **Parent-child:** `cmd`/`powershell` spawned by Word or a browser; `svchost` not a child of `services.exe`
4. **Count:** `lsass`, `services`, `wininit` must be unique
5. **Behavior:** outbound connections, `-enc` arguments, RWX memory + `MZ`

Two or more signs on the same process = strongly suspicious.

**Normal parent-child rules:**
- `svchost.exe` -> parent `services.exe`
- `services.exe`, `lsass.exe` -> parent `wininit.exe`
- `cmd.exe` / `powershell.exe` from Office, browser, or PDF reader = suspicious

---

## 13. Common Question -> Plugin

| Question | Plugin |
|---|---|
| Suspicious process name | `pstree`, `netscan` |
| Child / parent process | `pstree`, `psscan` (PPID column) |
| Memory protection | `malfind`, `vadinfo` |
| C2 IP / port | `netscan` |
| VPN / tunnel process | `pstree` (check parent-child) |
| Malware file path | `pstree`, `filescan`, `cmdline` |
| Malware family | dump -> `sha256sum` -> VirusTotal |
| Attack time | Create Time column, `info` (dump time) |
| Persistence | `printkey` Run, `svcscan` |
| Passwords / hashes | `hashdump`, `lsadump` |

---

## 14. Terminal Tricks

```bash
grep -i            # ignore upper/lower case
grep -E "a|b"      # a OR b
grep -A3 / -B3     # show 3 lines after / before the match
awk '$2==1234'     # filter by column 2 (PPID)
2>/dev/null        # hide python-magic error
```

---

## 15. Example: Redline Lab (CyberDefenders)

| Finding | How it was found |
|---|---|
| Suspicious process `oneetx.exe` in `AppData\Local\Temp\<random>\` | `pstree`, `netscan` |
| Protection `PAGE_EXECUTE_READWRITE` + `MZ header` | `malfind` |
| VPN: `Outline.exe` (parent) launched `tun2socks.exe` (child) | `pstree` (PID/PPID) |
| VPN tunnel hides traffic from NIDS | `tun2socks` = encrypted tunnel via SOCKS proxy |
