# Common modulus attack Write-Up / claude
## catatan
chall ini dibuat ai
**Kategori:** crypto 
**Tingkat Kesulitan:** Medium

## 1. Deskripsi Tantangan
tidak ada deskripsi karena chall dari ai

## 2. Langkah-Langkah Penyelesaian
1. **Analisis Awal**
   source code:
   ```python
    from Crypto.Util.number import getPrime, bytes_to_long
    
    FLAG = b"CTF{d3m0_fl4g_g4nti_saat_deploy}"
    
    def gen_keys(bits=1024):
        p = getPrime(bits // 2)
        q = getPrime(bits // 2)
        n = p * q
        return n
    
    def encrypt(m, e, n):
        return pow(m, e, n)
   
    def main():
        # Modulus dibuat SENGAJA lebih kecil (512-bit) dan flag dipadding
        # supaya m^e1 (e1=3) melebihi n -> hasil enkripsi benar2 "membungkus"
        # (wrap around modulo), sehingga akar pangkat 3 biasa TIDAK cukup
        # dan Common Modulus Attack sungguhan diperlukan.
        n = gen_keys(bits=512)

    # Dua public exponent yang BERBEDA tapi coprime satu sama lain,
    # dipakai untuk mengenkripsi pesan yang SAMA dengan modulus N yang SAMA.
    e1 = 3
    e2 = 65537

    # Padding di depan flag supaya m cukup besar sehingga m^e1 > n
    m = bytes_to_long(b"PADDING_BIAR_PANJANG_" + FLAG)
    if m >= n:
        raise ValueError("Flag terlalu panjang untuk modulus ini")
    assert pow(m, e1) > n, "m^e1 masih < n, low-exponent attack masih bisa dipakai!"

    c1 = encrypt(m, e1, n)
    c2 = encrypt(m, e2, n)

    with open("output.txt", "w") as f:
        f.write(f"n  = {n}\n")
        f.write(f"e1 = {e1}\n")
        f.write(f"e2 = {e2}\n")
        f.write(f"c1 = {c1}\n")
        f.write(f"c2 = {c2}\n")

    print("[+] output.txt berhasil dibuat")
    

    if __name__ == "__main__":
        main()
   ```
output:

n  = 8116853386096482397453056739055220209443721295865462685427972588270635065141818346338879518010691722840215355226602186793733126935061012848693818440519383

e1 = 3

e2 = 65537

c1 = 1127420969645058237490738491811195571934962762890973084108854689712951196098908937629334761909644867757722028152233432637845665939799278604401535621052225

c2 = 6622745364486873207289248738658625121070658961211100528911496770440739681356065939800341171189002318542641942843254576752545184081841731360511458991746842

dari source code kita tau bahwa c1 dan c2 adalah hasil enkripsi dari plaintext yang sama yaitu flag,tapi c1 menggunakan e1 = 3 dan c2 menggunakan e2 = 65537.
dan keduannya sama sama menggunakan n yang sama.oleh karena itu chall ini memiliki kerentanan Common Modulus Attack.kenapa?
karena:
1.pesan kita di enkripsi 2 kali menggunakan n yang sama
2.nilai gcd(e1,e2) = 1 yang artinya coprime 

jadi dengan 2 poin di atas kita bisa mendapatkan flag tanpa harus mencari p dan q.cara adalah dengan identitas bezout.dimana

**a . e1 + b . e2 = 1 mod n**

jadi untuk mendapat persamaan tersebut kita harus mencari nilai dari a dan b dengan extended GCD,yang nantinya 
dari extend GCD ini kita bisa mendapat a dan b dari bozout.setelah dapat a dan b selanjutnya adalah mendapatkan flag dengan
membuat pangkat c jadi identitas bozeout.karena saat dicari dengan extended GCD nilai a atau b pasti bernilai negatif maka pertama-tama
kita harus me-inverse modulo c yang pangkatnya negatif(misalkan a) supaya saat di kali nilai pangkatnya menjadi **a . e1 + b . e2 = 1 mod n**

**($$c1 ^ {-1} $$) ^ $${-a}$$ + ($$c2 ^ {b}$$)) mod n**

**($$m ^ {e1 . a}$$) + ($$m ^ {e2 . b}$$) mod n** 

yang nanti nya akan menjadi $$m^{1}$$ karena indentitas bozout

$$m^{1}$$ mod_n


3. **solver:**
```python
from gmpy2 import iroot
from Crypto.Util.number import long_to_bytes
from sympy import gcdex
c1 = 1127420969645058237490738491811195571934962762890973084108854689712951196098908937629334761909644867757722028152233432637845665939799278604401535621052225
e1 = 3
n = 8116853386096482397453056739055220209443721295865462685427972588270635065141818346338879518010691722840215355226602186793733126935061012848693818440519383
c2 = 6622745364486873207289248738658625121070658961211100528911496770440739681356065939800341171189002318542641942843254576752545184081841731360511458991746842
e2 = 65537

def Common_modulus_attck(c1,e1,c2,e2,n):
    a,b,g = gcdex(e1,e2)
    a = int(a)
    b = int(b)
    if a < 0:
        c1 = pow(c1,-1,n)
        a = -a
    if b < 0 :
        c2 = pow(c2,-1,n)
        b = -b
    m = (pow(c1,a,n) * pow(c2,b,n)) %n
    return m

m = Common_modulus_attck(c1,e1,c2,e2,n)
print(long_to_bytes(m))
```
flag =


<img width="391" height="49" alt="image" src="https://github.com/user-attachments/assets/b9eef39d-1566-4444-bfa3-b91ce1d31142" />

