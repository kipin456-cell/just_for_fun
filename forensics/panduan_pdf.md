# PDF Forensics CTF — Latihan Bertahap

13 challenge PDF, diurutkan dari yang paling mudah. Format flag: `CTF{...}`.

Kerjakan berurutan. Kalau stuck lebih dari 20–30 menit, baca bagian **Hint** di level tersebut,
dan kalau masih buntu baru buka `writeups.md`.

## Cek flag tanpa melihat solusi

Setiap flag punya hash SHA-256. Cek flag kamu dengan:

```bash
echo -n 'CTF{flag_kamu}' | sha256sum
```

| #  | File         | SHA-256 dari flag |
|----|--------------|-------------------|
| 01 | chall01.pdf  | 875047ae60fcfc3c975b1a85d2983249f9dce7cc22db303e17d8712372dc391f |
| 02 | chall02.pdf  | 9cfdc6e461cc612b953329f6dc424fee90a86ae55dcbf322270ef0fa34e1126b |
| 03 | chall03.pdf  | ddd181893eee83fe6c10e6442a7f6922a4afe4aec1fb1ace59414c8119e978e0 |
| 04 | chall04.pdf  | 93e533a80ced837c4cc3a2f00d57b10a37959654b52b90f82a596febd08daacb |
| 05 | chall05.pdf  | 95a7da1ccacd56f5175b81cd6730fc4e5d1bb28f0aa513da684ddd0d6f521274 |
| 06 | chall06.pdf  | ff1202b07e71a70faaa2aec1904acc0c492a33c901dd9cd32ba651f0ef020ccd |
| 07 | chall07.pdf  | 5d30a13e42f734126c00920e379afee98b6a5d75f2cdb70ddc0145cc0ea04b78 |
| 08 | chall08.pdf  | 5cb60dd18b8b8b0b8fea65a01b5d03ad64308f8c663b05025c71361b170af1a9 |
| 09 | chall09.pdf  | de57c9dc83102ff0313f49ea5c0019f1b37f98e4ab1942cdd07a0f132a2e3fce |
| 10 | chall10.pdf  | 2ff161d18c11a31f7a49883f1d584c0301ce2ba5234cccd4844e4cafabb0049a |
| 11 | chall11.pdf  | e30cc53821ae11e8812c684cfd5c9c8f221f0e9b8c3eff682f5b908db58d5284 |
| 12 | chall12.pdf  | 540018039a4bab57ab136e779db9074c1fb2116c8b07bc324b1af247a0cfbc45 |
| 13 | chall13.pdf  | 9b616a7d1e135677ce23b1624e56ec0897d700e43bcda2b7444234ecd2fbfc7b |

## Setup tools

Linux (Kali/Ubuntu/WSL):

```bash
sudo apt install poppler-utils qpdf binwalk libimage-exiftool-perl unzip john
pip install pikepdf pypdf pillow
# opsional
sudo gem install zsteg
# pdf-parser.py (Didier Stevens) -> https://github.com/DidierStevens/DidierStevensSuite
```

**Aturan penting:** selalu kerjakan di VM / mesin latihan. Semua file di sini aman, tapi
kebiasaan membuka PDF tak dikenal di mesin utama itu buruk untuk dibawa ke event sungguhan
(terutama challenge yang berisi JavaScript — di Level 3 sebaiknya analisis tanpa membuka di Adobe Reader).

---

## Level 1 — Easy (pemanasan)

Tujuannya: membiasakan diri bahwa **yang kamu lihat di layar ≠ isi sebenarnya dari file**.

| #  | Soal |
|----|------|
| 01 | Seorang intern mengirim draft laporan kuartal. Halamannya membosankan, tapi mungkin ia ceroboh. |
| 02 | Pengumuman yang "sengaja dikosongkan". Benarkah kosong? |
| 03 | Berkas personel yang sudah "disensor" sebelum dikirim. Apakah sensornya benar-benar bekerja? |
| 04 | Dokumen ini hanya menampilkan sebagian halamannya. Cari tahu apa yang ada di luar jendela. |

**Hint Level 1:** `pdfinfo`, `exiftool`, `strings`, `pdftotext`, Ctrl+A di viewer, `pdftoppm`.

## Level 2 — Medium

Tujuannya: memahami **struktur** PDF — lampiran, urutan halaman, data di luar struktur, layer.

| #  | Soal |
|----|------|
| 05 | Laporan insiden dengan "bukti pendukung". Di mana buktinya? |
| 06 | Laporan 6 halaman. Setiap halaman terlihat biasa, tetapi urutannya tampak janggal. |
| 07 | Brosur produk biasa. Ukuran file-nya sedikit lebih besar dari seharusnya. |
| 08 | Dokumen publik. Menurut pembuatnya, ada layer lain yang tidak untuk publik. |

**Hint Level 2:** `pdfdetach`, `pdftotext -layout`, `file`, `binwalk`, `unzip -l`, `qpdf --qdf --object-streams=disable`,
panel Layers di Firefox/Okular/Adobe.

## Level 3 — Hard

Tujuannya: **membedah file PDF di level objek/byte**, dan menggabungkan beberapa teknik.

| #  | Soal |
|----|------|
| 09 | Arsip dokumen. Halamannya jelas tidak menyimpan semuanya. Ada data yang tidak pernah ditampilkan. |
| 10 | Memo rahasia yang sudah "dihapus kontennya". Tapi file ini pernah disimpan lebih dari sekali. |
| 11 | Dokumen yang mengaku "terverifikasi". Lihat apa yang sebenarnya dijalankan saat dibuka (jangan dijalankan!). |
| 12 | Dokumen terkunci. Kata sandinya tidak sulit; sudah disediakan `wordlist.txt` untuk membantu. |
| 13 | Foto dokumentasi yang tampak biasa. Mungkin yang disimpan bukan hanya piksel. |

**Hint Level 3:** `pdf-parser.py`, `qpdf --qdf`, mencari jumlah `%%EOF`, decode base64/hex/XOR,
`pdf2john` + `john`, `pdfimages`, `zsteg` atau script Python kecil untuk LSB.

---

## Setelah selesai

Kalau ke-13-nya sudah selesai, lanjut berlatih dengan topik: PDF yang header/xref-nya rusak,
font/ToUnicode yang dimanipulasi, XMP metadata, form field (AcroForm), dan analisis stream dengan `peepdf`.
