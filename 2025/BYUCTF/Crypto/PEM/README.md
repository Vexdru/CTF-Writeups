#PEM

Author: overllama

Challenge Prompt:
$flag ∈ \sqrt N$

File Given: ssh_host_rsa_key.pub (PEM-format RSA public key)

We're given a PEM-formatted RSA public key and a mathematical hint:

$flag ∈ \sqrt N$

This tells us that the flag (converted to an integer) is less than or equal to √N, where N is the RSA modulus. No ciphertext is provided — just the public key.
We know that RSA encryption typically works like:

ciphertext = m^e mod n

But if the plaintext m is small enough that:

m^e < n

Then:
-The mod n has no effect
-The ciphertext is just c = m^e
-And the decryption becomes:

m = ⌊c ** (1/e)⌋

However, no ciphertext is given. Instead, the challenge hint suggests: The flag itself is simply √N

Exploit Steps
-Load the public RSA key from the provided PEM file
-Extract n (the modulus)
-Take the integer square root of n
-Convert the result back to bytes to get the flag

Solution code
```
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.backends import default_backend
from math import isqrt
with open("ssh_host_rsa_key.pub", "rb") as f:
    pubkey = serialization.load_pem_public_key(f.read(), backend=default_backend())
n = pubkey.public_numbers().n
m = isqrt(n)
flag = m.to_bytes((m.bit_length() + 7) // 8, byteorder='big').decode()
print(flag)
```

Output:
fjagkdflgfkdsjgfdltyugvjcbghjqfsdjvfdhbjfd byuctf{P3M_f0rm4t_1s_k1ng} cmxvblalsfiuqeuipplvdldbnmjzxydhjgfdgppsksjq

Flag: byuctf{P3M_f0rm4t_1s_k1ng}
