src : [Wazuh - TryHackMe ](https://tryhackme.com/room/wazuhct)
``` shell
Endpoint detection and response (EDR) are a series of tools and applications that monitor devices for an activity that could indicate a threat or security breach.
```


---
# Wazuh Agents

Agents are devices that record the events and processes of a system.
On wazuh you can deploy new agent on different OS like Windows, MacOS, Ubuntu, CentOS.

# Wazuh Vulnerability Assessment & Security Events

``` xml
This module is a powerful tool that can be used to periodically scan an agent's operating system for installed applications and their version numbers
```

The vulnerability scanner module will perform a full scan when the Wazuh agent is first installed on a device and **must** be configured to run at a set interval then after (by default, this is set to 5 minute intervals when enabled)
![[Pasted image 20250501181948.png]]
On _/var/ossec/etc/ossec.conf_

# Wazuh Policy Auditing

``` xml
Wazuh is capable of auditing and monitoring an agent's configuration whilst proactively recording event logs. When the Wazuh agent is installed, an audit is performed where a metric is given using multiple frameworks and legislations such as NIST, MITRE and GDPR.
```

![[Pasted image 20250501182809.png]]
# Monitoring Logons with Wazuh
``` xml
Wazuh's security event monitor is capable to actively record both successful and unsuccessful authentication attempts.
```

| **Field**            | **Description**                                                                                                                                          |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| agent.ip             | This is the IP address of the agent that the alert was triggered on.                                                                                     |
| agent.name           | This is the hostname of the agent that the alert was triggered on.                                                                                       |
| rule.description     | This field is a brief description of what the event is alerting to.                                                                                      |
| rule.mitre.technique | This field explains the MITRE technique that the alert pertains to.                                                                                      |
| rule.mitre.id        | This field is the MITRE ID of the alert                                                                                                                  |
| rule.id              | This field is the ID assigned to the alert by Wazuh's ruleset                                                                                            |
| location             | This field is the location of the file that the alert was generated from on the agent. In this example, it is the authentication log on the linux agent. |
The alerts is stored in a specific file on the Wazuh management server ``/var/ossec/logs/alerts/alerts.log ``

# Collecting Windows Logs with Wazuh
``` shell
All sorts of actions and events are captured and recorded on a Windows operating system. This includes authentication attempts, networking connections, files that were accessed, and the behaviours of applications and services. This information is stored in the Windows event log using a tool called Sysmon.
```

# Collecting Linux Logs with Wazuh
``` shell
Capturing logs from a Linux agent is a simple process similar to capturing events from a Windows agent. We will be using Wazuh’s log collector service to create an entry on the agent to instruct what logs should be sent to the Wazuh management server.
```

Wazuh comes with many rules that enable Wazuh to analyze log files and can be found in `/var/ossec/ruleset/rules`

# Auditing Commands on Linux with Wazuh

Wazuh utilises the `auditd` package that can be installed on Wazuh agents running on Debian/Ubuntu and CentOS operating systems.

First, we will need to install the `auditd` package and an `auditd` plugin. 
``` shell
sudo apt install auditd audispd-plugins
```
And enable this service to run currently as well as on boot
``` shell
sudo systemctl enable auditd.service & sudo systemctl start auditd.service
```

We will need to configure `auditd` to create a rule for the commands and events that we wish for it to monitor.
`Auditd` rules are located in the following directory: `/etc/audit/rules.d/audit.rules`.

We will now need to inform audits of this new rule, so let's run this command `sudo auditctl -R /etc/audit/rules.d/audit.rules` to now read the new _audit.rules_ file.

Now, we'll add `auditd` log on `/var/ossec/etc/ossec.conf`
``` xml
<localfile>
    <location>/var/log/audit/audit.log</location>
    <log_format>audit</log_format>
</localfile>
```

# wazuh API

## Using Our Own Client
``` shell
The Wazuh management server features a rich and extensive API to allow the Wazuh management server to be interacted with using the command line. Because the Wazuh management server requires authentication, we must first authenticate our client.
```

First, we will need to authenticate ourselves by providing a valid set of credentials to the authentication endpoint.
Once we are authenticated, the Wazuh management server will give us a token (similar to a session) that we will need to provide for any further interaction. We can store this token as an environment variable on our Linux machine
``` shell
TOKEN=$(curl -u : -k -X GET "https://WAZUH_MANAGEMENT_SERVER_IP:55000/security/user/authenticate?raw=true")
```
To confirm that we have authenticated okay and have been given a token by the Wazuh management server, we run this command
``` shell
curl -k -X GET "https://WAZUH_MANAGEMENT_SERVER_IP:55000/" -H "Authorization: Bearer $TOKEN"
```

We can use the Wazuh API to list some statistics and important information about the Wazuh management server, including what services are being monitored and some general settings about the Wazuh management server:

``` shell
curl -k -X GET "https://WAZUH_MANAGEMENT_SERVER_IP:55000/manager/configuration?pretty=true§ion=global" -H "Authorization: Bearer $TOKEN"
```

## Using Wazuh's API Console
``` shell 
Wazuh has a powerful, integrated API console within the Wazuh website to query management servers and agents. Whilst it is not as extensive as using your own environment (where you can create and run scripts using python, for example), it is convenient.
```

To find this API console, we need to open the "Tools" category within the Wazuh heading at the top. We can view the Wazuh's detailed [APIdocumentation](https://documentation.wazuh.com/current/user-manual/api/reference.html)

# Generating Reports with Wazuh
``` shell
Wazuh features a reporting module that allows you to view a summarised breakdown of events that have occurred on an agent.
```
First, we will need to select a view to generate reports from. In this example, I want to generate a report of the security events in the last 24 hours. To do so, I will need to open the view: **1. Modules -> 2. Security Events**
![[Pasted image 20250501192432.png]]

Now, if there have been alerts within the last 24 hours, I can generate a report like so:
![[Pasted image 20250501192453.png]]

First, press on the "Wazuh" heading at the top of the screen and select "**Management**", and then click on the "**Reporting**" text located under the "**Status and Reports**" sub-heading:
![[Pasted image 20250501192600.png]]
The report overview dashboard lists all generated reports. To download a report, press the save icon on the right of the report located under the "**Actions**" heading.

