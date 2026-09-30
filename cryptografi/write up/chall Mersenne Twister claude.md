# chall Mersenne Twister claude - Write up

## deskripsi chall
tidak ada karena ini adalah chall dari ai


## langkah penyelesaian

**analisis**

source code : 
```python3
import random

FLAG = "LKS{mt19937_state_bisa_di_clone}"

rng = random.Random()  # seed acak, kamu tidak tahu seed-nya

print("=== Tebak Angka Random ===")
print("Ini 624 angka pertama dari generator:")
for _ in range(624):
    print(rng.getrandbits(32))

print("\nTebak 3 angka berikutnya untuk dapat flag!")
for i in range(3):
    try:
        tebakan = int(input(f"Angka ke-{i+1}: "))
    except ValueError:
        print("Input harus angka.")
        raise SystemExit
    if tebakan != rng.getrandbits(32):
        print("Salah! Coba lagi.")
        raise SystemExit

print("Benar semua!", FLAG)

```
jika kita jalankan kode dari source code di terminal maka kita akan mendapat 624 angka random dan kita disuruh untuk menebak 3 angka selanjutnya untuk mendapat kan flag

<img width="492" height="172" alt="image" src="https://github.com/user-attachments/assets/2cc5ee50-6fdb-4667-bb55-93981f0b99c8" />

di source code bisa kita lihat bahwa angka acak yang muncul adalah dari PRNG Marsene Twister ``` print(rng.getrandbits(32))``` jadi kita bisa langsung menebak 3 angka selanjutnya dengan randcrack.karena syarat randcrack yang sudah terpenuhi(624 angka 32 bit) maka kita tinggal eksekusi randcrack nya.


kenapa kita memerlukan angka 32 bit output sebanyak 624?karena untuk mencari angka selanjutnya kita harus mengetahui state,state ini adalah angka random 32 bit sebelum di acak oleh shift dan xor.jadi untuk menghitung angka yang akan muncul selanjutnya kita harus menncari state,oleh karena itu kita harus tau nilai dari 624 angka yang telah di acak secara berurutan.  



solver : 
```python
from pwn import *
from randcrack import RandCrack

io = process(['python3','/mnt/d/py_repin/chall_Mersenne_Twister_claude.py'])
rc = RandCrack()

io.recvline()
io.recvuntil(b":\n")

angka_624 = []
while len(angka_624) < 624:
    angka_624.append(int(io.recvline().strip()))

for x in angka_624:
    rc.submit(x)

for _ in range(3):
    io.recvuntil(b": ")
    io.sendline(str(rc.predict_getrandbits(32)).encode())

io.interactive()
```

**flag** = LKS{mt19937_state_bisa_di_clone}
