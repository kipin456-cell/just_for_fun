# catatan - picoCTF_Collaborative_Development

## deskripsi chall 
My team has been working very hard on new features for our flag printing program! I wonder how they'll work together?

You can download the challenge files here:

## langkah - langkah 

- pertama setelah kita ekstrak file nya maka kita akan mendapat folder .git dan flag.py dan ketika kita buka flag.py nya cuma ada kata
  ```python
   print("Printing the flag...")
  ```
  jadi aku coba jalankan ```git status``` untuk cek file yang dimodifikas yang hasilnya
  <img width="842" height="149" alt="image" src="https://github.com/user-attachments/assets/518fe9f4-5997-435b-8f3c-bd7862a38cea" />

- jadi aku coba menggunakan command ```git log``` untuk melihat history dan git show untuk melihat apa isi yang diubah dari flag.py tapi hasilnya sama aja
  <img width="875" height="414" alt="image" src="https://github.com/user-attachments/assets/ac897b86-2c47-4405-8aed-55c11100c049" />

- jadi karena nama dari chall nya adalah "Collaborative Development" jadi aku coba menggunaka ```git branch``` dan aku mengetahui bahwa repository git itu punya branch lebih dari 1
  <img width="652" height="87" alt="image" src="https://github.com/user-attachments/assets/187456ed-5c5d-4a74-8f19-3dfa85b1e25f" />
  jadi aku coba menampilkan isi dari masing masing branch dengan menggunakan ```git show <nama branch>``` dan di tiap branch berisi flag yang di pecah sesuai nama branch nya (part1,part2,part3)

  <img width="1110" height="563" alt="image" src="https://github.com/user-attachments/assets/e5dead66-7dcd-4aa6-b128-be3efe409923" />
