![[Pasted image 20250501113158.png]]

``` shell
Intrusion detection systems (IDS) are a tool commonly deployed to defend networks by automating the detection of suspicious activity.
```
## What IDS detection methodology relies on rule sets?
``` shell
signature-based detection
```

---

# Network-based IDS (NIDS)
![[Pasted image 20250501113958.png]]

NIDS (eg *Suricata*) monitor networks for malicious activity by checking packets for traces of activity associated with a wide variety of hostile or unwanted activity including:

``` C
 Malware command and control  
 Exploitation tools
 Scanning  
 Data exfiltration
 Contact with phishing sites  
 Corporate policy violations
```

## What widely implemented protocol has an adverse effect on the reliability of NIDS?
``` shell
TLS
```


---
# Reconnaissance Evasion

We can use tools like *nmap* or *nikto* to perform more aggressive scans to enumerate the services.
``` shell
nikto -p port -h IP_ADRR
```
