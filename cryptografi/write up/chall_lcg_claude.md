# chall_lcg_claude - claude
## catatan 
ini adalah chall yang di buat oleh ai dan hanya bertujuan sebagai bahan latihan

## langkah penyelesaian

deskripsi chall :
tidak ada 

analisis:
source code:

```python
import secrets


MODULUS = 2_147_483_647
MULTIPLIER = 48271  # sekarang multiplier DIKETAHUI (dicetak ke player)
INCREMENT = secrets.randbelow(MODULUS - 1) + 1  # hanya ini yang rahasia
MENU = (
    "\n"
    "1. Get lucky number\n"
    "2. Predict next lucky number\n"
    "3. Exit\n"
    "> "
)


class LuckyEngine:
    def __init__(self, seed: int):
        self.seed = seed

    def next(self) -> int:
        self.seed = (MULTIPLIER * self.seed + INCREMENT) % MODULUS
        return self.seed


def main() -> None:
    flag = 'FLAG{rEP1n_sUK4_m1kU_:v}'#anggap tidak ada 
    lucky_engine = LuckyEngine(secrets.randbelow(MODULUS - 1) + 1)
    samples_left = 2  # cukup 2 sampel, bukan 3

    print(f"Modulus: {MODULUS}", flush=True)
    print(f"Multiplier: {MULTIPLIER}", flush=True)

    while True:
        print(MENU, end="", flush=True)
        try:
            choice = input().strip()
        except EOFError:
            break

        if choice == "1":
            if samples_left == 0:
                print("No more lucky numbers available", flush=True)
                continue
            samples_left -= 1
            print(f"Lucky number: {lucky_engine.next()}", flush=True)

        elif choice == "2":
            print("Your guess: ", end="", flush=True)
            try:
                guess = int(input().strip())
            except (ValueError, EOFError):
                print("Invalid number", flush=True)
                continue

            if guess == lucky_engine.next():
                print(f"Correct. {flag}", flush=True)
                break

            print("Wrong guess", flush=True)
            break

        elif choice == "3":
            print("Goodbye", flush=True)
            break

        else:
            print("Unknown option", flush=True)


if __name__ == "__main__":
    main()

```
dari source code kita tahu bahwa ini adalah chall interactive yang bisa kita jalan kan di terminal kita.jika kita jalankan di terminal maka akan muncul seperti ini :
<img width="863" height="187" alt="image" src="https://github.com/user-attachments/assets/e688afdc-f06c-4d0c-be41-14479cea12f7" />
dan jika kita memilih opsi 1 lebih dari 2 kali maka server tidak memberikan angka random lagi


<img width="257" height="403" alt="image" src="https://github.com/user-attachments/assets/028f5deb-ebdb-4213-8ec0-268530daacb5" />

dari source code dan dari terminal saat kita menjalankan file,kita di kasih tau modulus dan multiplier,serta ada nilai increment yang di sembunyikan.dari source code juga ada rumus ```(MULTIPLIER * self.seed + INCREMENT) % MODULUS``` yang dimana ini adalah rumus dari LCG(Linear Congruential Generator)

rumus : 

*x_next =(a * x_now + c)mod m*



x_next  = angka selanjutnya

a       = multiplier

x_now   = angka sekarang

c       = increment

m       = modulus


jadi untuk mengsolve chall yang diberikan claude ini pertama tama kita harus mencari nilai dari increment nya.karena server memberi kita 2 angka acak maka kita bisa masukan pada rumus

**x_2 =(a * x_1 + c)mod m**

jadi karena kita tahu x_2 dan x_1 dari server maka untuk mencari c kita bisa pakai rumus

**c =(x_2 - a * x_1 )mod m**

nah setelah dapat c kita baru bisa menebak angka yang akan muncul selanjutnya

solver : 
```python
from pwn import *

io = process(['python3','chall_LCG_claude.py'])

io.recvuntil(b"> " )
io.sendline(b"1")
io.recvuntil(b": ")
x1 = io.recvline().strip().decode()
print(f"x1 = {x1}")

io.recvuntil(b"> " )
io.sendline(b"1")
io.recvuntil(b": ")
x2 = io.recvline().strip().decode()
print(f"x2 = {x2}")

a = 48271
m = 2147483647

c = (int(x2) - a * int(x1))%m
xs = (a*int(x2)+int(c))%m

io.sendline(b"2")

io.sendlineafter(b": ",str(xs).encode())
io.interactive()
# rumus lcg
#x_selanjutnya = (a(multiplier).x_sekarang+c(increment)) mod m
# c = x2 - a.x1 mod m

```
**flag = FLAG{rEP1n_sUK4_m1kU_:v}**



