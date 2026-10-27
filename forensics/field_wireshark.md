# Field Filter Wireshark Per Protokol

Pola nama field: `protokol.bagian` (contoh: `http.host`, `dns.qry.name`, `smtp.req.command`).

Lupa nama field? Klik field di panel detail → lihat status bar bawah, atau klik kanan → **Apply as Filter**. Atau ketik `namaprotokol.` di kolom filter dan pilih dari autocomplete.

---

## Umum (IP, TCP, UDP)

| Field | Fungsi |
|---|---|
| `ip.addr`, `ip.src`, `ip.dst` | Filter berdasarkan IP |
| `tcp.port`, `udp.port` | Filter berdasarkan port |
| `tcp.stream` | Nomor aliran koneksi (untuk Follow Stream) |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Awal koneksi baru |
| `frame contains "teks"` | Cari teks di seluruh paket |

## DNS

| Field | Fungsi |
|---|---|
| `dns.qry.name` | Domain yang ditanyakan |
| `dns.flags.response` | 0 = query, 1 = response |
| `dns.a` | IP (IPv4) hasil jawaban |

## HTTP

| Field | Fungsi |
|---|---|
| `http.request` / `http.response` | Hanya request / respons |
| `http.host` | Domain tujuan |
| `http.request.uri` | Path/file yang diminta |
| `http.request.method` | GET, POST, dll. |
| `http.user_agent` | Identitas klien |
| `http.response.code` | 200, 404, 500, dll. |

## SMTP / Email

| Field | Fungsi |
|---|---|
| `smtp.req.command` | EHLO, AUTH, MAIL, RCPT, DATA |
| `smtp.command_line` | Satu baris perintah lengkap |
| `imf.from`, `imf.to`, `imf.subject` | Pengirim, penerima, subjek email |

## FTP

| Field | Fungsi |
|---|---|
| `ftp.request.command` | USER, PASS, STOR, RETR |
| `ftp.request.arg` | Isi perintah (username, password, nama file) |

## SMB

| Field | Fungsi |
|---|---|
| `smb2.filename` | File yang dibuka |
| `smb2.tree` | Share yang diakses |
| `ntlmssp.auth.username` | Username yang login |

## TLS (HTTPS)

| Field | Fungsi |
|---|---|
| `tls.handshake.extensions_server_name` | Nama domain di koneksi HTTPS (SNI), walau isinya terenkripsi |

---

## Operator

| Operator | Arti |
|---|---|
| `==` | Sama persis |
| `!=` | Tidak sama |
| `&&` | Dan |
| `\|\|` | Atau |
| `!` | Bukan |
| `contains` | Mengandung teks (huruf besar/kecil dibedakan) |
| `matches` | Regex, contoh: `matches "(?i)ehlo\|helo"` |

---

## Contoh Pemakaian

```
dns.flags.response == 0                      # query DNS (Destination = IP DNS server)
dns.qry.name contains "ipify"                # cari layanan pengecek IP
http.host contains "whatismyipaddress"       # request ke pengecek IP publik
http.request.uri contains ".exe"             # unduhan file executable
smtp.command_line contains "EHLO"            # awal tiap sesi email
imf                                          # isi email (subject, body)
ftp.request.command == "PASS"                # password FTP
smb2.tree                                    # share SMB yang diakses
ip.src == 10.4.10.132 && smtp                # SMTP dari korban saja
```

---

## Prioritas Hafalan

Yang paling sering dipakai, hafalkan dulu:

- [ ] `ip.addr`
- [ ] `tcp.port`
- [ ] `dns.qry.name`
- [ ] `http.host`
- [ ] `http.request.uri`
- [ ] `smtp.req.command`
- [ ] `frame contains`

---

## Filter Temuan Sendiri

| Tujuan | Filter |
|---|---|
|  |  |
|  |  |
|  |  |
