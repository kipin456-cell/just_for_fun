# chall_stego_3 - Claude

## dekskripsi chall
kategori = forensic
Sebuah foto pemandangan air tenang dikirim oleh seorang kurir tanpa pesan apa pun. Pemeriksaan metadata dan string tidak menemukan apa-apa, dan alat scan otomatis yang biasa dipakai juga gagal. Namun kurir bersumpah gambar ini membawa sesuatu. Temukan pesannya, lalu pastikan kamu membacanya dengan benar.

hint = 
- fokus di channel Green, dengan Extract By: Column. Bit plane-nya tinggal kamu cari sendiri di antara 1-7, dan hanya satu yang menghasilkan teks rapi
## langkah penyelesaian 
karena ini adalah chall latihan yang di berikan claude dan ku suruh untuk membuatkan chall yang bertujuan untuk latihan menggunakan stegsolve jadi aku langsung membuka stegsolver untuk menganalisis chall yang di berikan.

<img width="716" height="555" alt="image" src="https://github.com/user-attachments/assets/4e3a5cf9-0c34-44c2-94d7-1c52cf20ab0f" />

dari gambar yang diberikan hanya ada sebuah gambar yang tidak apa apa.jadi saat ku zsteg juga tidak muncul apa apa

<img width="393" height="109" alt="image" src="https://github.com/user-attachments/assets/16a83994-cf9a-494e-99fb-c98a7a8aa43b" />

jadi karena dari hint kita di suruh untuk mencari di bagian green jadi diantar bit plane 1-7 dan order setting nya column.jadi setelah kku coba di bit plain ke 1 dan order column yang ku dan ku geser keatas hasilnya adalah base64 

<img width="727" height="551" alt="image" src="https://github.com/user-attachments/assets/10c4d553-063e-4c91-88eb-531307b80583" />

dan ketika kita decode kita baru dapat flagnya

<img width="820" height="494" alt="image" src="https://github.com/user-attachments/assets/d31bfeb0-8e73-424a-919d-bddb9e5b20ca" />
