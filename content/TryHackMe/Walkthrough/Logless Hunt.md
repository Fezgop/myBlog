# Scenario
```
_Our IT team received an IPS alert on suspicious network behaviour and began investigating._ **_They reviewed the Security and System logs on all our Windows servers and concluded, "All event logs are empty, so hackers did not breach the servers."_**  _But guess what? A few days later, our website started showing some crypto scam ads and some servers were running at 100% CPU load!_

_While our IT team is recovering the critical servers, can you look at our old HR server (_**_HR01-SRV_**_)? We hosted salary review automations there that got unpopular, and the server is now rarely used. However, we noticed a spike in HTTP traffic from the Users' subnet and suspect it to be a part of the attack. We would_ **_appreciate seeing any evidence you can find there!_**

```

I launch the Machine and connect directly to the machine using the giving information *Username*, *Password* and *IP(RDP*)
```  Shell
sudo openvpn file.ovpn
```

and in another tab 

```  Shell
xfreerdp3 /u:Username /v:IP_ADRR /p:Password123
```


---
# Initial Access | Web Access Logs
## Web Access Logs

Web-based services are extremely popular and are the number one target of most threat actors (after exposed RDP). In Microsoft environments, most web apps are behind IIS or Apache, which log incoming requests by default. These access logs provide crucial insight during the investigation of web attacks and can help if you are dealing with:

- Microsoft Exchange (Mail server that uses IIS)
- AD Services (AD CS / SharePoint / RD Web use IIS)
- On-premise web apps (Accounting / CRMs / Wikis)
- Millions of websites worldwide using IIS or Apache

The most crucial context you can get from these logs is:
``` C
- Source IP
- Timestamp
- HTTP Method
- Requested URL 
- Status Code
```

Finally, the default location for these access logs is:

- **Apache**: `C:\Apache24\logs`
- **IIS**: `C:\inetpub\logs\LogFiles\<WEBSITE>`