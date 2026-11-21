# Hidden_in_the_Ports - THPCTF

## deskripsi chall 

Seorang admin lupa menutup sebuah service rahasia di server perusahaan. Service tersebut tidak berjalan di port yang umum, dan petunjuk untuk mengaksesnya tersebar di lebih dari satu tempat.

Tugasmu: lakukan reconnaissance terhadap server berikut, temukan service yang tersembunyi, lalu kumpulkan seluruh bagian flag.

Server: 70.70.70.150

Tools yang disarankan: nmap, curl, netcat Format flag: THPCTF{...}

**author** = **AosLie** 

## Analisis 

di deksripsi chall ada sebuah ip server dan tertulis bahwa ada seorang admin yang lupa menutup sebuah service rahasia yang dimana serveice tersebut ada di port yang tidak umum dan ada petunjuk yang tersebar untuk mengakses nya.
jadi tugas kita adalah menginvestigasi dan menemukan service tersembunyi tersebut lalu mengumpulkan seluruh bagian flag.jadi dari sini bisa kita simpulkan bahwa flag tersebar jadi beberapa bagian,yang kemungkinan ada yang berada di service rahasia.

karena di deskripsi chall juga menyebutkan bahwa ada port rahasia yang lupa di tutup jadi sesuai dengan petunjuk soal,kita bisa menggunakan tools nmap untuk scanning dan mencari port yang terbuka

<img width="634" height="207" alt="image" src="https://github.com/user-attachments/assets/2a9cf877-8d4e-40cd-8415-3925c90b7198" />

dari hasil nmap,bisa kita bisa lihat bahwa ada 3 port yang status nya terbuka,jadi kita bisa menjalankan command ```nmap -sV -p- 2121 8888 31337 70.70.70.150``` untuk memastikan dan melihat versi service dari port yang terbuka

<img width="1911" height="872" alt="image" src="https://github.com/user-attachments/assets/ac20e20d-ef1e-4886-b729-ae86f5a385ab" />

yang dimana kita bisa lihat di bagian version dari tiap service,tapi yang aneh ada di port 31337 dimana tidak muncul versinya.jadi aku coba dengan command ```nc 70.70.70.150 31337``` untuk check apakah port yang terbuka tersebut bisa di akses pakai netcat.

<img width="700" height="159" alt="image" src="https://github.com/user-attachments/assets/399eb3c1-bebe-4b99-b36e-70d213179dbc" />

ternyata ketika ku jalankan commandnya akan muncul part ke 1 dari flag dan perintah untuk mengirim help,tapi ketika ku kirim help kita malah di suruh mengirim sebuah password.karena ada masih ada 2 port terbuka yang belum ku investigasi maka kita coba check port port tersebut untuk mencari petunjuk. 

di port 8888 cukup menarik,karena service dari port tersebut adalah http (Apache) jadi aku coba menggunakan command ```curl -i 70.70.70.150:8888  ``` untuk melihat response http nya (seperti yang di rekomendasikan oleh deskripsi chall).

<img width="968" height="205" alt="image" src="https://github.com/user-attachments/assets/05a9db05-b276-44b9-a431-6946131c3b48" />

jadi yang kita dapat "n0t_4ll_p0rts_4r3_sc4nn3d" yang ada dikolom X-Passphrase,dimana ini kemungkinan adalah password yang di minta oleh service yang ada di port 31337 tadi.jadi kita coba masukan password tersebut ke service tadi

<img width="711" height="139" alt="image" src="https://github.com/user-attachments/assets/0e151fd4-c283-4d29-9c1a-ae351de8a8a3" />

ternyata benar,pasword tersebut adalah password yang diminta dan kita sudah mendapat 2 bagian flag

## FLAG = THPCTF{nmap_full_port_scan_is_the_key_7c3e}
