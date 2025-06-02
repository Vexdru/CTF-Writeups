## Tung Tung Tung Sahur  
Author: elijah5399  
Challlenge prompt:   
New to the world of brainrot? Not sure what names to pick from? We've got you covered with a list of our faves:  
-Tralalero Tralala  
-Chef Crabracadabra  
-Boneca Ambalabu  
-Tung Tung Tung Tung Tung Tung Tung Tung Tung Sahur 

We are given:  
tung_tung_tung_sahur.py: a custom RSA encryption script  
output.txt: the output log from running that script  

The encryption logic deviates from standard RSA:  
```
from Crypto.Util.number import getPrime, bytes_to_long

flag = "grey{flag_here}"
e = 3
p, q = getPrime(512), getPrime(512)
N = p * q
m = bytes_to_long(flag.encode())
C = pow(m, e)  # RSA encryption (no mod N!)

assert C < N
while (C < N):
    C *= 2
    print("Tung!")

while (C >= N):
    C -= N
    print("Sahur!")
```
Observations:
-Standard RSA is C = m^e mod N, but mod N is skipped  
-Instead, the script multiplies C by 2 repeatedly (Tung!) until C >= N  
-Then it subtracts N repeatedly (Sahur!) until C < N again  

This behavior is fully reversible:  
-Every "Tung!" means one C *= 2  
-Every "Sahur!" means one C -= N  

Extracted From output.txt:  
-Tung! count = 164  
-Sahur! count = 1  


# Reversing the Transformation  
Let:  
C_final = the final C from output  
N = given modulus  
k = 164 (number of Tung!)  
s = 1 (number of Sahur!)  

Then:
```
C′ = C_final + N  # Undo 1 Sahur
m³ = C′ // 2^164  # Undo 164 Tung
```
Then we just compute the cube root of m³ to get the original message.  

Solution script:  
```
from Crypto.Util.number import long_to_bytes
from sympy import integer_nthroot

# Given values
e = 3
num_tung = 164
N = 140435453730354645791411355194663476189925572822633969369789174462118371271596760636019139860253031574578527741964265651042308868891445943157297334529542262978581980510561588647737777257782808189452048059686839526183098369088517967034275028064545393619471943508597642789736561111876518966375338087811587061841
C = 49352042282005059128581014505726171900605591297613623345867441621895112187636996726631442703018174634451487011943207283077132380966236199654225908444639768747819586037837300977718224328851698492514071424157020166404634418443047079321427635477610768472595631700807761956649004094995037741924081602353532946351

# Reconstruct m^3
C_prime = C + N
m_cubed = C_prime // (2 ** num_tung)

# Take integer cube root
m, exact = integer_nthroot(m_cubed, 3)

# Convert to bytes
try:
    flag = long_to_bytes(m).decode()
except UnicodeDecodeError:
    flag = long_to_bytes(m)

print(flag)
```
Flag: grey{tUn9_t00nG_t0ONg_x7_th3n_s4hUr}
