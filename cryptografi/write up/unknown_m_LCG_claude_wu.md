# unknown m LCG - chall claude
## deskripsi 
tidak ada karena ini chall dari claude ai dan aku tidak meminta deksripsi chall nya

##langkah penyelesaian

analisis source code:

```python3
from Crypto.Util.number import getPrime
import random

FLAG = b"FLAG{r3pyn_s4y4n9_m1ku}"#anggap tidak ada

m = getPrime(64)                 # rahasia
a = random.randrange(2, m)       # rahasia
c = random.randrange(1, m)       # rahasia
s = random.randrange(m)          # seed rahasia

def nxt():
    global s
    s = (a * s + c) % m
    return s

leak = [nxt() for _ in range(10)]          # output yang dibocorkan

ks = b"".join(nxt().to_bytes(8, "big") for _ in range(len(FLAG) // 8 + 1))
enc = bytes(f ^ k for f, k in zip(FLAG, ks))


print("leak =", leak)
print("enc  =", enc.hex())
```
output = 
```
leak = [12902449736011051051, 11982557265463608342, 10619194739834800954, 243016971156753384, 4154929254686691677, 8398750082937682266, 12846766113833796072, 568932804396179388, 11540648841373719468, 10113390186710605184]

enc  = 59bc398ae6396371f1386906b69e4159b35a3306776c5e
```

dari source code kita bisa tau bahwa flag di enkripsi dengan cara di xor-kan dengan ks atau keystream,dimana ks di buat dengan cara menyambung bytes dari angka yang di hasilkan oleh fungsi nxt() dimana fungsi nxt() menghasilkan angka random dengan LGC(Linear Congruential Generator).
saya tau nxt() membuat angka acak menggunakan LCG adalah dari rumus nxt() yang ada di source code ```s = (a * s + c) % m```.jadi untuk bisa mendapat flag,kita harus bisa mendapatkan nilai dari seed yang di gunakan di ks.

Di ks,seed yang di gunakan adalah seed ke 11 dan seterusnya,sedangkan yang ada di leak itu hanya seed 1 sampai 10.jadi untuk menebak seed yang di gunakan di ks kita perlu mengetahui a,c,dan m



solver :
```python
from functools import reduce
from math import gcd

leak = [684372622402415265, 5513535117005148630, 838092191466684301, 799802325057397708, 1018229783864238977, 12850993986433462488, 11929549474449280788, 14047364265113721813, 8523811814983631483, 9048674771774306676]
enc  = bytes.fromhex('848bb088029084870caceff9798881feb7f6d50678d3ad')
t=[]
for i in range(len(leak)-1):
    t.append(leak[i+1]-leak[i])
T = []
for i in range(len(t)-2):
    T.append(abs(t[i+2]*t[i]-t[i+1]**2))

m = reduce(gcd,T)
a = (leak[2]-leak[1])*pow(leak[1]-leak[0],-1,m) %m
c = (leak[1]-a*leak[0])%m

seed = leak[-1]
ks = b""
for _ in range(len(enc) // 8 + 1):
    seed = (a * seed + c) % m
    ks += (seed.to_bytes(8,"big"))
print(f'keystream = {ks}')

flag = bytes(x^y for x,y in zip(enc,ks))
print(flag.decode())

```


flag = FLAG{r3pyn_s4y4n9_m1ku}






## catatan claude:
Catatan
Hint (opsional): "Selisih antar output itu menarik."

Jika gcd(t0, m) != 1, pow(t0, -1, m) error. Cukup ganti pasangan t(i) yang dipakai, atau generate ulang challenge.

Jika GCD menghasilkan k·m (bukan m persis), bagi dengan faktor kecil, atau tambah jumlah output bocor (misal 20).

Tingkat kesulitan: mudah–sedang. Untuk versi lebih sulit, bocorkan hanya sebagian bit atas dari tiap output (truncated LCG), sehingga solusinya butuh lattice/LLL.
