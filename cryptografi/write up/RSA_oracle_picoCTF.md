# RSA_oracle_picoCT - write up


## deskripsi chall
Can you abuse the oracle?

An attacker was able to intercept communications between a bank and a fintech company. They managed to get the message
(ciphertext) and the password
that was used to encrypt the message.
Instance
Additional details appear once you launch your instance.
Hints :

1. Crytography Threat models: chosen plaintext attack.

2. OpenSSL can be used to decrypt the message. e.g openssl enc -aes-256-cbc -d ...

3. The key to getting the flag is by sending a custom message to the server by taking advantage of the RSA encryption algorithm.

4. Minimum requirements for a useful cryptosystem is CPA security.

## langkah penyelesaian 
**analisis :**

di deskripsi chall kita di kasih file secret.enc dan password.enc yang dimana kedua file ini adalah hasil dari enkripsi.di deskripsi chall juga kita di kasih sebuah intance yang jika kita remote maka akan muncul sebuah server yang bisa mengenkripsi dan mendekripsi input kita

<img width="339" height="155" alt="image" src="https://github.com/user-attachments/assets/96a5b3c6-8e09-461a-b69e-69b7d7221175" />

tapi jika kita masukan isi dari password.enc maka server akan menolak nya

<img width="709" height="178" alt="image" src="https://github.com/user-attachments/assets/3e137e87-8c27-4c66-a844-751465afdfcf" />

dari hint chall,kita di beri tahu ```OpenSSL can be used to decrypt the message. e.g openssl enc -aes-256-cbc -d``` dimana command ini adalah command untuk mendekripsi kan sebuah file yang dienkripsi pakai AES jadi bisa kita simpulkan bahwa secret.enc adalah file yang dienkripsi menggunakan AES dan password.enc adalah key dari enkripsi tersebut.tapi saat ku buat password.enc jadi bytes hasilnya adalah

<img width="955" height="65" alt="image" src="https://github.com/user-attachments/assets/b508a4d6-23a2-4d1f-a4e4-0871f7a74b91" />

jadi bisa kita simpulkan password.enc adalah key dari AES yang di enkripsi pakai rsa.Dikarenakan di AES key yang digunakan untuk enkripsi sama dengan key yang di gunakan unutuk dekripsi jadi untuk mendekripsi kan secret.enc yang dienkripsi pakai AES kita harus mendapat key yang dienkripsi pakai RSA di password.enc


Karena kita di kasih server yang bisa mengenkripsi dan mendekripsi input kita dan jika kita masukann angka dari password.enc secara langsung server menolak,jadi kita harus menyamarkan nilai dari password.enc tadi untuk menipu server dan mendekripsi kan input kita yang berupa nilai password.enc yang telah kita modifikasi.lalu setelah mendapat deskripsi nilai dari password.enc modifikas dari server kita tinggal menghilangkan modifikasinya(karena kita tahu nilai dari angka yang memodifikasi angka password.enc) supaya kita bisa mendapat deskripsi dari password.enc. 
**
*c_modif =c.r*

keterangan : 

- c_modif = angka dari password.enc yang kita ubah untuk mengelabui server
- c = angka dari password.enc yang di tolak server
- r = angka yang kita gunakan untuk merubah c (kita tahu nilai nya)

jadi setelah kita kirim c_modif ke server kita bisa mendapat deskripsi dari c_modif dan dikarenakan kita tau nilai yang memodifikasi c maka kita tinggal menghilangkan hasil dari modifikasi untuk mendapat massage yang telah di deskripsikan server.untuk menghilangkan modifikasi di c_modif kita tidak bisa membagi biasa angka c_modif dengan angka r,karena c_modif itu modulus dari n.jadi kita harus menginverse nilai deskripsi dari angka r lalu mengalikannya dengan m_modif untuk menghilangkan nilai modifikasi nya


*m_modif = c_modif ^d mod n*

*m = m_modif * (D(r),-1,mod n)*

penjelasan :

- m_modif = hasil deskripsi server dari C_modif kita
- m = pesan yang kita cari
- D(r) = deskripsi dari r(nilai yang kita pakai untuk memodifikasi c)

ken


