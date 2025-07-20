(*source : TryhackMe - MS Sentinel Module*)
Microsoft Sentinel is a combination of a SIEM and a SOAR.
``` shell
SIEM stand for Security Information and Event Management. the purpose is to provide a _holistic view_ of an organization's information security by collecting and analyzing log data from various sources across its IT infrastructure.

SOAR Stand for Security Orchestration, Automation, and Response. It is a set of technologies and practices designed to improve the efficiency and effectiveness of an organization's cyber security operations.
```

SIEM functionality by :
- collecting and querying logs
- Doing **correlation** or **anomaly detection**
- Creating **alerts** and **incidents** based on findings

SOAR functionality by :
- Defining playbooks
- Automating threat responses


![[6601e243753b8d484668851e-1734522464655.png]]

## Collect
- **Data connectors** : ingest data into MS Sentinel.
- **Log retention** data is stored for further correlation and analysis. data can be queried to gain further insights using Kusto Query Language (KQL)

## Detect
- **Workbooks** : essentially dashboards used to visualize data
- **Analytics rules** : Analytics rules provide proactive analytics so that SOC teams get notified when suspicious things happen. 
- **Threat Hunting** : the need to perform proactive threat hunting

## Investigate
Once Analytics rules detect suspicious activities, i.e., once an alert is triggered, security incidents are created for SOC analysts to triage and **investigate**. Typical incident management activities include:

- **Changing** the incident **status**
- **Assigning** to other analysts for further investigation
- **Mapping** entities to the investigation
- Investigating the incident **timeline**
- Deep-dive into investigation details using **investigation maps**  
-  Recording **investigation comments**

## Respond
- **Automation via playbooks** : 