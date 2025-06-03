## Connection Issues  
Author: jovantanyk  
Challenge prompt: One of our employees was browsing the web when he suddenly lost connection! Can you help him figure out why?  
Files given: chall.pcap  

First of foremost, we scroll to the last HTTP packet to confirm where the client loses connection.  
Everything appears normal: a GET / to 192.168.100.1, and a 200 OK response.   
However, shortly after this, the client stops receiving replies — no more HTTP responses or ACKs come back.  

After the last HTTP exchange, the client continues to send TCP SYN packets to port 80.  
However, we notice multiple retransmissions — this is the first sign that something is broken in the connection layer.  

The client’s TCP packets are going unanswered.  
This usually means either the server is down — or something lower in the network stack, like ARP, is misdirecting them.   
That’s our cue to check for ARP spoofing.  
```
arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.100.1  
```
We use the filter arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.100.1 to isolate only ARP replies  
claiming to be from the server. The first condition ensures we’re only looking at ARP responses   
(not requests), and the second narrows it to those using the server’s IP.   

While scrolling through the spoofed ARP replies, we notice something strange:  
Opening the hex view of the packets reveals the same few strings being transmitted over and over again, which are likely encoded with base64.  

After base64-decoding the chunks, we assemble the full message:  
# Decoding  
```
import base64  
parts = ["Z3JleXtk=", "MWRfMV9q=", "dXM3X2dl=", "N19wMDFz=", "b24zZH0="]  
flag = "".join(base64.b64decode(p).decode() for p in parts)  
print(flag)  
```
Flag: grey{d1d_1_jus7_ge7_p01son3d}  
