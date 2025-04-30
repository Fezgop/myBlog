# # Wazuh — TryHackMe Walkthrough

*By jcm3 - Feb 21, 2023*
*Original Article: [https://medium.com/@jcm3/wazuh-tryhackme-walkthrough-b2c94b0b0dcf](https://medium.com/@jcm3/wazuh-tryhackme-walkthrough-b2c94b0b0dcf)*

---

Wazuh is an open source security platform that provides unified SIEM and XDR capabilities. It is used for threat detection, visibility, monitoring, **deployment**, and **incident response**. In this walkthrough, we will be working through the Wazuh room on [TryHackMe](https://tryhackme.com/room/wazuh).

## Task 1: Introduction

This task introduces Wazuh and its capabilities. Some key features include:

-   Log data analysis
-   Intrusion and malware detection
-   File integrity monitoring
-   Configuration assessment
-   Vulnerability detection
-   Incident response
-   Regulatory compliance
-   Cloud security
-   Container security

There are no questions to answer in this task.

## Task 2: Wazuh Server

In this task, we will deploy the Wazuh server.

1.  Deploy the machine provided by TryHackMe.
2.  Once the machine has loaded, SSH into the server using the provided credentials.
3.  Start the Wazuh manager using the following command:

    ```bash
    sudo systemctl start wazuh-manager
    ```

4.  Access the Wazuh dashboard by navigating to `https://<SERVER_IP>` in your web browser.
5.  Log in using the default credentials: `` `wazuh:wazuh` ``

    ![Image showing Wazuh dashboard login screen]
    *(Placeholder: Image of the Wazuh dashboard login screen)*

6.  After logging in, you should see the Wazuh dashboard overview.

    ![Image showing Wazuh dashboard overview]
    *(Placeholder: Image of the Wazuh dashboard overview)*

There are no questions to answer in this task.

## Task 3: Wazuh Agent

In this task, we will deploy the Wazuh agent on a separate machine.

1.  Deploy the agent machine provided by TryHackMe.
2.  SSH into the agent machine using the provided credentials: `` `agent:agent` ``
3.  Download the Wazuh agent package using the following command:

    ```bash
    wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.3.10-1_amd64.deb -P /tmp
    ```

4.  Install the agent, replacing `<SERVER_IP>` with the IP address of your Wazuh server:

    ```bash
    sudo WAZUH_MANAGER='<SERVER_IP>' dpkg -i /tmp/wazuh-agent_4.3.10-1_amd64.deb
    ```

5.  Reload the systemd daemon, enable the agent service, and start the agent:

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable wazuh-agent
    sudo systemctl start wazuh-agent
    ```

6.  Verify the agent connection by going back to the Wazuh dashboard and navigating to `Modules > Agents`. You should see the agent listed with a status of `Active`.

    ![Image showing agent successfully added in Wazuh dashboard]
    *(Placeholder: Image showing the agent connected in the Wazuh dashboard)*

There are no questions to answer in this task.

## Task 4: Basic SIEM

This task demonstrates basic SIEM capabilities by simulating failed login attempts.

1.  On the agent machine, attempt to SSH into it with an incorrect password multiple times:

    ```bash
    ssh agent@<AGENT_IP>
    ```
    (Enter incorrect passwords when prompted)

2.  Go back to the Wazuh dashboard and navigate to `Modules > Security events`.
3.  Filter the events by searching for `authentication` or `failed password`. You should see events related to the **failed password attempts**.

    ![Image showing SSH failed password attempt events in Wazuh]
    *(Placeholder: Image showing failed SSH login events in Wazuh dashboard)*

4.  **Question:** What is the Rule ID for the failed password attempts?
    *Answer:* Look at the details of one of the failed login events. The Rule ID is `` `5710` ``.

## Task 5: Vulnerability Assessment

This task covers setting up and viewing vulnerability scans.

1.  On the Wazuh server machine, edit the Wazuh configuration file:

    ```bash
    sudo nano /var/ossec/etc/ossec.conf
    ```

2.  Locate the `<vulnerability-detector>` block and ensure it is enabled and configured correctly. The configuration provided in the room enables scanning for Ubuntu Bionic, Focal, and Jammy:

    ```xml
    <vulnerability-detector>
      <enabled>yes</enabled>
      <interval>5m</interval>
      <min_full_scan_interval>6h</min_full_scan_interval>
      <run_on_start>yes</run_on_start>

      <!-- Ubuntu OS vulnerabilities -->
      <provider name="canonical">
        <enabled>yes</enabled>
        <!-- <os>trusty</os> -->
        <!-- <os>xenial</os> -->
        <os>bionic</os>
        <os>focal</os>
        <os>jammy</os>
        <update_interval>1h</update_interval>
      </provider>

      <!-- Debian OS vulnerabilities -->
      <provider name="debian">
        <enabled>no</enabled>
        <os>stretch</os>
        <os>buster</os>
        <update_interval>1h</update_interval>
      </provider>

      <!-- RedHat OS vulnerabilities -->
      <provider name="redhat">
        <enabled>no</enabled>
        <update_from_year>2010</update_from_year>
        <update_interval>1h</update_interval>
      </provider>

      <!-- Arch OS vulnerabilities -->
      <provider name="arch">
        <enabled>no</enabled>
        <update_interval>1h</update_interval>
      </provider>

      <!-- ALAS OS vulnerabilities -->
      <provider name="alas">
        <enabled>no</enabled>
        <update_interval>1h</update_interval>
      </provider>

      <!-- Aggregate vulnerabilities -->
      <provider name="aggregate">
        <enabled>yes</enabled>
        <update_interval>1h</update_interval>
      </provider>

      <!-- Microsoft Windows OS vulnerabilities -->
      <provider name="msu">
        <enabled>no</enabled>
        <update_interval>1h</update_interval>
      </provider>

      <!-- National Vulnerability Database -->
      <provider name="nvd">
        <enabled>yes</enabled>
        <update_from_year>2010</update_from_year>
        <update_interval>1h</update_interval>
      </provider>
    </vulnerability-detector>
    ```

3.  Restart the Wazuh manager to apply the changes:

    ```bash
    sudo systemctl restart wazuh-manager
    ```

4.  Wait a few minutes for the vulnerability scan to run. Go to the Wazuh dashboard, select the agent, and navigate to the `Vulnerabilities` tab.

    ![Image showing vulnerability scan results in Wazuh]
    *(Placeholder: Image showing vulnerability results for the agent)*

5.  **Question:** What is the CVE for the vulnerability related to "Double Free"?
    *Answer:* Find the vulnerability entry related to "Double Free" in the list. The CVE is [CVE-2022-3715](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-3715).

## Task 6: File Integrity Monitoring (FIM)

This task explores the File Integrity Monitoring feature.

1.  On the Wazuh **agent** machine, edit the Wazuh configuration file:

    ```bash
    sudo nano /var/ossec/etc/ossec.conf
    ```

2.  Locate the `<syscheck>` block (this controls FIM). We need to add a directory to monitor. Add the following line within the `<syscheck>` block, for example, under the existing `<directories>` lines:

    ```xml
    <!-- Inside the <syscheck> block -->
    <directories check_all="yes">/home/agent/config</directories>
    ```
    *Note: The original article might have shown adding it to the server config, but FIM monitoring directories are usually configured on the agent.* Ensure the full block looks similar to this (adjusting the added line's position if needed):

    ```xml
    <syscheck>
      <disabled>no</disabled>
      <!-- Frequency that syscheck is executed - default every 12 hours -->
      <frequency>43200</frequency>
      <scan_on_start>yes</scan_on_start>
      <!-- Generate alert when new file detected -->
      <alert_new_files>yes</alert_new_files>
      <auto_ignore>no</auto_ignore>
      <!-- Directories to check (perform all possible verifications) -->
      <directories>/etc,/usr/bin,/usr/sbin</directories>
      <directories>/bin,/sbin,/boot</directories>
      <!-- Add this line -->
      <directories check_all="yes">/home/agent/config</directories>
      <!-- Files/directories to ignore -->
      <ignore>/etc/mtab</ignore>
      <ignore>/etc/hosts.deny</ignore>
      <!-- ... other ignores ... -->
    </syscheck>
    ```

3.  Restart the Wazuh **agent** service:

    ```bash
    sudo systemctl restart wazuh-agent
    ```

4.  On the agent machine, create the directory and a file inside it:

    ```bash
    mkdir /home/agent/config
    touch /home/agent/config/test.txt
    ```

5.  Go back to the Wazuh dashboard and check the security events (`Modules > Security events`). You should see an alert related to the new file being created in the monitored directory.

    ![Image showing FIM alert for new file creation]
    *(Placeholder: Image showing the FIM alert in Wazuh)*

6.  **Question:** What is the full path to the file that was created?
    *Answer:* Based on the command run, the path is `/home/agent/config/test.txt`.

## Task 7: Security Configuration Assessment (SCA)

This task focuses on assessing security configurations against benchmarks.

1.  On the Wazuh **server** machine, edit the configuration file:

    ```bash
    sudo nano /var/ossec/etc/ossec.conf
    ```

2.  Locate the `<sca>` block and ensure it is enabled and points to the correct policy file for your agent's OS (Ubuntu 20.04 in this case):

    ```xml
    <sca>
      <enabled>yes</enabled>
      <scan_on_start>yes</scan_on_start>
      <interval>12h</interval>
      <skip_nfs>yes</skip_nfs>
      <!-- Add or ensure this policy line exists and is correct -->
      <policies>
          <policy>/var/ossec/ruleset/sca/cis_ubuntu20-04_linux.yml</policy>
      </policies>
    </sca>
    ```

3.  Restart the Wazuh **manager** service:

    ```bash
    sudo systemctl restart wazuh-manager
    ```

4.  Wait for the SCA scan to complete (this can take some time). In the Wazuh dashboard, select the agent and navigate to the `Security Configuration Assessment (SCA)` tab.

    ![Image showing SCA scan results, highlighting passed/failed checks]
    *(Placeholder: Image showing the SCA scan results)*

5.  **Question:** How many SCA checks have failed?
    *Answer:* Examine the SCA results overview. The number of failed checks will be displayed (In the article's screenshot, it was 68, but check your own results).

## Conclusion

This walkthrough covered the basic deployment and configuration of Wazuh server and agent, as well as exploring some of its core features like SIEM event analysis, vulnerability detection, file integrity monitoring, and security configuration assessment using the TryHackMe Wazuh room.