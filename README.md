# AIOPS-POC/Demo center setup Guide

## DEMO Introduction
By integrating Prometheus, Alertmanager, Ansible EDA, Deepseek AI and AAP, this project realizes automatic detection, root cause analysis and self-healing of Nginx process crash. Core processes include:
1.	Monitoring and alerting: Prometheus and Alertmanager detect Nginx failures and send alerts.
2.	Event-driven: Ansible EDA receives alerts, saves information and triggers Workflow.
3.	AI Analytics: Deepseek provides root cause analysis and fixes Playbook.
4.	Automated repair: AAP performs the repair and notifies the administrator.

The flow chart of Nginx process crash self-healing is as follows:

###	DEMO Scenario
![image](https://github.com/user-attachments/assets/64a703ab-8691-472e-9842-e68c0fa52099)
The Process is as follows:
1. Prometheus/Alertmanager keep watching a target hosts which running Nginx service;
2. EDA runs rulebook with webhook to listen for defined events from Alertmanager
3. when the Nginx Service state is failed in the target hosts have being detected, Alertmanager will send alert to EDA 
4. EDA receiving the event, trigger the Rulebook action “run_module” to save the events information to a file（/tmp/alert_info.txt）;
5. Using a Playbook to analyze the file（/tmp/alert_info.txt）which contains alert events information then to send to Deepseek via API call. the playbook also received the Deepseek’s response(json format), Performing data analysis, formatting and refinement, then send the diagnostic file（/tmp/rhel7_RCA_TIMESTAMP.txt）& Remediate playbook(/tmp/rhel7_remediate.yml) to AAP
6. AAP using a playbook(send_mail.yml) to send the diagnostic information & Remediate playbook to ADMIN mail for decision making;
7. Remediate the Issue if ADMIN approved or after tuned;
8. AAP exec Health check & send Report to ADMIN;



