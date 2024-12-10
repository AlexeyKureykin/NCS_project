Report from moodle:
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

5. Results and Benefits
The Automated SOC Platform offers:
Real-time Incident Detection: Wazuh detects and forwards logs efficiently.
Automated Enrichment: Shuffle adds valuable context to raw alerts.
Streamlined Incident Management: TheHive enables structured handling of security cases.
Proactive Response: Automated workflows reduce response times.

6. Conclusion
This project demonstrates the creation of a fully automated SOC platform capable of detecting, enriching, and responding to security incidents in real-time. By leveraging open-source tools and integrating them effectively, the platform provides a scalable and robust solution for modern cybersecurity challenges.

