## Real Smooth
Author: overllama

Challenge prompt:

Just do the dance, that's the solve

nc smooth.chal.cyberjousting.com 1350

We are given a remote service that runs the following code:
```
from Crypto.Cipher import ChaCha20
from Crypto.Random import get_random_bytes
from secrets import FLAG

key = get_random_bytes(32)
nonce = get_random_bytes(8)

cipher = ChaCha20.new(key=key, nonce=nonce)
print(bytes.hex(cipher.encrypt(b'Slide to the left')))
print(bytes.hex(cipher.encrypt(b'Slide to the right')))

user_in = input().rstrip('\n')
cipher = ChaCha20.new(key=key, nonce=nonce)
decrypted = cipher.decrypt(bytes.fromhex(user_in))
if decrypted == b'Criss cross, criss cross':
    print("Cha cha real smooth")
    print(FLAG)
else:
    print("Those aren't the words!")
```
We're expected to somehow send a ciphertext that decrypts to:

b"Criss cross, criss cross"

But we are never given the key or nonce, and both are randomly generated per connection.
This is a classic stream cipher setup using ChaCha20. Here's what matters:
-ChaCha20 generates a keystream from the (key, nonce) pair.
-It XORs the keystream with the plaintext to produce ciphertext.
-The server prints:
    encrypt("Slide to the left") → 18 bytes
    encrypt("Slide to the right") → 19 bytes
    
That means we have 37 total ciphertext bytes.
Since we know the corresponding plaintexts, we can recover 37 bytes of keystream.

After failing with partial inputs like:
    b"Criss cross, criss " (only 18 bytes, matched ct1)
    b"Criss cross, criss cr" (19 bytes, matched ct2)

I realized that the ChaCha20 cipher object was reused, so the second .encrypt() call continues the keystream stream right after the first.
Hence, ct1 + ct2 gives a continuous 37-byte stretch of keystream if we combine them.

# Exploit Plan
-Recover the keystream:
```
keystream = xor(ct1 + ct2, b"Slide to the leftSlide to the right")
```
-Encrypt the desired message:
```
    payload = xor(keystream[:24], b"Criss cross, criss cross")
```
-Send the payload hex back to the server

Solution Script
```
from pwn import *
from binascii import unhexlify, hexlify

def xor_bytes(a, b):
    return bytes([x ^ y for x, y in zip(a, b)])

pt_full = b"Slide to the leftSlide to the right"
target = b"Criss cross, criss cross"

conn = remote("smooth.chal.cyberjousting.com", 1350)
ct1 = unhexlify(conn.recvline().strip().decode())
ct2 = unhexlify(conn.recvline().strip().decode())
ct_full = ct1 + ct2
keystream = xor_bytes(ct_full, pt_full)
payload = xor_bytes(keystream[:len(target)], target)
conn.sendline(hexlify(payload))
print(conn.recvall().decode())
```
Flag: byuctf{ch4ch4_sl1d3?...n0,ch4ch4_b1tfl1p}
