# Chall LKSP Lucky Number Generator - Write Up

## Kategori 
Cryptografi

## deskripsi 
tebak tebakan dikit gak ngaruh lah ya :p

connection_info: nc 52.221.202.190 9998

## author: lordrukie

## langkah penyelesaian
**analisis**

source code:
```python
import secrets


MODULUS = 2_147_483_647
MULTIPLIER = secrets.randbelow(MODULUS - 2) + 2
INCREMENT = secrets.randbelow(MODULUS - 1) + 1
MENU = (
    "\n"
    "1. Get lucky number\n"
    "2. Predict next lucky number\n"
    "3. Exit\n"
    "> "
)


def load_flag() -> str:
    with open("flag.txt", "r", encoding="utf-8") as flag_file:
        return flag_file.read().strip()


class LuckyEngine:
    def __init__(self, seed: int):
        self.seed = seed

    def next(self) -> int:
        self.seed = (MULTIPLIER * self.seed + INCREMENT) % MODULUS
        return self.seed


def forge_lucky_engine() -> LuckyEngine:
    return LuckyEngine(secrets.randbelow(MODULUS - 1) + 1)


def main() -> None:
    flag = load_flag()
    lucky_engine = forge_lucky_engine()
    samples_left = 3

    print(f"Modulus: {MODULUS}", flush=True)

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

dari deskripi chall kita di kasih sebuah intance dan port dan saat kita nc ke intance dan port tersebut maka akan muncul sebuah tebak tebakan angka sederhana

<img width="917" height="552" alt="image" src="https://github.com/user-attachments/assets/ba57c6eb-e733-42de-94c5-0c72ba95fcb1" />

Jadi dari intance dan port tadi saat kita jalankan di terminal akan muncul sebuah mini game tebak tebakan angka sederhana dimana kita bisa meminta "lucky number" dan menebak angka selajutnya untuk mendapatkan flag tapi kita cuma diberi 3 kali kesempatan untuk mendapat "lucky number" 

karena lucky number -nya di generate menggunakan LCG (sc = ```(MULTIPLIER * self.seed + INCREMENT) % MODULUS```) jadi kita bisa menebak angka selanjutnya menggunakan rumus 

*x_next =(a * x_now + c)mod m* 

tapi dari source code dan intance kita hanya di beri x1,x2,x3 dan modulus(m) jadi untuk menebak angka selanjutnya kita harus mendapatkan a dan c terlebih dahulu

untuk mencari a kita bisa menggunakan rumus

*x2 =(a * x1 + c)mod m*

*x3 =(a * x2 + c)mod m*

x3 - x2 = (a * x2-x1 ) mod m

pindah ruaskan a

*a = (x3 - x2) * (x2 - x1)**-1 mod m) mod m*

setelah dapat a kita bisa menghitung c menggunakan rumus 

*c = (x2 - a * x1)mod m*

baru setelah mendapat mendapat a dan c kita baru bisa menebak angka selanjutnya dengan

*x_next =(a * x_now + c)mod m* 

**solver :**

```python3
from pwn import *

INTANCE , PORT = "localhost",9998
io = remote(INTANCE,PORT)
def get_x():
    io.recvuntil(b"> ")
    io.sendline(b"1")
    io.recvuntil(b"Lucky number: ")
    x = int(io.recvline().strip())
    return x

def send_x_next(x_next):
    io.recvuntil(b"> ")
    io.sendline(b"2")
    io.sendline(str(x_next).encode())
    flag = io.recvline().strip()
    return flag.decode()

m  = 2147483647
x1 = get_x()
x2 = get_x()
x3 = get_x()

print(f"x1 = {x1}")
print(f"x2 = {x2}")
print(f"x3 = {x3}")

a = ((x3 - x2)*pow((x2-x1),-1,m))%m
c = (x2-a*x1)%m
x_next = (a*x3+c)%m
flag = send_x_next(x_next)
print(flag)

```

flag : 

<img width="818" height="116" alt="image" src="https://github.com/user-attachments/assets/e3abe575-18bd-4743-8cc4-7e27664c5db1" />








