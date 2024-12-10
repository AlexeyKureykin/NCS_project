Project assignment
Network and Cyber Security 
CS-01 and CS-02
 Aleksei Kureikin
 Muhibullo Khujabekov

Automated Security Operations Center (SOC) Platform
DEMO: https://youtu.be/WQ9WOM46VtM
GitHub repo: https://github.com/AlexeyKureykin/NCS_project
Table of Contents
Introduction
Architecture Overview
Responsibilities 
Implementation Steps
Configure Wazuh
Set Up Shuffle
Integrating TheHive for Incident Management
Automate Response Actions
Results and Benefits
Conclusion

1. Introduction
This report details the implementation of an Automated SOC Platform designed to detect, enrich, and respond to security incidents in real time. By integrating tools such as Wazuh, Shuffle, and TheHive, this platform ensures continuous monitoring, incident enrichment, and automated response for comprehensive security operations.

2. Architecture Overview
The platform comprises the following key components:
Wazuh: Acts as the central security event and alert generator.
Shuffle: Automates alert enrichment and response workflows.
TheHive: Manages incidents and facilitates response collaboration.
Workflow
Security events are sent from the Wazuh Agent (Windows 10 with Sysmon and Mimikatz) to the Wazuh Manager.
Shuffle receives alerts from Wazuh and enriches them using open-source tools like VirusTotal.
Enriched alerts are sent to TheHive, where incident cases are created and managed.
Shuffle triggers automated responses sending to Telegram bot notifications for critical incidents to engineers.

3. Responsibilities
Muhibullo Khujabekov - Configure Wazuh, Windows/Sysmon/Mimikatz
Aleksei Kureikin - Configure TheHive , Shuffle


4. Implementation Steps
Step 1: Configure Wazuh
Objective: Set up the Wazuh Manager to collect logs and generate alerts while configuring the Wazuh Agent for log forwarding.
Specifications / Minimal requirements:
Hardware Requirements:
RAM: 8 GB+
HDD: 50 GB+
Software Requirements:
Operating System: Ubuntu 22.04 LTS
Wazuh Version: 4.9.2
Commands for Wazuh Installation:

curl -sO https://packages.wazuh.com/4.9.2/wazuh-install.sh && 
sudo bash ./wazuh-install.sh -a


Command to extract Wazuh Credentials:

sudo tar -xvf wazuh-install-files.tar


To add agent:
	Go to wazuh-dashboard and Endpoints/Deploy new agent:



In step 4, wazuh will generate a command, which we will run in a PowerShell on wazuh-agent. 


And run NET START WazuhSvc:


Install and Configure Wazuh Agent (Windows 10)
Installed the agent on a Windows 10 endpoint.
	You can download a Windows 10 ISO from the official Microsoft website. You can install it on VMware Workstation or Virtualbox.
Added Sysmon for event logging and Mimikatz to simulate credential-based attacks.
Sysmon: Download Sysmon
Sysmon Configuration: Sysmon Config File
Screenshot:

After downloading all needed file, run PowerShell as administrator and move to the file directory of sysmon. And run ./sysmon64.exe


For checking that sysmon is installed: open Sercives and find for sysmon.
In screenshot below we can see that sysmon64 is installed.





Test the Setup
So now, we will generate test logs from Windows 10 to verify successful event forwarding to the Wazuh Manager.
To do this will need to modify the ossec.conf file , before modifying it, make a backup.
Here is a location: C:\Program Files (x86)\ossec-agent


Also we need the full name of sysmon : Microsoft-Windows-Sysmon/Operational
Here is a path: EventViewer→Applications&ServicesLog →Microsoft→Windows→Sysmon:



Modified ossec.conf under <Log analysis> should be only this 2 section:
  <localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
  </localfile>

  <localfile>
    <location>active-response\active-responses.log</location>
    <log_format>syslog</log_format>
  </localfile>




So after this we can check Wazuh-dashboard for alerts from sysmon.
Go to Wazuh-dashboard → Discover

How we can see, we get alerts from sysmon:


Also additional modification on Wazuh :
/var/ossec/etc/ossec.conf
<global>
    <jsonout_output>yes</jsonout_output>
    <alerts_log>yes</alerts_log>
    <logall>yes</logall>
    <logall_json>yes</logall_json>
    <email_notification>no</email_notification>
    <smtp_server>smtp.example.wazuh.com</smtp_server>
    <email_from>wazuh@example.wazuh.com</email_from>
    <email_to>recipient@example.wazuh.com</email_to>
    <email_maxperhour>12</email_maxperhour>
    <email_log_source>alerts.log</email_log_source>
    <agents_disconnection_time>10m</agents_disconnection_time>
    <agents_disconnection_alert_time>0</agents_disconnection_alert_time>
  </global>

 


 <integration>
    <name>shuffle</name>          <hook_url>https://shuffler.io/api/v1/hooks/webhook_63503313-c046-472c-aa37-a24b75006d41 </hook_url>
    <rule_id>100002</rule_id>
    <alert_format>json</alert_format>
  </integration>




/etc/filebeat/filebeat.yml — Also we need to ingest archives in Wazuh, for this we need to modify filebeat and add wazuh-archive in wazuh-dashboard

filebeat.modules:
  - module: wazuh
    alerts:
      enabled: true
    archives:
      enabled: true




To add wazuh-archive in wazuh-dashboard : go to DashboardManagment/IndexPattern


Create index pattern →


In step 2 choose a timestamp. 

For testing I run mimikatz in wazuh-agent and capture it in wazuh-dashboard.
I also created a rule that, even if mimikatz will change it name, wazuh will capture it:


How we can see it works

Step 2: Integrate TheHive for Incident Management
Deploying TheHive:
We started by starting another virtual machine on the cloud service for the subsequent integration of TheHive:


First, install the necessary utilities. We will need a distributed database management system Cassandra. We have to configure and run it on thehive virtual machine:







Also, for TheHive to work correctly, we need to install and configure the Elasticsearch search utility:










Now we can install and run TheHive:








Set up TheHive:

Now we can log into TheHive at http://92.51.46.150:9000 and start setting it up:


We can go in as default admin user:


We also need to add our personal profile(aleksei@test.com) and service profile(shuffle@test.com) for future communication with Shuffle via API key:




Step 3: Set Up Shuffle
Objective: Integrate Wazuh alerts into Shuffle for enrichment and automation.

 Lets create a workflow in Shuffle to pull alerts from the Wazuh Manager.

Creating new workflow:


Initializing Webhook utility as Wazuh-Alerts to pull alerts from the Wazuh Manager:


We used this link(https://shuffler.io/api/v1/hooks/webhook_63503313-c046-472c-aa37-a24b75006d41) in the Wazuh config above.

Let's run the .exe file from the Windows client and make sure that workflow processes requests correctly:


Incident enrichment by Virustotal.
So now we need to get the hash from the response in order to use it for incident enrichment in Virustotal:


Adding Virustotal from the list of applications and connecting it to our workflow. As an argument, we will use the hash of alert there for further analysis.

After another test run of the .exe file on client, we get the following analysis from Virustotal:


We can see that 65 of antivirus programs have considered our code as “malicious”.
Connecting Shuffle and TheHive to create alerts.
First, let's add TheHive app to Shuffle and attach it to our workflow:


TheHive configuration for Shuffle:




Now let's run another test of the .exe file from the client and see how TheHive generates alerts:





As we can see, everything works fine. After launching the file on the client, Wazuh Manager automatically sends an alert to TheHive by using Shuffle workflow. At the same time, the alert is sent to Virustotal using a hash and enriches it with additional information about the threat.



For now, our Shuffle workflow looks like:



Step 4: Automate Response Actions
Objective: Enable automated configuring critical notifications.
We also want to implement automatic notification so that the engineer receives the message immediately in case of a threat and can take the necessary measures.
To do this, adding the Telegram bot application to Shuffle and attaching it to our workflow:


Now let's run the test file on the client again and check the automatic notification in Telegram:


Now we can receive a threat alert in a Telegram in real time.



Finally, our workflow in Shuffle looks like this:



5. Results and Benefits
The Automated SOC Platform offers:
Real-time Incident Detection: Wazuh detects and forwards logs efficiently.
Automated Enrichment: Shuffle adds valuable context to raw alerts.
Streamlined Incident Management: TheHive enables structured handling of security cases.
Proactive Response: Automated workflows reduce response times.

6. Conclusion
This project demonstrates the creation of a fully automated SOC platform capable of detecting, enriching, and responding to security incidents in real-time. By leveraging open-source tools and integrating them effectively, the platform provides a scalable and robust solution for modern cybersecurity challenges.

