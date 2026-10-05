# AgentSmith_Redlimit_CTF - WU

## deksripsi chall
Server web REDLIMIT dipindai oleh seseorang. Admin curiga penyerang meninggalkan pesan di salah satu request. Periksa log akses berikut dan temukan pesannya!

link chall(season 2) = https://hack.redlimit.id/challenges/

## langkah penyelesaian 

chall memberi kita sebuah file.log yang berisi hasil pemindaian server web dan kita di suruh untuk mencari pesan yan di tinggal kan oleh penyerang

jika kita buka langsung filenya maka akan kita bisa lihat ribuan log dan kita tidak mungkin bisa mencari satu persatu di antara ribuan log tersebut.
jadi aku mencoba dengan strings grep dan tidak dapat apa apa

<img width="509" height="336" alt="image" src="https://github.com/user-attachments/assets/9241927b-29e7-430a-a957-ac9bc08fb91e" />

jadi karena strings grep tidak menghasilkan apa apa jadi kemungkinan flag nya terenkripsi atau ter-encode.jadi kita harus mencari request yang berbeda dari yang lain,yaitu request yang muncul paling sedikit.Maka command nya adalah 

``` awk -F '"' '{print $6}' access.log | sort |uniq -c```

penjelasan=
- ```awk -F '"' '{print $6}' access.log``` = command untuk memisah sebuah text di suatu baris dengan patokan tanda " lalu menampilkan text yang ada di tanda " ke enam 
- ```sort``` = mengurutkan pesan yang sama
-  ```uniq -c``` = menghitung berapa banyak pesan yang sama

contoh 
baris yang ada di log = ```182.52.114.230 - - [20/Sep/2026:00:00:21 +0700] "GET /api/v1/status HTTP/1.1" 404 27765 "-" "python-requests/2.31.0"```

maka command ``` awk -F '"' '{print $6}' access.log | sort |uniq -c``` akan menampilkan kalimat yang ada di tanda kutip ke 6 yaitu ```python-requests/2.31.0``` dan ```sort``` akan mengurutkan text yang sama lalu  ```uniq -c``` akan menampilkan seberapa banyak text serupa
.maka pesan yang di tinggal kan penyerang kemungkinan adalah text yang paling sedikit muncul dari hasil ``` awk -F '"' '{print $6}' access.log | sort |uniq -c```

<img width="1103" height="374" alt="image" src="https://github.com/user-attachments/assets/4013c5fd-8fb7-4047-a9cd-74e7671b74f5" />

disana text yang paling sedikit muncul adalah UkVETElNSVR7dXMzcl80ZzNudF9jNHJyMTNzX3MzY3IzdHN9 dan jika kita decode base64 tersebut,kita akan mendapat flag

flag = REDLIMIT{us3r_4g3nt_c4rr13s_s3cr3ts}

## catatan dari claude

| Bagian | Kolom (`-F'"'`) | Contoh |
|---|---|---|
| Path / query string | `$2` | `GET /?msg=halo HTTP/1.1` |
| Referer | `$4` | `"http://pesan-rahasia.com"` |
| User-Agent | `$6` | `"Halo admin"` |

