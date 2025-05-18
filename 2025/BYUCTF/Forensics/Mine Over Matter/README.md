## Mine Over Matter  
Author: Welsh Dragon  
Challenge prompt: Your SOC has flagged unusual outbound traffic on a segment of your network.  
After capturing logs from the router during the anomaly, they handed it over to you—the network analyst.   
Somewhere in this mess, two compromised hosts are secretly mining cryptocurrency and draining resources.   
Analyze the traffic, identify the two rogue IP addresses running miners, and report them to the Incident Response team before your network becomes a crypto farm.  

Objective  
Identify two internal IPs running crypto miners by analyzing:  
-Connections to known mining ports  
-High external connection volume from internal IPs  

Methodology  
We wrote a Python script with a dual detection strategy:  
1. Mining Port Detection
We flagged IPs that initiated TCP connections to known mining ports:  
```
MINING_PORTS = {3333, 5555, 7777, 14444, 9999, 1688}
```
If an internal IP sent packets to these ports, it incremented a suspicious activity counter.  
```
if proto == "tcp" and int(dst_port) in MINING_PORTS:
    internal_ip_mining_ports[src_ip] += 1
```
2. External Connection Frequency  
For a broader detection method, we counted how many times each internal IP connected to external IP addresses (not in 172.16.0.0/12).  
```
if not dst_ip.startswith("172.16."):
    external_conn_counter[src_ip] += 1
```
This helped us catch stealthy miners that might avoid common ports by tunneling over port 443 (HTTPS).  

# Solve script
```
from collections import defaultdict, Counter

# Common crypto mining ports (can be expanded)
MINING_PORTS = {3333, 5555, 7777, 14444, 9999, 1688}

internal_ip_mining_ports = defaultdict(int)
external_conn_counter = Counter()

with open("logs(2).txt", "r") as f:
    for line in f:
        parts = line.strip().split(",")
        if len(parts) < 22:
            continue

        try:
            proto = parts[16].lower()
            src_ip = parts[18]
            dst_ip = parts[19]
            src_port = parts[20]
            dst_port = parts[21]

            # Detect connections to mining ports (TCP usually)
            if proto == "tcp":
                try:
                    dport = int(dst_port)
                    if dport in MINING_PORTS:
                        internal_ip_mining_ports[src_ip] += 1
                except ValueError:
                    continue

            # Count external connections (assuming external IPs are outside 172.16.0.0/12)
            if not dst_ip.startswith("172.16."):
                external_conn_counter[src_ip] += 1

        except IndexError:
            continue

print("=== Likely Miners Based on Mining Ports ===")
for ip, count in sorted(internal_ip_mining_ports.items(), key=lambda x: -x[1]):
    print(f"{ip}: {count} mining port connections")

print("\n=== Top Talkers to External IPs ===")
for ip, count in external_conn_counter.most_common(10):
    print(f"{ip}: {count} external connections")
```

Output   
=== Likely Miners Based on Mining Ports ===  
  
=== Top Talkers to External IPs ===  
172.16.0.4: 130831 external connections  
172.16.0.10: 91915 external connections  
172.16.0.5: 71588 external connections  
192.168.1.20: 55366 external connections  
172.16.96.109: 42252 external connections  
172.16.96.57: 38645 external connections  
172.16.0.70: 18913 external connections  
172.16.96.86: 7544 external connections  
172.16.96.105: 6414 external connections  
172.16.96.26: 4475 external connections  

After some light guesswork, we singled out: 172.16.0.10 and 172.16.0.5  
Final Submission: byuctf{172.16.0.10, 172.16.0.5}  
