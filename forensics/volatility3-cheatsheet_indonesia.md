# Cheatsheet Volatility 3 (Forensik Memori / CTF)

Referensi cepat untuk menganalisis dump memori Windows (`chall.mem`) dengan Volatility 3.

**Format perintah:**

```bash
vol -f chall.mem <plugin> 2>/dev/null
```

- Simpan hasil ke file: tambahkan `> nama.txt`
- `2>/dev/null` menyembunyikan error `python-magic` di akhir (tidak berbahaya)
- Folder hasil dump: `-o out/` (ditaruh **sebelum** nama plugin)
- Nama plugin bisa berubah antar versi (contoh: `windows.malfind` menjadi `windows.malware.malfind`). Cek daftarnya dengan `vol -h | grep windows`

---

## 1. Alur Cepat

```
info -> pstree/psscan -> netscan -> cmdline -> malfind -> filescan -> dump -> hash -> VirusTotal
```

---

## 2. Dasar

```bash
windows.info                  # versi OS, waktu dump (cek file valid)
```

---

## 3. Proses

```bash
windows.pslist                # proses aktif (PID, PPID, waktu buat)
windows.psscan                # termasuk proses yang sudah exit / tersembunyi
windows.pstree                # tampilan parent-child
windows.cmdline               # argumen command line
windows.cmdline --pid 1234    # satu proses saja
windows.envars --pid 1234     # environment variable
windows.getsids --pid 1234    # user pemilik proses
```

**Tips:**
- Di output `pslist`/`psscan`, **PPID ada di kolom 2**: `awk '$2==1234'`
- Kalau proses muncul di `psscan` tetapi tidak di `pslist`, berarti sudah exit atau disembunyikan
- Di `pstree`, semakin banyak `*` berarti semakin dalam (child)

---

## 4. Jaringan

```bash
windows.netscan               # semua koneksi + socket (paling lengkap)
windows.netstat               # koneksi aktif
```

Cari: `ESTABLISHED` ke IP luar, port aneh (4444, 8080), dan PID pemiliknya.

```bash
windows.netscan | grep -E "ESTABLISHED|LISTENING"
windows.netscan | grep 1234
```

---

## 5. Injeksi dan Memori

```bash
windows.malware.malfind                 # region RWX + header MZ
windows.malware.malfind --pid 1234
windows.vadinfo --pid 1234              # detail region memori + proteksi
windows.dlllist --pid 1234              # DLL yang dimuat
windows.ldrmodules --pid 1234           # DLL tersembunyi
windows.handles --pid 1234              # handle (file, registry, mutex)
```

Proteksi mencurigakan: `PAGE_EXECUTE_READWRITE`.
Cek kolom **Notes**: `MZ header` berarti ada file PE yang disuntikkan.

Kolom `malfind`: `PID | Process | Start VPN | End VPN | Tag | Protection | CommitCharge | PrivateMemory | File output | Notes`

---

## 6. File

```bash
windows.filescan | grep -i nama         # cari file, catat offset
windows.dumpfiles --virtaddr 0xOFFSET   # dump file dari offset (atau --physaddr)
windows.dumpfiles --pid 1234            # semua file milik proses
windows.mftscan.MFTScan                 # entri MFT
```

```bash
vol -f chall.mem -o out/ windows.dumpfiles --virtaddr 0xOFFSET
```

---

## 7. Dump Proses dan Memori

```bash
-o out/ windows.pslist --pid 1234 --dump            # dump file exe proses
-o out/ windows.memmap --pid 1234 --dump            # dump seluruh memori proses
-o out/ windows.malware.malfind --pid 1234 --dump   # dump region injeksi
```

Lalu:

```bash
sha256sum out/*                  # hash -> cek di VirusTotal
strings -a out/file | less       # string ASCII
strings -a -el out/file | less   # string Unicode (Windows)
```

---

## 8. Registry dan Persistence

```bash
windows.registry.hivelist
windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"
windows.registry.userassist              # program yang pernah dijalankan user
windows.svcscan                          # service (persistence)
```

Persistence umum: `Run`, `RunOnce`, service baru, Scheduled Task.

---

## 9. Kredensial dan User

```bash
windows.hashdump                 # hash NTLM user
windows.lsadump                  # secret LSA
windows.cachedump                # cached domain credential
```

---

## 10. Riwayat Command

```bash
windows.cmdscan                  # riwayat cmd.exe
windows.consoles                 # input/output console (lebih lengkap)
```

---

## 11. Lainnya

```bash
windows.timeliner                # timeline kejadian
windows.modscan                  # modul kernel
windows.driverscan               # driver (cek rootkit)
windows.vadyarascan --yara-rules "string"   # cari pola di memori proses
```

---

## 12. Cara Mengenali Proses Mencurigakan

1. **Nama:** salah eja atau asing (`svch0st`, `oneetx`)
2. **Path:** `Temp`, `AppData`, `Downloads`, `Public`, nama folder acak
3. **Parent-child:** `cmd`/`powershell` dari Word atau browser; `svchost` bukan anak `services.exe`
4. **Jumlah:** `lsass`, `services`, `wininit` hanya boleh satu
5. **Perilaku:** koneksi keluar, argumen `-enc`, memori RWX + `MZ`

Dua tanda atau lebih pada proses yang sama berarti kuat dicurigai.

**Aturan parent-child yang wajar:**
- `svchost.exe` -> parent `services.exe`
- `services.exe`, `lsass.exe` -> parent `wininit.exe`
- `cmd.exe` / `powershell.exe` dari Office, browser, atau PDF reader = mencurigakan

---

## 13. Pertanyaan Soal -> Plugin

| Pertanyaan soal | Plugin |
|---|---|
| Nama proses mencurigakan | `pstree`, `netscan` |
| Child / parent process | `pstree`, `psscan` (kolom PPID) |
| Proteksi memori | `malfind`, `vadinfo` |
| IP / port C2 | `netscan` |
| Proses VPN / tunnel | `pstree` (cek parent-child) |
| Path file malware | `pstree`, `filescan`, `cmdline` |
| Malware family | dump -> `sha256sum` -> VirusTotal |
| Waktu serangan | kolom Create Time, `info` (waktu dump) |
| Persistence | `printkey` Run, `svcscan` |
| Password / hash | `hashdump`, `lsadump` |

---

## 14. Trik Terminal

```bash
grep -i            # abaikan huruf besar/kecil
grep -E "a|b"      # a ATAU b
grep -A3 / -B3     # tampilkan 3 baris sesudah / sebelum yang cocok
awk '$2==1234'     # filter kolom ke-2 (PPID)
2>/dev/null        # sembunyikan error python-magic
```

---

## 15. Contoh: Lab Redline (CyberDefenders)

| Temuan | Cara menemukan |
|---|---|
| Proses mencurigakan `oneetx.exe` di `AppData\Local\Temp\<folder acak>\` | `pstree`, `netscan` |
| Proteksi `PAGE_EXECUTE_READWRITE` + `MZ header` | `malfind` |
| VPN: `Outline.exe` (parent) menjalankan `tun2socks.exe` (child) | `pstree` (PID/PPID) |
| Tunnel VPN menyembunyikan trafik dari NIDS | `tun2socks` = tunnel terenkripsi lewat proxy SOCKS |
