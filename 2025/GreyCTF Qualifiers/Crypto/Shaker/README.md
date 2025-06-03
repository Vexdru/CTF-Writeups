## Shaker  
Author: hadnot  
Challenge prompt: You found a shaker. Can you get the flag out? nc challs.nusgreyhats.org 33302  

We were given: shaker.py    
Looking into the server's source code, we find an opening: the md5 fingerprint of the flag.  
```
from hashlib import md5  
assert md5(flag).hexdigest() == "4839d730994228d53f64f0dca6488f8d"  
```
# Server Logic  
The server holds a secret:  
-flag: a 64-byte string  
-x: a 64-byte secret mask  

Every time you "shake" (option 3), it:
-Picks a random permutation perm of 64 indices  
-Updates state = permute(flag) ^ x  

When you "see inside" (option 2), it:  
-Prints bytes(state) in hex  

But here’s the key detail:  
The first time you choose option 2, the state is just flag ^ x — unpermuted.  
 
So what the server is really leaking every time is:  
```
output = P(flag) ^ x  
```
Where:  
-P is the identity permutation initially  
-Then randomized permutations for all future resets  
# Attack Strategy  
We want to recover the flag, but we only see flag ^ x, P1(flag) ^ x, P2(flag) ^ x, etc.  

There are two things we don’t know:  
-The permutation P  
-The XOR mask x  

But here's the trick:  
1. Fix the permutation problem using multisets  
For each output byte position (i.e. state[i]), collect a multiset S_i of all the values seen over many samples.  
Since the permutations shuffle the flag, each S_i eventually contains every flag byte, just XOR-ed with the fixed mask byte x[i].   

2. Now eliminate the mask (up to a constant)   
Suppose:
$$
S_0 = { b ^ x[0] | b in flag }  
S_j = { b ^ x[j] | b in flag }
$$
Then
$$
S_j = S_0 ^ (x[0] ^ x[j])  
$$
So we brute-force x[0] ^ x[j] by finding which value aligns S_0 to S_j.  
Now know the XOR mask x, up to the unknown byte x[0].  

3. Recover the flag (modulo x[0])  
We already saw the very first output: flag ^ x.  

Using our known x[j] ^ x[0] = d[j], we undo the mask:  
```
flag[j] = (flag_xor_x)[j] ^ d[j] ^ x[0]  
```
Now we brute-force all 256 possible values of x[0].  

Finally, we check if our reconstructed flag's MD5 hash matches the one given in the challenge:  
```
md5(flag) == "4839d730994228d53f64f0dca6488f8d"  
```
Only one of the 256 candidates will match. That's our flag.  

# Solution Script:  
```
#!/usr/bin/env python3
from pwn import *
import hashlib
import string

HOST, PORT = "challs.nusgreyhats.org", 33302
ROUNDS      = 1200          # plenty, stays well below timeouts
TARGET_MD5  = "4839d730994228d53f64f0dca6488f8d"

def get_hex(conn):
    line = conn.recvline().strip()  # "Result: <hex>"
    return bytes.fromhex(line.split(b":")[1].strip().decode())

def main():
    io = remote(HOST, PORT)

    # ---- collect outputs --------------------------------------------------
    sets   = [set() for _ in range(64)]

    io.sendlineafter(b"> ", b"2")        # first read = flag ^ x
    y0 = get_hex(io)
    for _ in range(ROUNDS):
        io.sendlineafter(b"> ", b"2")
        out = get_hex(io)
        for j in range(64):
            sets[j].add(out[j])

    # ---- derive mask differences d_j = x[0] ^ x[j] ------------------------
    d = [0]*64
    S0 = sets[0]
    for j in range(1, 64):
        for cand in range(256):
            if all((b ^ cand) in sets[j] for b in S0):
                d[j] = cand
                break
        else:
            log.failure(f"Could not find d for column {j}")
            return

    # ---- brute x0 & test MD-5 --------------------------------------------
    for x0 in range(256):
        flag_bytes = bytes(y0[j] ^ x0 ^ d[j] for j in range(64))
        if hashlib.md5(flag_bytes).hexdigest() == TARGET_MD5:
            log.success(f"FLAG FOUND:\n{flag_bytes.decode()}")
            io.close()
            return

    log.failure("No x0 worked")

if __name__ == "__main__":
    main()
```
Flag: grey{kinda_long_flag_but_whatever_65k2n427c61ww064ac3vhzigae2qg}  
