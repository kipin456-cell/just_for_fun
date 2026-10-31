# The Sleuth Kit (TSK) — Cheatsheet Lengkap

Referensi command-line untuk digital forensics pakai The Sleuth Kit (TSK). Disusun berdasarkan alur kerja: identifikasi image → partisi → filesystem → file → timeline → hash.

---

## 0. Alur Kerja Umum

```
1. img_stat / mmls   → kenali image & layout partisi
2. fsstat            → kenali filesystem di tiap partisi
3. fls                → list file & direktori (termasuk yang terhapus)
4. icat / istat       → ekstrak & inspeksi file lewat inode
5. fls -m + mactime   → bikin timeline aktivitas
6. hfind              → cek hash file vs database (NSRL)
7. tsk_recover        → recover semua file sekaligus
```

---

## 1. Image Layer — Info Dasar Image

### `img_stat`
Menampilkan detail teknis file image (format, ukuran sektor, dll).
```bash
img_stat image.dd
img_stat -t image.dd        # tampilkan tipe image saja
```

### `img_cat`
Ekstrak data mentah dari image (raw bytes), berguna buat convert format.
```bash
img_cat image.E01 > image.dd
```

---

## 2. Volume/Partition Layer

### `mmls`
Menampilkan tabel partisi (MBR/GPT) — WAJIB dijalankan pertama buat tahu offset tiap partisi.
```bash
mmls image.dd
mmls -t gpt image.dd         # paksa tipe partition table
```
Output penting: kolom **Start** (offset dalam sector) dipakai di command selanjutnya.

### Hitung offset byte dari sector
```bash
# offset byte = start_sector * sector_size (biasanya 512)
echo $((2048 * 512))
```

---

## 3. File System Layer

### `fsstat`
Detail filesystem (tipe, ukuran blok, metadata filesystem).
```bash
fsstat image.dd
fsstat -o 2048 image.dd       # kalau image berisi banyak partisi, pakai offset (sector) dari mmls
```

---

## 4. File Name Layer

### `fls`
List file & direktori — termasuk file yang **sudah dihapus** (ditandai `*`).
```bash
fls image.dd
fls -o 2048 image.dd          # dengan offset partisi
fls -r image.dd                # rekursif ke semua subdirektori
fls -p image.dd                 # tampilkan full path
fls -d image.dd                 # tampilkan file yang terhapus saja
fls -m / image.dd > bodyfile.txt   # output format mactime (buat timeline)
```

### `ffind`
Cari nama file dari nomor inode.
```bash
ffind image.dd 1234
ffind -o 2048 image.dd 1234
```

---

## 5. Metadata Layer (Inode)

### `istat`
Detail metadata lengkap dari sebuah inode (timestamp MAC, ukuran, block pointer, dll).
```bash
istat image.dd 1234
istat -o 2048 image.dd 1234
```
Berguna buat lihat **MACB times** (Modified, Accessed, Changed, Born/Created).

### `icat`
Ekstrak isi file berdasarkan nomor inode (termasuk file terhapus kalau blok belum ketimpa).
```bash
icat image.dd 1234 > hasil_file.bin
icat -o 2048 image.dd 1234 > hasil_file.bin
```

### `ifind`
Cari nomor inode dari nama file atau dari offset data block.
```bash
ifind -n /path/to/file image.dd
ifind -d 5000 image.dd          # cari inode pemilik block tertentu
```

### `icat` + `file` (kombinasi umum)
```bash
icat image.dd 1234 | file -       # identifikasi tipe file hasil ekstrak
```

---

## 6. Data Unit Layer (Block-level)

### `blkls`
List/ekstrak blok data mentah (termasuk slack space & unallocated space).
```bash
blkls image.dd > unallocated.dd
blkls -A image.dd > unallocated_only.dd   # hanya block yang unallocated
```
File hasil `blkls` sering di-`strings`/`grep`/`binwalk` buat cari data tersembunyi.

### `blkcat`
Ekstrak isi dari block number tertentu.
```bash
blkcat image.dd 5000
blkcat -o 2048 image.dd 5000
```

### `blkstat`
Detail status sebuah block (allocated/unallocated).
```bash
blkstat image.dd 5000
```

### `blkcalc`
Konversi nomor block antara image asli dan image hasil `blkls -A` (dipakai kalau korelasi lokasi data terhapus).
```bash
blkcalc -u 100 image.dd
```

---

## 7. Journal Layer (khusus filesystem ber-journal, misal ext3/ext4, NTFS)

### `jls`
List entry journal filesystem.
```bash
jls image.dd
```

### `jcat`
Ekstrak isi block journal tertentu.
```bash
jcat image.dd 15
```

---

## 8. Timeline Analysis

### Alur bikin timeline (paling sering dipakai di CTF forensik)
```bash
# 1. Generate body file dari semua partisi yang relevan
fls -r -m / image.dd > bodyfile.txt
# kalau ada beberapa partisi:
fls -r -m / -o 2048 image.dd >> bodyfile.txt

# 2. Convert body file ke timeline human-readable
mactime -b bodyfile.txt -d > timeline.csv

# 3. Filter timeline berdasar range tanggal tertentu
mactime -b bodyfile.txt 2024-01-01..2024-01-31 -d > timeline_jan.csv
```

### `mactime`
```bash
mactime -b bodyfile.txt                # tampilkan ke stdout, format teks
mactime -b bodyfile.txt -d              # output CSV
mactime -b bodyfile.txt -y              # format tanggal ISO 8601
mactime -b bodyfile.txt -z Asia/Jakarta # set timezone
```

---

## 9. Hash & File Identification

### `hfind`
Lookup hash file terhadap database hash (misal NSRL — buat exclude file-file OS standar yang "aman").
```bash
hfind -i nsrl-md5 nsrl_hashset.txt          # index database dulu
hfind nsrl_hashset.txt <MD5_HASH>            # lookup satu hash
```

### `md5sum` / `sha1sum` (built-in Linux, sering dikombinasi)
```bash
icat image.dd 1234 | md5sum
```

---

## 10. Recovery & Sorting Tools

### `tsk_recover`
Recover/ekstrak seluruh file dari image sekaligus ke direktori output.
```bash
tsk_recover -a image.dd output_folder/       # -a = recover file yang masih allocated
tsk_recover -o 2048 image.dd output_folder/  # dengan offset partisi
```

### `sorter`
Kategorikan file hasil ekstraksi berdasarkan tipe (gambar, dokumen, dll) — berguna buat triase cepat.
```bash
sorter -d output_sorted/ image.dd
```

---

## 11. Signature & Carving

### `sigfind`
Cari pola binary signature (magic bytes) tertentu dalam image — berguna buat file carving manual.
```bash
sigfind -t jpeg image.dd            # cari signature JPEG
sigfind -o 0xFFD8 image.dd          # cari byte pattern custom
```

---

## 12. Opsi Global yang Sering Dipakai

| Opsi | Fungsi |
|------|--------|
| `-o <sector>` | Offset partisi (sector), hasil dari `mmls` |
| `-f <fstype>` | Paksa tipe filesystem (ntfs, ext4, fat32, dll) |
| `-i <imgtype>` | Paksa tipe image (raw, ewf, aff) |
| `-r` | Rekursif |
| `-p` | Tampilkan full path |
| `-v` | Verbose |
| `-z <timezone>` | Set timezone untuk timestamp |

---

## 13. Quick Reference — Skenario CTF Umum

**"Cari file yang dihapus"**
```bash
fls -rd image.dd          # list file terhapus
icat image.dd <inode> > recovered_file
```

**"Buat timeline aktivitas user"**
```bash
fls -r -m / image.dd > body.txt
mactime -b body.txt -d > timeline.csv
```

**"Cari data tersembunyi di unallocated space"**
```bash
blkls image.dd > unalloc.dd
strings unalloc.dd | grep -i "flag"
binwalk unalloc.dd
```

**"Ekstrak semua file sekaligus"**
```bash
tsk_recover -a image.dd output/
```

**"Identifikasi partisi & filesystem sebelum mulai"**
```bash
mmls image.dd
fsstat -o <offset_dari_mmls> image.dd
```

---

## Catatan Instalasi

```bash
# Debian/Kali/Ubuntu
sudo apt install sleuthkit

# Cek versi
tsk_recover -V
```

Semua tool di atas dirancang buat dipakai bersama (chained), bukan satu tool serba bisa — identifikasi layer yang relevan dulu (image → volume → filesystem → file → block) sebelum pilih command yang tepat.
