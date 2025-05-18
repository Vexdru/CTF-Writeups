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
2025-05-06T14:57:53+00:00  144,,,4fecac63b6446ce80c3df5c947ca3aa1,vtnet0,match,pass,in,4,0x0,,64,59381,0,DF,6,tcp,60,172.16.96.57,216.239.32.106,37609,53,0,S,193292005,,29200,,mss;sackOK;TS;nop;wscale  
2025-05-06T14:57:58+00:00 146,,,5c6fe00c82fd64877509b3fc99e38d2c,vtnet0,match,pass,in,4,0x0,,64,39767,0,DF,17,udp,76,172.16.96.30,66.59.198.178,36945,123,56  
2025-05-06T14:57:58+00:00   144,,,4fecac63b6446ce80c3df5c947ca3aa1,vtnet0,match,pass,in,4,0x0,,64,33389,0,DF,6,tcp,60,172.16.96.109,216.239.32.106,57497,53,0,S,690687104,,29200,,mss;sackOK;TS;nop;wscale  
2025-05-06T14:57:59+00:00 21,,,02f4bab031b57d1e30553ce08e0ec131,vtnet0,match,block,in,4,0x0,,64,48731,0,DF,17,udp,268,192.168.1.20,255.255.255.255,47486,10001,248  
...  

From the logs, we can see that:  
Fields are comma-separated.  
IPs appear as: ...<source_ip>,<destination_ip>,<source_port>,<destination_port>...  
The second IP in the list is the destination (i.e., the server being contacted).  

2. Parse and Count IPs with Python

We’ll extract destination IPs and count frequency:  

Solution script  
```
from collections import Counter
import re

# Load the logs
with open("logs.txt") as f:
    logs = f.readlines()

# Extract IPs – assuming the destination IP is the last or second IP in each line
ip_pattern = r'(?:\d{1,3}\.){3}\d{1,3}'
ip_counter = Counter()

for line in logs:
    ips = re.findall(ip_pattern, line)
    if len(ips) >= 2:
        dest_ip = ips[1]  # usually 2nd IP is the DNS server
        ip_counter[dest_ip] += 1

# Get the IP with the highest DNS requests
most_common_ip, count = ip_counter.most_common(1)[0]

# Print the flag
print(f"byuctf{{{most_common_ip}}}")
```

Flag: byuctf{172.16.0.1}  
