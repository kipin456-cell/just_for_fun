# Git Forensics Cheat Sheet

Catatan perintah Git untuk menelusuri riwayat, melacak perubahan, dan memulihkan data yang "hilang".

---

## 1. Alur Investigasi Cepat

1. **Gambaran umum**: `git log --oneline --graph --all`
2. **Cek semua branch, tag, stash**: `git branch -a`, `git tag -l`, `git stash list`
3. **Cari kata kunci di seluruh history**: `git log -S "kata" --all -p`
4. **Telusuri file/baris tertentu**: `git blame`, `git log -p -- <file>`
5. **Lihat isi commit yang mencurigakan**: `git show <hash>`
6. **Cari yang terhapus/tersembunyi**: `git reflog`, `git fsck --lost-found`
7. **Bedah objek mentah**: `git cat-file -p <hash>`

---

## 2. `git log` : Membaca Riwayat

| Tujuan | Perintah |
|---|---|
| Ringkas, satu baris per commit | `git log --oneline` |
| Semua branch + visual graf | `git log --oneline --graph --all` |
| Lihat isi perubahan tiap commit | `git log -p` |
| Ringkasan file yang berubah | `git log --stat` |
| Filter per author | `git log --author="nama"` |
| Filter per rentang waktu | `git log --since="2024-01-01" --until="2024-02-01"` |
| Cari commit yang menambah/menghapus string | `git log -S "kata" -p` |
| Sama, tapi pakai regex | `git log -G "regex" -p` |
| Cari di pesan commit | `git log --grep="kata"` |
| Riwayat file, termasuk yang sudah dihapus | `git log --all --full-history -- <file>` |
| Ikuti file yang di-rename | `git log --follow -- <file>` |
| Info lengkap (author + committer) | `git log --format=fuller` |

**Kapan dipakai:** langkah pertama di hampir semua kasus. Perhatikan author, tanggal, dan pesan commit yang janggal.

---

## 3. `git blame` : Siapa Mengubah Baris Ini?

| Tujuan | Perintah |
|---|---|
| Petakan tiap baris ke commit + author | `git blame <file>` |
| Batasi rentang baris | `git blame -L 10,20 <file>` |
| Abaikan perubahan whitespace | `git blame -w <file>` |
| Deteksi baris yang dipindah/disalin antar file | `git blame -C <file>` |
| Blame pada versi commit tertentu | `git blame <hash> -- <file>` |

**Kapan dipakai:** kamu sudah tahu file/baris yang menarik dan ingin tahu commit asalnya.

---

## 4. `git show` : Membuka Satu Objek

| Tujuan | Perintah |
|---|---|
| Lihat commit (metadata + diff) | `git show <hash>` |
| Hanya daftar file yang berubah | `git show --stat <hash>` |
| Lihat isi file di commit tertentu | `git show <hash>:<path/file>` |
| Lihat tag (anotasi + commit) | `git show <tag>` |

**Kapan dipakai:** setelah `log` atau `blame` menunjuk commit yang mencurigakan.

---

## 5. `git diff` : Membandingkan Dua Titik

```bash
git diff <hash-lama> <hash-baru>              # semua perbedaan
git diff <hash-lama> <hash-baru> -- <file>    # hanya satu file
git diff <branch-a>..<branch-b>               # antar branch
```

**Kapan dipakai:** membandingkan dua versi untuk melihat apa persisnya yang berubah.

---

## 6. `git reflog` : Jejak Pergerakan HEAD

| Tujuan | Perintah |
|---|---|
| Riwayat pergerakan HEAD lokal | `git reflog` |
| Reflog semua ref | `git reflog show --all` |
| Pulihkan commit yang "hilang" jadi branch | `git branch pulih <hash>` |
| Pindah sementara ke commit tersebut | `git checkout <hash>` |

**Kapan dipakai:** setelah `reset --hard`, rebase, atau branch terhapus. Commit lama masih bisa dijangkau lewat reflog selama belum dibersihkan (garbage collection).

**Catatan:** reflog bersifat **lokal**. Repo hasil clone tidak membawa reflog dari repo asal.

---

## 7. `git fsck` : Mencari Objek Yatim

| Tujuan | Perintah |
|---|---|
| Cek integritas + tampilkan objek dangling | `git fsck` |
| Tampilkan objek yang tidak terjangkau | `git fsck --unreachable` |
| Simpan objek dangling ke `.git/lost-found/` | `git fsck --lost-found` |
| Abaikan reflog (cari yang benar-benar yatim) | `git fsck --no-reflogs --lost-found` |

**Kapan dipakai:** commit/blob sudah tidak ada di branch maupun reflog, tapi objeknya mungkin masih ada di disk.

**Catatan:** `git gc` dan `git prune` bisa menghapus objek yatim secara permanen.

---

## 8. `git cat-file` : Membedah Objek Mentah

| Tujuan | Perintah |
|---|---|
| Lihat tipe objek (blob/tree/commit/tag) | `git cat-file -t <hash>` |
| Lihat ukuran objek | `git cat-file -s <hash>` |
| Tampilkan isi objek | `git cat-file -p <hash>` |
| Daftar semua objek + tipenya | `git cat-file --batch-all-objects --batch-check` |

**Kapan dipakai:** kamu punya hash (dari `fsck`, `reflog`, atau file `.git/objects`) dan ingin tahu isinya.

---

## 9. Perintah Pendukung

```bash
# Cari string di SEMUA commit (bukan hanya working tree)
git grep "kata" $(git rev-list --all)

# Daftar semua hash commit
git rev-list --all

# Lihat struktur direktori di commit tertentu
git ls-tree -r <hash>

# Cek branch, tag, stash
git branch -a
git tag -l
git stash list
git stash show -p stash@{0}

# Lihat remote dan konfigurasi
git remote -v
git config --list
```

---

## 10. Kasus `.git` Terekspos di Web

```bash
# Rekonstruksi repo dari direktori .git yang bocor
git-dumper http://target/.git/ ./hasil

# Setelah itu, analisis seperti repo biasa
cd hasil
git log --oneline --all
git show <hash>
```

Alternatif: **GitTools** (`Dumper`, `Extractor`).

---

## 11. Scanner Secret Otomatis

```bash
# gitleaks
gitleaks detect --source . -v

# trufflehog
trufflehog git file://. 
```

**Tips:** bandingkan hasil scanner dengan temuan manualmu untuk melihat apa yang terlewat.

---

## 12. Hal yang Perlu Diingat

- **Author dan tanggal commit bisa dipalsukan.** Jangan jadikan satu-satunya bukti.
- Commit yang "dihapus" sering masih ada: cek `reflog`, `fsck`, dan branch/tag/stash lain.
- Secret yang pernah di-commit dianggap **bocor**, walaupun sudah dihapus di commit berikutnya. Lakukan rotasi.
- `git gc` dan `git prune` menghapus objek yatim. Backup dulu `.git` sebelum bereksperimen.
- Selalu cek `--all`, karena `git log` biasa hanya menampilkan branch yang sedang aktif.

---

## 13. Template Catatan per Soal

```markdown
## Nama Soal / Lab
- **Sumber:** 
- **Petunjuk awal:** 
- **Perintah (berurutan) + alasan:** 
- **Jalan buntu:** 
- **Konsep yang dipelajari:** 
```
