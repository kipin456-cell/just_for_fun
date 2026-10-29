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


Claude

Tinggal dua pertanyaan lagi: host name dan user account. Keduanya dicari lewat protokol yang “membocorkan” identitas komputer.

Host name (pilih salah satu, cocokkan hasilnya):

    Filter dhcp && eth.addr == <MAC kamu>, buka paket Request, lalu cari field Option: (12) Host Name.
    Filter nbns, lalu lihat nama NetBIOS yang didaftarkan oleh 10.6.13.133.
    Filter kerberos.CNameString. Nama yang berakhiran $ adalah nama komputer, dan itu juga bisa jadi host name.

User account:

    Filter kerberos.CNameString && ip.src == 10.6.13.133.
    Pilih nama yang tidak berakhiran $. Akhiran $ artinya akun komputer, sedangkan yang tanpa $ adalah akun manusia.
    Cara cepat: klik kanan field CNameString → Apply as Column, supaya semua nama langsung terlihat berjajar.

Kalau Kerberos tidak muncul: filter ntlmssp.auth.username, atau smb2 && ip.src == 10.6.13.133. Nama user kadang muncul di salah satunya.

Terakhir: kumpulkan 4 jawabanmu (IP, MAC, host name, user), lalu cek apakah semuanya konsisten. Misalnya, MAC dari DHCP harus sama dengan MAC di Ethernet II paket 10.6.13.133. Kalau ada yang tidak cocok, berarti ada yang salah ambil.

Setelah itu, kirim hasilnya ke sini dan kita cek bareng.

kalau di pcap lain,misalkan kalau di suruh mencari hostname lagi,yang perlu di cek apa ya(yang mutlak)
Mengumpulkan sumber hostname dari opsi DHCP dan NBNS

Tidak ada satu sumber yang mutlak, tapi ada beberapa protokol yang hampir selalu membocorkan host name. Cek sesuai urutan ini dan filter selalu berdasarkan IP atau MAC korban.

Urutan pengecekan:
#	Protokol	Filter	Field yang dilihat
1	DHCP	dhcp atau dhcp.option.hostname	Option 12 (Host Name)
2	NBNS	nbns	Nama di Name Registration atau Query
3	Kerberos	kerberos.CNameString	Nama berakhiran $ (akun komputer)
4	NTLM/SMB	ntlmssp.auth.hostname	Workstation name
5	mDNS/LLMNR	mdns atau llmnr	Nama di query atau response

Cara pakainya:

    Mulai dari DHCP. Di sinilah host name paling sering muncul, terutama di paket Request.
    Kalau DHCP kosong, lanjut NBNS lalu Kerberos. Di jaringan Windows (Active Directory) salah satunya hampir pasti ada.
    Cocokkan minimal dua sumber. Kalau DHCP dan Kerberos memberi nama yang sama, itu jawabanmu.
    Jangan salah ambil. Nama yang berakhiran $ di Kerberos kadang milik DC atau komputer lain, jadi pastikan IP sumbernya sama dengan korban.

Kebiasaan yang membantu:

    Pasang kolom eth.src dan ip.src supaya cepat melihat siapa pengirimnya.
    Kalau jaringannya bukan Windows (Linux atau Mac), mDNS dan DHCP biasanya yang paling berguna.

Mau aku masukkan ini ke checklist markdown pcap untuk dibawa ke event tanpa AI?

Create a spreadsheet from your last answer.
Read xlsx skill before creating spreadsheetSKILL.md
Read xlsx skill before creating spreadsheet SKILL.md
Claude’s response was interrupted.

buat file md nya
Created a file, shared files

File pcap-hostname-checklist.md sudah jadi. Isinya tabel protokol beserta filter Wireshark-nya, langkah kerja, catatan penting, dan tips, jadi bisa dibawa ke event tanpa AI.

Kalau mau, aku bisa gabungkan dengan checklist untuk IP, MAC, dan user account jadi satu file pcap lengkap.
Pcap hostname checklist
Document · MD 


Claude is AI and can make mistakes. Please double-check responses.
Pcap hostname checklist · MD
# Checklist: Mencari Host Name di PCAP
 
Gunakan urutan ini dan **selalu filter berdasarkan IP atau MAC korban**.
 
## Urutan Pengecekan
 
| # | Protokol | Filter Wireshark | Field yang Dilihat |
|---|----------|------------------|--------------------|
| 1 | DHCP | `dhcp` atau `dhcp.option.hostname` | Option 12 (Host Name) |
| 2 | NBNS | `nbns` | Nama di Name Registration / Query |
| 3 | Kerberos | `kerberos.CNameString` | Nama berakhiran `$` (akun komputer) |
| 4 | NTLM / SMB | `ntlmssp.auth.hostname` | Workstation name |
| 5 | mDNS / LLMNR | `mdns` atau `llmnr` | Nama di query atau response |
 
## Langkah Kerja
 
- [ ] Catat dulu IP dan MAC korban
- [ ] Mulai dari **DHCP** (paling sering muncul, terutama di paket Request)
- [ ] Kalau DHCP kosong, lanjut **NBNS**, lalu **Kerberos**
- [ ] Cocokkan **minimal dua sumber**. Kalau hasilnya sama, itu jawabanmu
- [ ] Pastikan IP sumber paket = IP korban (jangan ambil nama milik DC atau komputer lain)
## Catatan Penting
 
- Di Kerberos, akhiran `$` = **akun komputer**, tanpa `$` = **akun user**
- Jaringan Windows (Active Directory): NBNS atau Kerberos hampir pasti ada
- Jaringan Linux / Mac: **DHCP** dan **mDNS** biasanya paling berguna
## Tips Wireshark
 
- Pasang kolom `eth.src` dan `ip.src` agar cepat melihat siapa pengirimnya
- Klik kanan sebuah field → **Apply as Column** untuk menampilkan banyak nilai sekaligus
- Cross-check: MAC dari DHCP harus sama dengan MAC di Ethernet II paket korban
 
Claude finished the response
