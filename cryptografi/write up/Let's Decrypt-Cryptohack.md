# Let's Decrypt - Cryptohack
## deskripsi chall
If you can prove you own CryptoHack.org, then you get access to one of our secrets.

Connect at socket.cryptohack.org 13391

Challenge files:
  - 13391.py

## langkah langkah penyelesaian
##analisis
source code:
```python
#!/usr/bin/env python3

import re
from Crypto.Hash import SHA256
from Crypto.Util.number import bytes_to_long, long_to_bytes
from utils import listener
from pkcs1 import emsa_pkcs1_v15
# from params import N, E, D

FLAG = "crypto{?????????????????????????????????}"

MSG = 'We are hyperreality and Jack and we own CryptoHack.org'
DIGEST = emsa_pkcs1_v15.encode(MSG.encode(), 256)
SIGNATURE = pow(bytes_to_long(DIGEST), D, N)


class Challenge():
    def __init__(self):
        self.before_input = "This server validates domain ownership with RSA signatures. Present your message and public key, and if the signature matches ours, you must own the domain.\n"

    def challenge(self, your_input):
        if not 'option' in your_input:
            return {"error": "You must send an option to this server"}

        elif your_input['option'] == 'get_signature':
            return {
                "N": hex(N),
                "e": hex(E),
                "signature": hex(SIGNATURE)
            }

        elif your_input['option'] == 'verify':
            msg = your_input['msg']
            n = int(your_input['N'], 16)
            e = int(your_input['e'], 16)

            digest = emsa_pkcs1_v15.encode(msg.encode(), 256)
            calculated_digest = pow(SIGNATURE, e, n)

            if bytes_to_long(digest) == calculated_digest:
                r = re.match(r'^I am Mallory.*own CryptoHack.org$', msg)
                if r:
                    return {"msg": f"Congratulations, here's a secret: {FLAG}"}
                else:
                    return {"msg": f"Ownership verified."}
            else:
                return {"error": "Invalid signature"}

        else:
            return {"error": "Invalid option"}


import builtins; builtins.Challenge = Challenge # hack to enable challenge to be run locally, see https://cryptohack.org/faq/#listener
listener.start_server(port=13391)



````
pubkey dan signature :
```
{'N': '0x80a5b8245992f64d2d11f1279a88d23dd76594fa0a37427e55482c819fe6eb727f7cde0a715c018f8e3c98cc721ff1f779e8c704f3ad2ce877eaecf680c7105685581ac1dc8d5040d602a47d8dba333e787599a7528154f8f72581f1119c1c1267078e8bfb4e2a67b00ad13827660381b8105590a72b7d1fa45bdd2c07cc12ab2d8f7c6578178e93aac1a29ccdd1ecbeeafa25fa425988376384f745a7613af437ad235826d7d1499a0b2e83401ce0781fb85aa801ae9c43a9f79b2bd4bc8817a2a0725272e847e3b397e1d27b86fa0ba038b2704534a98a12e5e133f2358521ee0ee59a892928543c4a71460cadc2c89e011b019c2094ca6d17a0c143dc9297',

'e': '0x10001',

'signature': '0x55c231eebc642cd1e44199e10937ee8b9e93c0c2d10a18b7b53a207fb1ddd4e6c2e08368a1943187bb1efe0378567340a0851710c426f609aa79d3b5bb3f8efe7f531cfdb54a9fba9e77e3ca2adcecdc299ebf601bd8926dd6ed4e7e71f96ef61cc041159eb0584ff4ce9f0d9e5cb49a91ba15226740f378340e40805aff2e20e275b783aa43a0ac670ec1af2d4e834acceda189add6ed7daf64ed8f9f9718f030c8a7d64afee7cf33beef5f790611eaef40e7c978e2355f3039a6df4f38113ce83ed669a733ce6a93e1fb04fdd6c28815beb6b62f886a47150fbdd34668aa7ff55787874a7b6787a5942da4d73b3197eb792b39d0e338f48fc5f4c01a16a178'
}

```



di source code chall kita di beri tahu bahwa untuk mendapatkan flag dari server adalah membuat message yang kita kirim ada kalimat I am Mallory di awal dan di akhir harus ada own CryptoHack.org.
tapi masalahnya untuk sampai di bagian itu kita harus melewati pemeriksaan signature.masalahnya calculated didest bernilai ```pow(bytes_to_long(SIGNATURE), D, N)``` dimana digest adalah hasil padding dari 'We are hyperreality and Jack and we own CryptoHack.org.

jadi untuk mendapatkan flag kita harus bypass bagian pengecekan signature dengan membuat ```calculated_diges``` bernilai  ```(bytes_to_long(b'padding I am Mallory.own CryptoHack.org')^e mod n```
untuk membuat  ```calculated_diges``` jadi hasil dari  ```(bytes_to_long(b'padding I am Mallory.own CryptoHack.org')^e mod n``` adalah mengubah n dan e,dikarenakan e dan n lah yang bisa kita ubah ubah dikarenakan server tidak mengverifikasi apakah n dan e di rubah atau tidak.

rumus calculated_digest:
**cal_digest = signature^e mod n**

jika e kita buat jadi e=1 untuk menyederhanakan 

**cal_digest = signature mod n**

karena cal_digest kit ubah jadi padding dari 'I am Mallory.own CryptoHack.org',untuk mencari n dari cal_digest kita maka

**n = signature - cal_digest_kita**

jadi dari source code di atas dan siganture yang kita dapat dari kita bisa memanipulasi ```calculated_diges``` dengan n modifikasi kita supaya kita bisa melewati pemeriksaan signature

solver :
```python
from pwn import *
from pkcs1 import emsa_pkcs1_v15
import json
from Crypto.Util.number import bytes_to_long

io = remote("socket.cryptohack.org",13391)
io.recvline()

io.sendline(json.dumps({"option": "get_signature"}).encode())
data_json = json.loads(io.recvline()) 
print(data_json)

s = int(data_json["signature"],16)
msg_target = b"I am Mallory.own CryptoHack.org"
digest_ku =  bytes_to_long(emsa_pkcs1_v15.encode(msg_target,256))
e = hex(1)
n = s - digest_ku
assert digest_ku<n

io.sendline(json.dumps(
    {
        "option": "verify","msg" : "I am Mallory.own CryptoHack.org","N" : hex(n),"e" : "0x01"
        }).encode()
    )

flag = io.recvline()
print(flag)


```

flag = crypto{dupl1c4t3_s1gn4tur3_k3y_s3l3ct10n}
