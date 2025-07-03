# AIOPS-POC/Demo center setup Guide

## DEMO Introduction
By integrating Prometheus, Alertmanager, Ansible EDA, Deepseek AI and AAP, this project realizes automatic detection, root cause analysis and self-healing of Nginx process crash. Core processes include:
1.	Monitoring and alerting: Prometheus and Alertmanager detect Nginx failures and send alerts.
2.	Event-driven: Ansible EDA receives alerts, saves information and triggers Workflow.
3.	AI Analytics: Deepseek provides root cause analysis and fixes Playbook.
4.	Automated repair: AAP performs the repair and notifies the administrator.

The flow chart of Nginx process crash self-healing is as follows:

###	DEMO Scenario
![image](https://github.com/user-attachments/assets/806c835b-d59b-42b3-a98b-9777c83defed)
![image](https://github.com/user-attachments/assets/917b6e6a-8b06-4e9e-acee-19c64397c697)

###	DEMO Expecting results
![image](https://github.com/user-attachments/assets/6bb87e61-a521-4752-8007-6e2492151706)
![image](https://github.com/user-attachments/assets/6f0033ac-01a6-4f75-9218-e9d630fbb22f)
![image](https://github.com/user-attachments/assets/c24ce408-7415-4aad-b516-bf0bae7e570d)
![image](https://github.com/user-attachments/assets/acccaf3a-dc4b-4a5e-8c09-786c54f36fb0)

#### Branch 1：Accepted the AI’s resolution 
![image](https://github.com/user-attachments/assets/c41edc1c-230b-42e1-ac1a-6976bb189008)

#### Branch 2：Branch 2: Tuning the playbook w/ Gitops
![image](https://github.com/user-attachments/assets/39cdfa2b-92f7-4e57-8e41-466981ab2c30)


 






