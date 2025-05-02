First, download [dns.cap](https://wiki.wireshark.org/SampleCaptures?action=AttachFile&do=get&target=dns.cap) on wireshark SampleCaptures wiki page.

``` shell
tshark -r dns.cap | wc -l
```
to read file with tshark and identify number of packets on a capture.

``` shell

tsark -r dns.cap -Y "dns.qry.type == 1"
```
This command show all of A records in our capture, including responses.

To extract data, we'll add to the previous command `-T fields -e [fieldname]` so we have now 
``` shell

tsark -r dns.cap -Y "dns.qry.type == 1" -T fields -e dns.qry.name
```


---
# Task *DNS Exfill*

1. How many packets are in this capture ?

``` shell
tshark -r dns.cap | wc -l
```
Answer : 125

2. How many DNS queries are in this pcap ? (Not responses!)
``` shell

tsark -r dns.cap -Y "dns.flags.response == 0"
```
Answer : 56

3. What is the DNS transaction ID of the suspicious queries (in hex)?
![[Pasted image 20250501175814.png]]
Answer : *0xbeef*



