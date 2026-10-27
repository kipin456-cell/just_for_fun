131# catatan cyberdefends - Web_Investigation_Lab

## pertanyaan 
1. By knowing the attacker's IP, we can analyze all logs and actions related to that IP and determine the extent of the attack, the duration of the attack, and the techniques used. Can you provide the attacker's IP?
2. If the geographical origin of an IP address is known to be from a region that has no business or expected traffic with our network, this can be an indicator of a targeted attack. Can you determine the origin city of the attacker?
3. Identifying the exploited script allows security teams to understand exactly which vulnerability was used in the attack. This knowledge is critical for finding the appropriate patch or workaround to close the security gap and prevent future exploitation. Can you provide the vulnerable PHP script name?
4. Establishing the timeline of an attack, starting from the initial exploitation attempt, what is the complete request URI of the first SQLi attempt by the attacker?
5. Can you provide the complete request URI that was used to read the web server's available databases?
6. Assessing the impact of the breach and data access is crucial, including the potential harm to the organization's reputation. What's the table name containing the website users data?
7. The website directories hidden from the public could serve as an unauthorized access point or contain sensitive functionalities not intended for public access. Can you provide the name of the directory discovered by the attacker?
8. Knowing which credentials were used allows us to determine the extent of account compromise. What are the credentials used by the attacker for logging in?
9. We need to determine if the attacker gained further access or control of our web server. What's the name of the malicious script uploaded by the attacker?

## jawaban 
1. 111.224.250.131

- penjelasan = ketika kita buka chall menggunakan wireshark dan kita lihat klik statistic -> converstion maka akan terlihat ip 111.224.250.131 yang berkomunikasi dengan 73.124.22.98 dan jumlah packages nya paling banyak,jadi ip attacker adalah 111.224.250.131
  <img width="1102" height="116" alt="image" src="https://github.com/user-attachments/assets/b59d7322-5d07-4253-8f57-c1d5d1337dba" />

2. Shijiazhuang

- penjelasan = setelah dapat ip dari nomor 1,untuk tau kota dari ip tersebut kita bisa menggunakan web https://www.geolocation.com
  <img width="1365" height="485" alt="image" src="https://github.com/user-attachments/assets/1ee5fda5-0db6-40cb-b3ff-bf33bcbc9fcd" />

3.search.php

- penjelasan = ketika kita cek filter dengan ```http contains ".php"``` maka akan muncul banyak packages search.php.dan ketika kita lihat clue nya,kita disuruh menganalisis file yang paling banyak muncul 
  <img width="1310" height="369" alt="image" src="https://github.com/user-attachments/assets/03c54f80-2ef1-4e55-8fd9-384b6972dff5" />
  
4./search.php?search=book and 1=1; -- -

- penjelasan = soal menyuruh kita mecari apa payload SQLI pertama yang digunakan di server,dan jika kita lihat di  wireshark maka payload pertama nya adalah
  <img width="1302" height="364" alt="image" src="https://github.com/user-attachments/assets/8922186b-7e31-499a-a82f-e3b965d47691" />
  dan jika kita copy payload tsb dan decode dengan cyberchef menggunakan recipe url maka kita akan dapat /search.php?search=book and 1=1; -- -

5. /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,schema_name)),0x7176706a71) FROM INFORMATION_SCHEMA.SCHEMATA-- -

- penjelasan = karena kita disuruh mencari payload URI yang di gunakan untuk membaca isi dari data base.jika kita filter frame contains "INFORMATION_SCHEMA" maka akan muncul
  <img width="891" height="371" alt="image" src="https://github.com/user-attachments/assets/1d236177-dece-4eda-91a7-e7e974b63672" /> dimana jika kita decode baris URI tersebut dan mengirim nya ke soal,maka akan benar

6. customers

- penjelasan = setelah kita follow stream di soal nomor 5 dan terus pergi ke stream selanjutnya maka kita akan melihat
  <img width="1361" height="624" alt="image" src="https://github.com/user-attachments/assets/ef134098-6e74-47fa-b1eb-4c494fb60a0d" />
  tepatnya di stream ke 154 kita akan melihat tabel yang berisi "admin", "books", "customers" dan karena soal menanyakan tabel mana yang berisi informasi users jadi tabel yang kita kirim ke soal adalah tabel "customers"

7. /admin/

- penjelasan = soal bertanya tentang hidden directory yang memiliki akses yang tidak seharusnya bisa diakses publik.jadi bisa kita simpulkan bahwa directory itu adalah akses untuk kita bisa mengotak atik database.karena di clue kita disuruh menggunakan filter ```http.request.method == POST```.kenapa karena POST fungsinya untuk mengirim sebuah login atau semacamnya,jadi jika di logikakan setelah penyerang tau aksesnya maka ia pasti akan langsung login menggunakan akun tersebut.
  <img width="1365" height="188" alt="image" src="https://github.com/user-attachments/assets/087a0693-1228-4b03-abe0-60939ef856f8" />

8. admin:admin123!

- penjelasan = dari filter ```http.request.method == POST``` jika kita follow maka kita akan melihat username yang dikirim attacker dan passwordnya.jadi kita tinggal mengcheck stream selanjutnya sampai menemukan login admin yang berhasil(di labs ini ada di stream ke 674). 
  <img width="829" height="452" alt="image" src="https://github.com/user-attachments/assets/afdd5c12-a5b4-4195-b66a-a14e4eac9c8c" />

9. NVri2vhp.php

- penjelasan = untuk melihat file yang di upload ke web,kita bisa menggunakan ```http.request.method == POST``` dan kita lihat ke riwayat yang berisi login admin yang sukses.dibawahnya ada paket yang panjangnya lebih besar dari packages diatasnya.dan karena directory admin punya pasti akses untuk mengupload file ke database,maka file tersebut di upload /admin/  
  <img width="913" height="548" alt="image" src="https://github.com/user-attachments/assets/800bdf8f-0676-433b-b3e6-e88649a4b4b7" />


## Alur umum investigasi web (pcap)
1. IP penyerang: Statistics → Conversations
2. Kota/asal IP: situs geolocation
3. Script rentan: http contains ".php" → yang paling sering muncul
4. SQLi pertama: ip.src==IP && http.request.uri contains "search.php", ambil frame paling awal, URL decode
5. Baca database: frame matches "(?i)information_schema" (schemata=database, tables=tabel, columns=kolom)
6. Tabel user: cari hasil query tables → tabel berisi data user (customers/users)
7. Hidden directory: POST ke /xxx/ (atau GET 404 → 200/301)
8. Kredensial: POST login → bandingkan response gagal vs sukses
9. Upload shell: POST multipart + filename="*.php" + application/x-php

## Cheat sheet filter
- http.request.method == POST
- ip.src == X && http.request
- frame matches "(?i)kata"
- http.cookie / http.set_cookie
- http.response.code == 200
- Follow → HTTP Stream
