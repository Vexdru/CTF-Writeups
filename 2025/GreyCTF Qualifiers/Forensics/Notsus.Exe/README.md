## Notsus.Exe  
Author: TheMythologist  
Challenge prompt: (Insert Guessy forensics challenge description here)

Step 1: Cracking the ZIP Password with bkcrack  

The ZIP file used the old and broken ZipCrypto encryption.   
This can be cracked if we know just 12 bytes of plaintext from an encrypted file inside it.   
Luckily, notsus.exe starts with the standard Windows MZ header:  
```
MZ\x90\x00\x03\x00\x00\x00\x04\x00\x00\x00
```
We used that to recover the decryption keys:  
```bash
printf '\x4d\x5a\x90\x00\x03\x00\x00\x00\x04\x00\x00\x00' > pt12.bin
bkcrack -C files.zip -c notsus.exe -x 12:24 | xxd -r -p > ct12.bin
bkcrack -C files.zip -c notsus.exe -p pt12.bin -d ct12.bin
```
This gave us the internal ZipCrypto keys:  
```
d1608c35 d11d350a 4bc3da9c
```
Then we decrypted the real target:  
```bash
bkcrack -C files.zip -c flag.txt.yorm -k d1608c35 d11d350a 4bc3da9c -d flag.txt
```
Unfortunately, flag.txt still looked like garbage:  
```
r\xA5\xF4\x1A... (random bytes)
```
That meant the flag was encrypted again inside the binary logic.  

Step 2: Unpacking the Executable (PyInstaller)  

We noticed notsus.exe was built using PyInstaller, which bundles Python programs into a single `.exe`.  
To extract the Python code, we used:  
```bash
wget https://raw.githubusercontent.com/extremecoders-re/pyinstxtractor/master/pyinstxtractor.py
python3 pyinstxtractor.py notsus_real.exe
```
This extracted the contents of notsus.exe and gave us the core file that likely contained the encryption logic: `notsus.pyc`.  

Step 3: Decompiling Python 3.12 .pyc Files  

Sadly, normal decompilers like uncompyle6 failed because the bytecode was Python 3.12. So we manually disassembled it:  
# Disassembly script  
```python  
import marshal, types, dis

with open("notsus.pyc", "rb") as f:
    f.read(16)
    top = marshal.load(f)

def dis_all(co, name="(top)", indent=0):
    prefix = " " * indent
    print(f"{prefix}Disassembly of {name}:")
    print(f"{prefix}" + "-"*40)
    for line in dis.Bytecode(co):
        print(f"{prefix}{line.opname:20} {line.argrepr}")
    print()
    for const in co.co_consts:
        if isinstance(const, types.CodeType):
            dis_all(const, const.co_name, indent + 2)

dis_all(top)
```
We found 3 key functions:  
* `c()` lists all files in a folder recursively  
* `b(key, data)` is a custom RC4 decryptor  
* `a()` is an XOR helper (not used in our path)  

A global variable `d = b"HACKED!"` was used as the RC4 key.  

The code was decrypting files using:  
```python
decrypted = b(d, asdf)  # where asdf = file.read()
```
Step 4: Reimplementing the Decryption in Python  

We rewrote the RC4 function from the disassembly:  
# Solution script
```python
def rc4(key, data):
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    i = j = 0
    out = bytearray()
    for byte in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        out.append(byte ^ S[(S[i] + S[j]) % 256])
    return bytes(out)

# Load your ciphertext
with open("flag.txt", "rb") as f:
    ct = f.read()

# Decrypt with key b"HACKED!"
pt = rc4(b"HACKED!", ct)
print(pt.decode())
```
Flag: grey{this_program_cannot_be_run_in_dos_mode_hehe}
