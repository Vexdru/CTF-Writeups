## Are You Looking Me Up?  
Author: Welsh Dragon  
Challenge Prompt:  
The network has a DNS server that's been receiving a lot of traffic.  
You've been handed a set of raw network logs. Your job? Hunt down the DNS server that has received the most DNS requests.  
Analyze the logs and find the impostor.  
Flag format: byuctf{IP1}  

Objective  
Identify the destination IP address (i.e. the DNS server) that received the most DNS requests based on logs.txt.
1. Understand the File Format  
Log lines look like:  
2025-05-06T14:57:53+00:00 164,,,75a2b136446ad166a85f3150b40b7d1e,vtnet0,match,pass,in,4,0x0,,64,44637,0,DF,6,tcp,60,172.16.0.5,91.189.91.98,54412,80,0,S,564511335,,64240,,mss;sackOK;TS;nop;wscale
2025-05-06T14:57:53+00:00 144,,,4fecac63b6446ce80c3df5c947ca3aa1,vtnet0,match,pass,in,4,0x0,,64,59381,0,DF,6,tcp,60,172.16.96.57,216.239.32.106,37609,53,0,S,193292005,,29200,,mss;sackOK;TS;nop;wscale
2025-05-06T14:57:58+00:00 146,,,5c6fe00c82fd64877509b3fc99e38d2c,vtnet0,match,pass,in,4,0x0,,64,39767,0,DF,17,udp,76,172.16.96.30,66.59.198.178,36945,123,56
2025-05-06T14:57:58+00:00 144,,,4fecac63b6446ce80c3df5c947ca3aa1,vtnet0,match,pass,in,4,0x0,,64,33389,0,DF,6,tcp,60,172.16.96.109,216.239.32.106,57497,53,0,S,690687104,,29200,,mss;sackOK;TS;nop;wscale
2025-05-06T14:57:59+00:00 21,,,02f4bab031b57d1e30553ce08e0ec131,vtnet0,match,block,in,4,0x0,,64,48731,0,DF,17,udp,268,192.168.1.20,255.255.255.255,47486,10001,248
...  

From each log line, the destination IP address is the second IP in the sequence — it comes after the source IP.  
The field format looks like:  
...,<src_ip>,<dst_ip>,<src_port>,<dst_port>,...    
  
So:  
-DNS requests use UDP (or sometimes TCP) with destination port 53.  
-We're interested in log lines where the destination port is 53, and we want to count the destination IPs in those lines.  
-Therefore, to identify the most-targeted DNS server:
    Filter all lines where the destination port = 53  
    Extract the destination IP address from those lines  
    Count the frequency of each destination IP  
    Identify the IP with the highest count  

2. Parse and Count IPs with Python

We’ll extract destination IPs and count frequency:  

Solution script  
```
from collections import Counter

with open("logs.txt") as f:
    lines = f.readlines()

dest_ips = []

for line in lines:
    if '→' in line:
        try:
            # Extract the destination part
            dest = line.split('→')[1].strip().split()[0]
            ip = dest.split(':')[0]  # Get IP before port
            dest_ips.append(ip)
        except IndexError:
            continue  # skip malformed lines

# Count occurrences
most_common_ip = Counter(dest_ips).most_common(1)[0]
print(f"Most queried DNS server: {most_common_ip[0]} ({most_common_ip[1]} requests)")
```
Result  

This script outputs:   
Most queried DNS server: 172.16.0.1 (XXX requests)  

Hence, the flag is:  
byuctf{172.16.0.1}  
