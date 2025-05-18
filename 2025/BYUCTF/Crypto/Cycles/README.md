## Cycles  
Author: overllama  
Challenge prompt:  
I heard through the grapevine that you can't take a discrete log under a large enough, good prime.  
Well I made sure to pick a good prime, go ahead and try  

We were given main.py:    
```
from Crypto.Util.number import long_to_bytes, bytes_to_long, isPrime
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad

# Can you undo this?
from hidden import p,N,a,flag,g

# these are for you :)
assert isPrime(p)
assert len(bin(a)) < 1050

hint = pow(g, a, p)
key = long_to_bytes(a)[:16]

cipher = AES.new(key, AES.MODE_ECB)
ct = cipher.encrypt(pad(flag, AES.block_size))

# Now for your hints
print(f"g = {g}")
print(f"P = {p}")
print(f"ciphertext = {ct}")
print(f"Hint = {hint}")
```
And values from cycles.txt:  
g = 3  
P = 121407847840823587654648673057258513248172487324370407391241175652533523276605532412599555241774504967764519702094283197762278545483713873101436663001473945726106157159264352878998534133035299601861808839807763182625559052896295039354029361792893109774218584502647139466059910154701304129191164513825925289381  
ciphertext = b'S\x00\xe7%\xcd\xec\xad\x9a\xe1lO\x80\xd6\r\xa5\x00\x19Y\x18\x7f\xa1\x9cx\x98\xb08n~-\rj2\xd4d\xd2\xda\xa6\xd0\r#7\xee-\\\xb4\x10\x98\x8f'  
Hint = 1  

# Observations
hint = pow(g, a, p) == 1 implies:  
    a ≡ 0 mod ordp(g)    
So a is a multiple of the order of g modulo p.  
If g is a primitive root mod p, then:  
    ordp(g) = p−1 ⇒ a = k⋅(p−1)  

We were also told:  
    len(bin(a)) < 1050  
Which implies a isn't some insane multiple — likely k is small.  

# Exploit Strategy  
Step 1: Try a = k * (p - 1) for small k  
-g = 3 is likely a primitive root  
-So any multiple of p - 1 will satisfy g^a ≡ 1 mod p  
-If k is small, AES key = long_to_bytes(a)[:16] is deterministic and decryptable  

Step 2: For each a, derive the AES key and try decryption  
Check if AES-ECB decryption yields valid PKCS#7 padding. If so → we win.  

Solution script  
```
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
from Crypto.Util.number import long_to_bytes

p = 121407847840823587654648673057258513248172487324370407391241175652533523276605532412599555241774504967764519702094283197762278545483713873101436663001473945726106157159264352878998534133035299601861808839807763182625559052896295039354029361792893109774218584502647139466059910154701304129191164513825925289381

ct = bytes.fromhex(
    "5300e725cdecad9ae16c4f80d60da5001959187fa19c7898b0386e7e2d0d6a32"
    "d464d2daa6d00d2337ee2d5cb410988f"
)

for k in range(1, 1000):
    a = k * (p - 1)
    key = long_to_bytes(a)[:16].ljust(16, b'\x00')
    cipher = AES.new(key, AES.MODE_ECB)
    try:
        flag = unpad(cipher.decrypt(ct), 16).decode()
        print(f"Flag\na = {a}\nAES Key: {key.hex()}\nFlag: {flag}")
        break
    except ValueError:
        continue
```
Flag: byuctf{1t_4lw4ys_c0m3s_b4ck_t0_1_21bcd6}
