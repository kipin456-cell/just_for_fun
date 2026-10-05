# disk, sleuth!2_picoCTF - Forensics

## deksripsi chall
All we know is the file with the flag is named down-at-the-bottom.txt...

dds2-alpine.flag.img.gz

Hints=
- 1.The sleuthkit has some great tools for this challenge as well.
- 2.Sleuthkit docs here are so helpful: TSK Tool Overview
- 3.This disk can also be booted with qemu!

## langkah penyelesaian
pertama kita harus ekstrak file yang kita dapatkan karena file tersebut di kompres.setelah itu kita buka terminal dan masuk ke folder hasil ekstrak dan kita akan pertama tama harus memastikan bahwa file tersebut bisa kita solver menggunakan tool sleuthkit

<img width="1067" height="84" alt="image" src="https://github.com/user-attachments/assets/14a9e45b-c19f-4933-9156-5db8b8929d80" />

jadi langsung saja kita cek dengan ```mmls``` untuk mengetahui ada di opset ke berapa folder linux

<img width="714" height="205" alt="image" src="https://github.com/user-attachments/assets/b00ab11c-9bf2-43ab-9316-9a4c0776d81f" />

setelah tau folder linux ada di opset ke berapa,kita tinggal mencari file yang bernama "down-at-the-bottom.txt" karena chall menyuruh kita untuk melakukan nya.

<img width="727" height="87" alt="image" src="https://github.com/user-attachments/assets/b30bae24-4d90-461c-a102-28b57a9501d1" />

penjelasan = 
- ```fls``` = command di tool sleuth kit untuk menampilkan folder dan file yang ada
-  ```-r ``` = = rekursif.yang artinya command sleuth kit kita akan masuk ke subfolder yang ada
-  ```-o 2048 dds2-alpine.flag.img``` = opset dari folder linux yang kita dapat dari mmls(angka 2048 di dapat dari kolom start)
-  ```| grep "down-at-the-bottom.txt"``` = command supaya saat kita enter output nya hanya akan menapilkan down-at-the-bottom.txt ada di opset keberapa

setelah tau file down-at-the-bottom.txt ada di mana,kita tinggal membukanya dengan command 

<img width="722" height="277" alt="image" src="https://github.com/user-attachments/assets/2fb1d9a3-9980-4466-be66-f708304e80ea" />

**flag = academy{f0r3ns1c4t0r_n0v1c3_95c645ad}**
