# Panduan SQL lnav untuk Analisis Log

Catatan pribadi: kolom, query siap pakai, dan penjelasannya untuk menganalisis **access log** (format nginx/Apache *combined*) dengan lnav.

---

## 1. Cara Pakai di lnav

| Langkah | Tombol / Perintah |
|---|---|
| Buka file | `TZ=UTC lnav namafile.log` |
| Masuk mode SQL | Tekan `;` lalu ketik query, Enter |
| Panggil query sebelumnya | `;` lalu `↑` (lalu edit seperlunya) |
| Kembali ke tampilan log | `q` (sekali saja; dua kali akan keluar dari lnav) |
| Lihat semua tabel dan kolom | `;.schema` |
| Hapus semua filter tampilan | `:reset-session` |

- Setiap query ditulis **dalam satu baris**.
- Teks (string) dalam query memakai tanda kutip tunggal: `'/login'`.
- `;` di awal adalah perintah lnav untuk masuk mode SQL, bukan bagian dari SQL.

---

## 2. Satu Baris Log dan Kolomnya

Contoh satu baris log:

```
203.0.113.45 - - [10/Nov/2026:02:15:30 +0000] "POST /api/login?next=/cart HTTP/1.1" 401 82 "https://tokokita.example.com/products" "python-requests/2.31.0"
```

### Kolom pada tabel `access_log`

| Kolom | Arti | Contoh dari baris di atas | Kegunaan dalam analisis |
|---|---|---|---|
| `log_time` | Waktu request | `2026-11-10 02:15:30` | Mencari kapan kejadian dimulai/berakhir, lonjakan per jam/menit |
| `c_ip` | IP pengunjung (*client*) | `203.0.113.45` | Mencari siapa pelaku, IP paling aktif, IP mencurigakan |
| `cs_method` | Metode request (`GET`, `POST`, ...) | `POST` | `POST` ke login/checkout sering jadi target serangan |
| `cs_uri_stem` | Halaman yang diminta, **tanpa** parameter | `/api/login` | Mengelompokkan per halaman, melihat halaman terbanyak/bermasalah |
| `cs_uri_query` | Parameter setelah tanda `?` | `next=/cart` | Mencari pola serangan (SQL injection, dll.) |
| `sc_status` | Kode jawaban server | `401` | Memisahkan sukses, kesalahan pengunjung, dan error server |
| `sc_bytes` | Ukuran jawaban (byte) | `82` | Menemukan unduhan besar, menghitung total traffic |
| `cs_referer` | Halaman asal pengunjung (`-` = tidak ada) | `https://tokokita.example.com/products` | Melihat dari mana trafik datang; `-` umum pada tool/bot |
| `cs_user_agent` | Identitas browser/program | `python-requests/2.31.0` | Membedakan manusia, bot, dan tool serangan |

### Kolom umum lnav (ada di semua format log)

| Kolom | Arti |
|---|---|
| `log_level` | Level yang dideteksi lnav (error, warning, info, ...) |
| `log_line` | Nomor baris pada log |

Daftar kolom lengkap yang tersedia di versi lnav Anda bisa dicek dengan `;.schema`.

### Arti awalan nama kolom

| Awalan | Arti |
|---|---|
| `c_` | *client*, data tentang pengunjung |
| `cs_` | *client to server*, yang dikirim pengunjung ke server |
| `sc_` | *server to client*, yang dikirim server ke pengunjung |

### Parameter URL

Parameter ditulis setelah `?` dengan pola `nama=nilai`, dipisah `&`.
Contoh: `/api/search?q=sepatu&page=2` → `cs_uri_stem` = `/api/search`, `cs_uri_query` = `q=sepatu&page=2`.

Karakter khusus ditulis dengan kode `%`: `%20` = spasi, `%27` = tanda kutip `'`.

### Arti kode status (`sc_status`)

| Kelompok | Arti | Contoh |
|---|---|---|
| `2xx` | Sukses | `200` OK |
| `3xx` | Dialihkan / tidak berubah | `301` pindah permanen, `304` pakai cache (bytes `0`) |
| `4xx` | Kesalahan dari sisi pengunjung | `401` belum berwenang, `403` dilarang, `404` tidak ditemukan, `429` terlalu banyak request |
| `5xx` | Kesalahan di sisi server | `500` error aplikasi, `502` bad gateway, `503` layanan tidak tersedia |

lnav biasanya mewarnai baris 4xx sebagai *warning* (kuning) dan 5xx sebagai *error* (merah). Tombol `w` dan `e` melompat ke baris-baris tersebut.

---

## 3. Kerangka Dasar SQL

```sql
SELECT   kolom, count(*)      -- apa yang ditampilkan
FROM     access_log           -- dari tabel mana
WHERE    kondisi              -- (opsional) saring baris SEBELUM dikelompokkan
GROUP BY kolom                -- (opsional) kelompokkan baris yang sama
HAVING   kondisi_hasil        -- (opsional) saring hasil SETELAH dikelompokkan
ORDER BY 2 DESC               -- (opsional) urutkan
LIMIT    10                   -- (opsional) batasi jumlah baris hasil
```

Urutan penulisan harus selalu seperti di atas. Bagian opsional boleh dibuang.

### Perintah dan fungsi

| Perintah/Fungsi | Fungsi | Contoh |
|---|---|---|
| `SELECT` | Memilih kolom yang ditampilkan | `SELECT c_ip, sc_status` |
| `FROM` | Menentukan tabel | `FROM access_log` |
| `WHERE` | Menyaring baris | `WHERE sc_status = 500` |
| `GROUP BY` | Mengelompokkan baris yang nilainya sama | `GROUP BY c_ip` |
| `HAVING` | Menyaring hasil pengelompokan | `HAVING count(*) >= 100` |
| `ORDER BY` | Mengurutkan (`DESC` besar→kecil, `ASC` kecil→besar) | `ORDER BY 2 DESC` |
| `LIMIT` | Membatasi jumlah baris hasil | `LIMIT 5` |
| `count(*)` | Menghitung jumlah baris | `SELECT count(*)` |
| `count(DISTINCT x)` | Menghitung nilai unik | `count(DISTINCT c_ip)` |
| `sum(x)` | Menjumlahkan | `sum(sc_bytes)` |
| `min(x)` / `max(x)` | Nilai terkecil / terbesar | `min(log_time)` |
| `round(x, n)` | Membulatkan ke n desimal | `round(x, 2)` |
| `strftime(f, x)` | Mengambil bagian waktu | `strftime('%H', log_time)` |
| `AS` | Memberi nama pada kolom hasil | `count(*) AS total` |

### Operator di `WHERE`

| Operator | Arti | Contoh |
|---|---|---|
| `=` / `!=` | Sama / tidak sama | `sc_status = 404` |
| `>` `>=` `<` `<=` | Perbandingan angka | `sc_status >= 500` |
| `BETWEEN a AND b` | Dalam rentang (termasuk a dan b) | `sc_status BETWEEN 400 AND 499` |
| `IN (...)` | Salah satu dari daftar | `sc_status IN (502, 503)` |
| `LIKE` | Mengandung pola teks (`%` = apa saja) | `cs_user_agent LIKE '%curl%'` |
| `AND` | Kedua syarat harus benar | `sc_status = 401 AND cs_uri_stem = '/login'` |
| `OR` | Salah satu syarat benar | `sc_status = 401 OR sc_status = 403` |
| `NOT` | Kebalikan syarat | `NOT cs_uri_stem LIKE '/static/%'` |

### Kode `strftime` yang sering dipakai

| Kode | Hasil | Contoh |
|---|---|---|
| `'%H'` | Jam | `02` |
| `'%H:%M'` | Jam:menit | `02:15` |
| `'%Y-%m-%d'` | Tanggal | `2026-11-10` |
| `'%Y-%m-%d %H'` | Tanggal dan jam | `2026-11-10 02` |

---

## 4. Query Siap Pakai

Salin query (termasuk `;` di awal), tempel di lnav, lalu ubah bagian yang perlu.

### 4.1 Gambaran Umum

**Total baris log**
```sql
;SELECT count(*) FROM access_log
```
Menghitung semua baris log.

**Rentang waktu log**
```sql
;SELECT min(log_time), max(log_time) FROM access_log
```
Waktu baris paling awal dan paling akhir.

**Jumlah IP unik**
```sql
;SELECT count(DISTINCT c_ip) FROM access_log
```
Berapa pengunjung berbeda (berdasarkan IP).

**Lihat 10 baris pertama sebagai contoh**
```sql
;SELECT * FROM access_log LIMIT 10
```
`*` berarti semua kolom. Berguna untuk mengenali isi tabel.

### 4.2 Status Code

**Jumlah per status**
```sql
;SELECT sc_status, count(*) AS total FROM access_log GROUP BY sc_status ORDER BY total DESC
```
Distribusi status. Sekilas terlihat apakah error banyak.

**Jumlah per kelompok status (2xx, 4xx, 5xx)**
```sql
;SELECT sc_status / 100 AS kelompok, count(*) AS total FROM access_log GROUP BY kelompok
```
Pembagian bilangan bulat: `200 / 100` menjadi `2`, `404 / 100` menjadi `4`.

**Persentase error server (5xx)**
```sql
;SELECT round(100.0 * sum(sc_status >= 500) / count(*), 2) AS persen_5xx FROM access_log
```
Seberapa besar bagian request yang gagal karena server.

**Daftar request yang error 5xx**
```sql
;SELECT log_time, c_ip, cs_method, cs_uri_stem, sc_status FROM access_log WHERE sc_status >= 500
```
Siapa, kapan, dan halaman mana yang memicu error server.

**Halaman yang paling sering 404**
```sql
;SELECT cs_uri_stem, count(*) AS total FROM access_log WHERE sc_status = 404 GROUP BY cs_uri_stem ORDER BY total DESC LIMIT 10
```
Halaman yang dicari tetapi tidak ada. Bisa tautan rusak, atau pencarian oleh scanner.

### 4.3 Analisis IP

**10 IP paling aktif**
```sql
;SELECT c_ip, count(*) AS total FROM access_log GROUP BY c_ip ORDER BY total DESC LIMIT 10
```
IP dengan request terbanyak. Belum tentu penyerang (bisa health check, bot, atau pengunjung biasa).

**IP dengan error 4xx terbanyak**
```sql
;SELECT c_ip, count(*) AS total FROM access_log WHERE sc_status BETWEEN 400 AND 499 GROUP BY c_ip ORDER BY total DESC LIMIT 10
```
IP yang sering ditolak server. Kandidat kuat pelaku brute force atau scanner.

**IP yang banyak meminta halaman berbeda (ciri scanner)**
```sql
;SELECT c_ip, count(DISTINCT cs_uri_stem) AS jumlah_halaman, count(*) AS total FROM access_log GROUP BY c_ip ORDER BY jumlah_halaman DESC LIMIT 10
```
Pengunjung biasa mengakses sedikit halaman, sedangkan scanner mencoba puluhan path berbeda.

**IP dengan minimal 100 request**
```sql
;SELECT c_ip, count(*) AS total FROM access_log GROUP BY c_ip HAVING total >= 100 ORDER BY total DESC
```
`HAVING` menyaring hasil setelah dikelompokkan. Ubah angka 100 sesuai kebutuhan.

**Semua aktivitas satu IP**
```sql
;SELECT log_time, cs_method, cs_uri_stem, sc_status FROM access_log WHERE c_ip = '203.0.113.50'
```
Mengikuti jejak satu IP dari awal sampai akhir. Ganti alamat IP-nya.

### 4.4 Analisis Halaman

**5 halaman paling sering diakses**
```sql
;SELECT cs_uri_stem, count(*) AS total FROM access_log GROUP BY cs_uri_stem ORDER BY total DESC LIMIT 5
```
Pakai `cs_uri_stem` (bukan gabungan dengan parameter) agar hitungan tidak terpecah.

**Jumlah per metode (GET/POST)**
```sql
;SELECT cs_method, count(*) AS total FROM access_log GROUP BY cs_method
```
Lonjakan `POST` yang tidak wajar bisa berarti serangan atau spam.

**Tanpa file statis**
```sql
;SELECT cs_uri_stem, count(*) AS total FROM access_log WHERE cs_uri_stem NOT LIKE '/static/%' AND cs_uri_stem NOT LIKE '/images/%' GROUP BY cs_uri_stem ORDER BY total DESC LIMIT 10
```
Menyembunyikan CSS/JS/gambar agar fokus ke halaman dan API.

### 4.5 Analisis Waktu

**Request per jam**
```sql
;SELECT strftime('%H', log_time) AS jam, count(*) AS total FROM access_log GROUP BY jam
```
Pola jam sibuk dan sepi.

**Error 5xx per jam**
```sql
;SELECT strftime('%H', log_time) AS jam, count(*) AS total FROM access_log WHERE sc_status >= 500 GROUP BY jam ORDER BY total DESC
```
Jam dengan gangguan paling parah.

**Error 5xx per menit (zoom ke satu insiden)**
```sql
;SELECT strftime('%H:%M', log_time) AS menit, count(*) AS total FROM access_log WHERE sc_status >= 500 GROUP BY menit
```
Melihat tepatnya kapan gangguan mulai dan berhenti.

**Awal dan akhir suatu jenis error**
```sql
;SELECT min(log_time), max(log_time), count(*) FROM access_log WHERE sc_status IN (502, 503)
```
Durasi insiden dan jumlah request yang terdampak.

**Request dalam rentang waktu tertentu**
```sql
;SELECT log_time, c_ip, cs_uri_stem, sc_status FROM access_log WHERE log_time >= '2026-11-10 13:00:00' AND log_time < '2026-11-10 14:00:00'
```
Menyorot satu jam kejadian. Ganti tanggal dan jamnya.

### 4.6 Traffic (Bytes)

**Total traffic dalam MB**
```sql
;SELECT round(sum(sc_bytes) / 1048576.0, 2) AS total_mb FROM access_log
```
1 MB = 1.048.576 byte. Untuk GB, bagi dengan `1073741824.0`.

**IP dengan traffic terbesar**
```sql
;SELECT c_ip, round(sum(sc_bytes) / 1048576.0, 2) AS mb FROM access_log GROUP BY c_ip ORDER BY mb DESC LIMIT 5
```
Berguna menemukan pengunduh data besar.

**Request dengan respons terbesar**
```sql
;SELECT log_time, c_ip, cs_uri_stem, sc_bytes FROM access_log ORDER BY sc_bytes DESC LIMIT 10
```
File atau respons paling besar yang dikirim server.

### 4.7 User-Agent dan Bot

**User-Agent paling umum**
```sql
;SELECT cs_user_agent, count(*) AS total FROM access_log GROUP BY cs_user_agent ORDER BY total DESC LIMIT 10
```
Browser biasanya diawali `Mozilla/5.0`. Nama seperti `python-requests`, `curl`, `sqlmap`, `Googlebot` menandakan program.

**Yang bukan browser biasa (contoh filter)**
```sql
;SELECT c_ip, cs_user_agent, count(*) AS total FROM access_log WHERE cs_user_agent NOT LIKE 'Mozilla%' GROUP BY c_ip, cs_user_agent ORDER BY total DESC
```
Perhatikan: bot resmi seperti Googlebot juga diawali `Mozilla/5.0`, jadi tidak muncul di sini.

### 4.8 Deteksi Keamanan

**Brute force login (banyak 401)**
```sql
;SELECT c_ip, count(*) AS gagal FROM access_log WHERE sc_status = 401 AND cs_uri_stem LIKE '%login%' GROUP BY c_ip ORDER BY gagal DESC LIMIT 10
```
IP dengan kegagalan login terbanyak. Setelah ketemu, cek apakah akhirnya berhasil (status 200) dengan query "Semua aktivitas satu IP".

**Percobaan SQL injection pada parameter**
```sql
;SELECT log_time, c_ip, cs_uri_query, sc_status FROM access_log WHERE cs_uri_query LIKE '%UNION%' OR cs_uri_query LIKE '%DROP%' OR cs_uri_query LIKE '%SLEEP%' OR cs_uri_query LIKE '%OR%'
```
Mencari kata kunci khas SQL injection. `LIKE` tidak membedakan huruf besar/kecil. Hasil `500` dari request ini perlu diselidiki lebih serius.

**Probing file sensitif**
```sql
;SELECT c_ip, cs_uri_stem, sc_status FROM access_log WHERE cs_uri_stem LIKE '%.env%' OR cs_uri_stem LIKE '%.git%' OR cs_uri_stem LIKE '%phpmyadmin%' OR cs_uri_stem LIKE '%wp-admin%'
```
Path yang tidak seharusnya diminta pengunjung biasa. Status `403` berarti ada, tetapi dilarang; `200` berarti file terbuka dan berbahaya.

**Pembatasan laju (429)**
```sql
;SELECT c_ip, count(*) AS total, min(log_time), max(log_time) FROM access_log WHERE sc_status = 429 GROUP BY c_ip ORDER BY total DESC
```
IP yang diblok karena terlalu banyak request, beserta rentang waktunya.

---

## 5. Ekspor Hasil (Non-interaktif)

Jalankan di terminal biasa (bukan di dalam lnav):

```bash
# Simpan hasil query ke CSV
lnav -n -c ':write-csv-to /tmp/error5xx.csv' \
        -c ';SELECT * FROM access_log WHERE sc_status >= 500' namafile.log

# Jalankan query dan cetak hasilnya di terminal
lnav -n -c ';SELECT sc_status, count(*) FROM access_log GROUP BY sc_status' namafile.log
```

- `-n` = tanpa antarmuka interaktif.
- `-c` = jalankan perintah, boleh diulang beberapa kali.

---

## 6. Alur Investigasi Singkat

1. **Gambaran umum:** jumlah baris, rentang waktu, distribusi status.
2. **Cari anomali:** status 5xx dan 4xx, IP teratas, lonjakan per jam.
3. **Persempit:** per menit pada jam yang bermasalah, atau semua aktivitas satu IP.
4. **Pahami pola:** halaman yang diminta, User-Agent, parameter.
5. **Simpulkan:** apa yang terjadi, kapan, siapa/apa yang terdampak, dan rekomendasi.

---

## 7. Kesalahan Umum

| Masalah | Penyebab / Solusi |
|---|---|
| Hasil hanya sedikit baris | Masih ada `WHERE` dari query sebelumnya, atau filter tampilan aktif. Hapus `WHERE`, atau jalankan `:reset-session` |
| `no such table` atau `no such column` | Salah ketik nama; cek dengan `;.schema` |
| Teks tidak cocok | String wajib memakai kutip tunggal `'...'`, bukan kutip ganda |
| Terkeluar dari lnav | `q` ditekan dua kali; satu kali sudah cukup untuk kembali dari hasil SQL |
| Jam tidak sesuai | Jalankan dengan `TZ=UTC lnav ...` jika log bertanda `+0000` |
| Hitungan terpecah-pecah | Gunakan `cs_uri_stem` (bukan gabungan dengan parameter) untuk `GROUP BY` |

---

## 8. Ringkasan Cepat (Contekan)

```
SELECT kolom, count(*) FROM access_log
WHERE ...  GROUP BY kolom  ORDER BY 2 DESC  LIMIT 10
```

| Pertanyaan | Kolom yang dipakai |
|---|---|
| Siapa? | `c_ip`, `cs_user_agent` |
| Kapan? | `log_time` |
| Apa yang diminta? | `cs_method`, `cs_uri_stem`, `cs_uri_query` |
| Hasilnya apa? | `sc_status` |
| Seberapa besar? | `sc_bytes` |
| Dari mana? | `cs_referer` |

Catatan: query di panduan ini sudah disusun mengikuti sintaks lnav/SQLite, tetapi perilaku bisa sedikit berbeda antarversi lnav. Jika ada query yang tidak berjalan, periksa nama kolom dengan `;.schema`.
