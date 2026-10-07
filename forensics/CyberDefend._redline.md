# redline_CyberDefend - WU

## link chall 

https://cyberdefenders.org/blueteam-ctf-challenges/redline

## langkah penyelesaian
1.soal = What is the name of the suspicious process?

jawaban = oneetx.exe

langkah =
pertama kita bisa mencari file mencurigakan dengan command voltra3 ```vol -f "file.mem" windows.pstree > pstree.txt```
kita check dulu yang mana kemunkinan file mencurigakan nya dengan ```cat pstree.txt | grep -Ei "temp|Appdata|Downloads|User|public|ProgramData"``` dan hasil yang muncul adalah

<img width="1363" height="205" alt="image" src="https://github.com/user-attachments/assets/1c11dc13-d8d5-4fbb-be75-3fae79385643" />

yang dimana file oneetx.exe muncul dua kali dan di directory ```AppData\local\temp\c3912af058\oneetx.exe``` dimana ini sangat mencurigakan karena ada di dalam folder dengan nama aneh dan di dalam temp.jadi ketika ku masukan ke soal jawabannya benar.

<img width="641" height="151" alt="image" src="https://github.com/user-attachments/assets/39a917fb-67a8-42fa-8d6d-ffc4dc1396e8" />

---

2.soal = What is the child process name of the suspicious process?

jawaban =  rundll32.exe

langkah = 

untuk mencari 'child' dari file yang mencurigakan artinya kita harus mencari file yang yang berasal dari oneetx.exe,untuk mencari nya kita harus mencari program exe yan yang PPID(Parent Program ID)-nya dari PID(Program ID) oneetx.exe dimana kita bisa pakai grep untuk mencari PPID nya 

<img width="1234" height="106" alt="image" src="https://github.com/user-attachments/assets/194640b6-975f-4e14-afa8-3d8d1ae35944" />

jadi child dari oneetx.exe adalah rundll32.exe

3.soal = What is the memory protection applied to the suspicious process memory region?

jawaban = PAGE_EXECUTE_READWRITE

langkah = 

untuk melihat protection di volatility3 kita bisa menggunakan command ```vol -f "file.mem windows.malfind > malfind.txt"``` lalu buka file nya.supaya 
output yang di berikan langsung di bagian oneetx.exe kita bisa pakai grep

<img width="1264" height="88" alt="image" src="https://github.com/user-attachments/assets/0c187c74-3bfb-4787-a52f-03bfb57c6069" />

untuk bagian protection nya ada di teks yang ke-6

4. soal = What is the name of the process responsible for the VPN connection?

jawaban = Outline.exe

langkah = 

untuk mencari process pusat yang bertanggung jawab atas vpn kita harus mencari program yang bernama tun2socks.exe,karena program tersebut adalah VPN yang berjalan.lalu kita lihat di bagian PPID untuk menemukan program yang menjalankan VPN.

<img width="1092" height="326" alt="image" src="https://github.com/user-attachments/assets/e1290f63-a38f-4b68-9dc2-f831a1455a00" />

5. soal = What is the attacker's IP address?

jawaban =  77.91.124.20

langkah = 

untuk mecari ip dari penyerang kita bisa menggunakan command ```vol -f 'MemoryDump.mem' windows.netscan > netscan.txt``` dan tinggal kita buka file netscan.txt dengan cat lalu grep nama file mencurigakan supaya langsung memunculkan ip dari penyerang

<img width="1091" height="311" alt="image" src="https://github.com/user-attachments/assets/143d7f64-0f31-4673-b8ba-0478d80bdba9" />

6. soal = What is the full URL of the PHP file that the attacker visited?

jawaban = http://77.91.124.20/store/games/index.php

langkah = untuk mendapat link yang di buka oleh attacker kita bisa strings untuk menampilkan text mentah (pasti berisi url) dan kita grep dengan ip penyerang yang sudah kita dapat agar output yang di berikan terminal langsung menampilkan url apa saja yang di akses ip tersebut

<img width="596" height="371" alt="image" src="https://github.com/user-attachments/assets/e2f514ac-b3bc-48f2-8a05-560167ff89a0" />

7. soal = What is the full path of the malicious executable?

jawaban =  c:\Users\Tammam\AppData\Local\Temp\c3912af058\oneetx.exe

langkah = 

untuk mencari path dari file yang mencurigakan kita bisa menggunakan command ```vol -f "MemoryDump.mem" windows.filescan > filescan.txt``` dan kita tinggal cat dan grep nama file mencurigakan nya untuk langsung mendapat path dari file tersebut

<img width="970" height="191" alt="image" src="https://github.com/user-attachments/assets/139fc57b-69e4-4c27-b499-f5630dd3bf2c" />





