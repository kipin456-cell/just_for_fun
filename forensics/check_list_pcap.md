# Checklist CTF & Forensik (Event Tanpa AI)

Cetak file ini atau simpan offline. Centang saat latihan, bukan hanya saat lomba.

---

## 0. Sebelum Event (H-7 sampai H-1)

- [ ] Kali/WSL siap dan sudah dites (Wireshark, tshark, binwalk, foremost, exiftool, steghide, zsteg, strings, file, xxd)
- [ ] Tools tambahan terpasang offline: Volatility, Autopsy/FTK Imager (kalau dipakai), CyberChef (simpan versi offline/HTML), john, hashcat
- [ ] Wordlist siap (`rockyou.txt`)
- [ ] Folder kerja rapi: `~/ctf/<nama-soal>/` untuk tiap soal
- [ ] Siapkan file catatan pribadi (cheat sheet) sesuai aturan event
- [ ] Latihan 2-3 soal timed tanpa AI dan tanpa write-up
- [ ] Baterai, charger, dan koneksi aman

---

## 1. Alur Umum Setiap Soal (Semua Kategori)

1. [ ] **Baca soal pelan-pelan.** Garis bawahi kata kunci (email, download, login, deleted, hidden, memory, dll.)
2. [ ] **Lihat format jawaban** (jumlah huruf/angka, format flag, contoh `****`)
3. [ ] **Identifikasi file:** `file nama_file`
4. [ ] **Cek info cepat:** `strings -n 8 file | less`, `exiftool file`, `xxd file | head`
5. [ ] **Cek file di dalam file:** `binwalk file`
6. [ ] **Catat temuan** di file notes (IP, domain, user, hash, waktu)
7. [ ] **Buntu 15 menit?** Pindah soal, kembali nanti
8. [ ] **Setelah dapat jawaban:** tulis 1-2 baris "caranya" untuk catatan

---

## 2. Checklist PCAP / Wireshark

### A. Gambaran umum (2 menit pertama)
- [ ] `Statistics → Capture File Properties` (jumlah paket, durasi)
- [ ] `Statistics → Protocol Hierarchy` (protokol apa saja yang ada)
- [ ] `Statistics → Conversations` (IP paling aktif; IP privat paling aktif biasanya korban)
- [ ] `Statistics → Endpoints`

### B. Periksa per protokol (sesuai hasil A)
- [ ] **DNS:** `dns` lalu `dns.flags.response == 0` (query) atau `== 1` (response); cek `dns.qry.name`
- [ ] **HTTP:** `http.request`; cek `Host`, path, dan User-Agent; `File → Export Objects → HTTP`
- [ ] **SMTP/IMF:** `smtp`, `imf`; banner `220` untuk software server; `EHLO` untuk hostname klien; `AUTH`, `MAIL FROM`, `RCPT TO`
- [ ] **FTP:** `ftp`, `ftp-data`; cek `USER`/`PASS`
- [ ] **TLS:** hanya terbaca kalau ada key; `Edit → Preferences → Protocols → TLS → RSA keys list`
- [ ] **TFTP/SMB/lainnya:** `File → Export Objects`
- [ ] **Telnet/plaintext:** Follow TCP Stream

### C. Follow stream
- [ ] Klik kanan paket → `Follow → TCP Stream` (ganti angka stream di pojok bawah untuk stream lain)
- [ ] Coba juga UDP Stream / HTTP Stream
- [ ] Cari `flag`, `pass`, `user`, `key` dengan `Ctrl+F`

### D. Pertanyaan yang sering muncul
| Pertanyaan | Cara cari |
|---|---|
| IP korban | Conversations / IP privat paling aktif |
| IP publik korban | Response layanan pengecek IP (`ipify`, `icanhazip`, dll.), lihat Line-based text data |
| IP DNS server | `dns.flags.response == 0` → kolom Destination |
| File yang diunduh | `http.request` → URI, atau Export Objects |
| Domain malware | `dns.qry.name`, `http.host` |
| Hash file | Export object lalu `sha256sum file` → cek VirusTotal |
| IP server email | `smtp` → Destination dari paket klien |
| Negara server | Geolokasi IP (abuseipdb, whois) |
| Software server | Banner `220` / header `Received` |
| Hostname korban | `EHLO`, DHCP, NBNS, SMB |
| Interval pengiriman | Selisih Time antar sesi (`smtp.command_line contains "EHLO"`) |
| Varian malware | Subject/body email, hash di VirusTotal |

### E. Filter dasar yang wajib hafal
```
ip.addr == x.x.x.x        ip.src == ...       ip.dst == ...
tcp.port == 80            udp.port == 53
dns   http   smtp   imf   ftp   tcp   udp
&&  (dan)    ||  (atau)   !  (bukan)    contains    ==
frame contains "flag"
```
Lupa nama field? Klik field di panel detail → lihat status bar bawah, atau klik kanan → `Apply as Filter`.

---

## 3. Checklist Kategori Lain (Sesuai Kisi-kisi)

### File carving / file signature
- [ ] `file`, `xxd | head`, cek magic bytes (PNG `89 50 4E 47`, JPG `FF D8 FF`, ZIP `50 4B 03 04`, PDF `25 50 44 46`)
- [ ] `binwalk -e file`, `foremost -i file`
- [ ] Header rusak? Perbaiki magic bytes dengan hex editor

### Steganografi
- [ ] `exiftool`, `strings`, `binwalk`
- [ ] Gambar: `zsteg` (PNG/BMP), `steghide extract -sf file` (JPG/BMP), `stegsolve`
- [ ] Audio: spectrogram (Audacity/Sonic Visualiser)
- [ ] Password? Cari di hint, metadata, atau file lain di soal

### Disk / OS / Windows
- [ ] Mount image atau buka di Autopsy
- [ ] Cek: file terhapus, Recycle Bin, Recent files, Browser history, Registry, Prefetch, AppData
- [ ] Catat timestamp (MAC times)

### Log analysis
- [ ] Format log dikenali (Apache, auth.log, Windows Event, syslog)
- [ ] `grep`, `awk`, `sort | uniq -c | sort -rn` untuk hitung IP/user paling sering
- [ ] Cari: login gagal berulang (brute force), IP asing, jam tidak biasa, perintah mencurigakan

### Memory forensics
- [ ] `vol -f mem.raw windows.info`
- [ ] `windows.pslist`, `windows.pstree`, `windows.cmdline`, `windows.netscan`, `windows.filescan`
- [ ] Dump proses/file mencurigakan lalu analisis

### Malware (analisis dasar)
- [ ] `sha256sum` lalu cek VirusTotal
- [ ] `strings`, cek URL/IP/domain tertanam
- [ ] Jangan jalankan sample di mesin utama

### Linux forensics
- [ ] `~/.bash_history`, `/var/log/auth.log`, `/etc/passwd`, `/etc/crontab`, `/tmp`
- [ ] Cek user baru, cron job, SSH keys

### Encoding / Kripto dasar
- [ ] Ciri Base64: huruf+angka, akhir `=`; Hex: `0-9a-f`; ROT13; Binary
- [ ] CyberChef → "Magic"
- [ ] Layered? Decode berulang sampai terbaca

---

## 4. Manajemen Waktu Saat Event

- [ ] Kerjakan soal termudah/poin kecil dulu
- [ ] Batas 15-20 menit per soal sebelum pindah
- [ ] Setiap 1 jam, cek ulang soal yang tertunda
- [ ] Simpan jawaban sementara di notes, submit begitu yakin
- [ ] Sisakan 10 menit terakhir untuk mengecek format jawaban

---

## 5. Kalau Buntu (Tanpa AI)

1. [ ] Baca ulang soal dan hint, mungkin ada kata kunci yang terlewat
2. [ ] Ulangi dari langkah paling awal (`file`, `strings`, `exiftool`)
3. [ ] Coba tool lain pada file yang sama
4. [ ] Cari di man page: `man tool`, `tool --help`
5. [ ] Cek dokumentasi offline (`tshark -G fields | grep smtp`)
6. [ ] Tanya diri sendiri: "Data apa yang belum aku buka?" (stream lain, packet lain, file lain)
7. [ ] Pindah soal

---

## 6. Rutinitas Latihan Harian

- [ ] 1 soal pcap tanpa bantuan (batas 30 menit)
- [ ] Setelah selesai/menyerah, baca write-up lalu **ulang sendiri dari awal**
- [ ] Tulis catatan singkat: tool apa, filter apa, kenapa berhasil
- [ ] Review catatan kemarin 5 menit sebelum mulai
- [ ] 1x seminggu: simulasi timed 2-3 jam tanpa AI dan tanpa write-up

---

## 7. Template Catatan Per Soal

```
Soal      :
Kategori  :
File      :
Format jawaban :
Temuan    :
 - 
Tool/Filter yang dipakai :
Jawaban   :
Pelajaran :
```

---

## 8. Daftar Filter Wireshark Pribadi (Isi Sendiri)

Tulis filter yang kamu temukan sendiri saat latihan:

| Tujuan | Filter |
|---|---|
|  |  |
|  |  |
|  |  |
