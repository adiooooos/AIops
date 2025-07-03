# AIOps Demo: Automated Nginx Self-Healing

This repository contains the resources for an AIOps demonstration focused on the automated detection, diagnosis, and self-healing of a crashed Nginx service. The solution integrates leading open-source monitoring tools with Red Hat Ansible Automation Platform and a Large Language Model (LLM) to create a closed-loop, event-driven automation workflow.

This project was developed by Fengyuan Zhang (fzhang@redhat.com) as a proof-of-concept for intelligent IT operations.

## Overview

In modern IT environments, manual intervention for common service failures is slow, error-prone, and inefficient. This AIOps demo showcases a modern approach where a system can autonomously handle a critical service failure.

When the Nginx web server process on a target host crashes, the following automated workflow is triggered:
1.  **Detect:** Prometheus detects the service failure.
2.  **Alert:** Alertmanager sends a structured alert to Ansible.
3.  **Trigger:** Event-Driven Ansible (EDA) receives the alert and triggers an automation workflow in Ansible Automation Platform (AAP).
4.  **Diagnose & Plan:** AAP invokes a playbook that queries a Large Language Model (Deepseek AI). The AI performs a root cause analysis (RCA) and generates a bespoke Ansible Playbook to resolve the specific issue.
5.  **Approve:** The AI-generated analysis and remediation plan are emailed to an administrator for approval via a built-in workflow gate.
6.  **Remediate:** Upon approval, AAP automatically executes the AI-generated playbook to repair the Nginx service.
7.  **Verify:** The system confirms that the service is restored and fully operational.

## Features

-   **Automated Monitoring:** Leverages Prometheus and `node_exporter` for real-time monitoring of systemd services.
-   **Event-Driven Automation:** Utilizes Event-Driven Ansible (EDA) to listen for webhooks from Alertmanager and initiate automated actions.
-   **AI-Powered Diagnostics:** Integrates with the Deepseek AI API to perform intelligent Root Cause Analysis (RCA).
-   **AI-Generated Remediation:** Dynamically generates Ansible Playbooks tailored to fixing the identified issue, incorporating best practices.
-   **Human-in-the-Loop:** Includes a workflow approval step, ensuring an administrator can review and validate the AI's proposed solution before execution.
-   **End-to-End Workflow:** A comprehensive Ansible Automation Platform workflow orchestrates the entire process from alert reception to final remediation.

## Architecture

The demo environment consists of three primary components:

1.  **Ansible Automation Platform (AAP) All-in-One Server:**
    * **OS:** Red Hat Enterprise Linux (RHEL) 9.2+.
    * **Services:** Hosts AAP Controller, Automation Hub, and Event-Driven Ansible (EDA). This server orchestrates the entire AIOps workflow.
2.  **Prometheus Server:**
    * **Services:** Hosts the Prometheus monitoring system and Alertmanager. It collects metrics and fires alerts.
3.  **Target Nginx Server:**
    * **OS:** Red Hat Enterprise Linux (RHEL) 7.
    * **Services:** Runs the critical Nginx service and the `node_exporter` agent for metric collection by Prometheus.

### High-Level Diagram
![image](https://github.com/user-attachments/assets/dffd87fc-114d-4393-a77b-22c0341ab930)
![image](https://github.com/user-attachments/assets/5a559f6c-6af6-41b8-9a45-79c2e440e377)


## Workflow Process

The self-healing process is executed in a sequence of automated steps orchestrated by the AAP Workflow Template `AIOPS_Nginx_self_healing`.

1.  **Observe:** Prometheus continuously monitors the `nginx.service` on the target host. If the service enters a `failed` state, an alert is triggered.
2.  **Evaluate (EDA):** Alertmanager forwards the alert to an Ansible EDA webhook. The rulebook `Linux_Service_failed_for_UIworkflow.yml` matches the alert and triggers the AAP workflow. EDA first saves the alert data to `/tmp/alert_info.txt`.
3.  **Evaluate (AAP & AI):** The first step in the workflow executes the `send_alert_to_deepseek_for_UI_exec.yml` playbook. This playbook:
    * Fetches the alert data from the EDA host.
    * Parses the alert to extract key details (IP address, service name, status).
    * Constructs a detailed prompt for the Deepseek AI, requesting an RCA and a remediation playbook.
    * Calls the Deepseek API and receives the response.
    * Saves the RCA and the generated playbook into separate files (`/tmp/rhel7_RCA_...txt` and `/tmp/rhel7_remediate_by_deepseek.yml`).
    * Emails these two files to a designated administrator.
4.  **Admin Approval:** The workflow pauses at an approval gate, waiting for an administrator to review the generated files and approve the remediation step within the AAP UI.
5.  **Respond (Remediate):** Once approved, the workflow proceeds to the final step, which executes the newly generated `rhel7_remediate_by_deepseek.yml` playbook against the target host.
6.  **Result:** The Nginx service is restarted and restored to a healthy, active state. The demo webpage becomes accessible again.

## Prerequisites

To replicate this demo, you will need:
-   A running instance of Red Hat Ansible Automation Platform 2.5 or newer.
-   A configured Prometheus and Alertmanager server.
-   A target Linux host (e.g., RHEL 7) with Nginx installed.
-   Network connectivity between all components.
-   An API key for the Deepseek AI service.
-   An SMTP server and credentials for email notifications.

## Setup and Configuration

The files required for this demo are located in the `AIOPS_in_Action/` directory. For a complete step-by-step installation guide for the entire environment, please refer to the `AIOPS_DEMO_场景1_Nginx进程崩溃自愈_StepByStepGuide_0505.docx` document.

Below is a high-level configuration summary:

1.  **Clone Repository:**
    ```bash
    git clone [https://github.com/adiooooos/AIops.git](https://github.com/adiooooos/AIops.git)
    ```

2.  **Monitoring Configuration:**
    * **`node_exporter`:** Install and enable `node_exporter` on the Nginx target server. Ensure the `systemd` collector is enabled.
    * **`prometheus.yml`:** Add the `nginx_service_alert.yml` to the `rule_files` section. Configure the `scrape_configs` to target the `node_exporter` on the Nginx host. Set the `alerting` section to point to your Alertmanager instance.
    * **`alertmanager.yml`:** Configure the `receivers` section with a `webhook_configs` entry pointing to the Ansible EDA listener URL (e.g., `http://<EDA_SERVER_IP>:5000/alerts`).

3.  **Ansible Configuration:**
    * **Inventory:** Update the `AIOPS_in_Action/inventory` file with the correct IP addresses for your `[rhel7]`, `[aap25EDA]`, and `[aap25]` hosts.
    * **Playbook Variables:** Edit `AIOPS_in_Action/send_alert_to_deepseek_for_UI_exec.yml` and update the `vars` section with your Deepseek API key (`Authorization: "Bearer sk-..."`) and your SMTP credentials.
    * **AAP Configuration:**
        * Create a new **Project** in AAP pointing to the local path of the cloned repository files.
        * Create a new **Inventory** in AAP and source it from the project's inventory file.
        * Create the required **Job Templates** and the main **Workflow Job Template** as detailed in the `StepByStepGuide` document.

4.  **Event-Driven Ansible (EDA):**
    * Run the rulebook from the command line on your EDA server to start listening for alerts. Ensure you provide the controller URL and credentials.
    ```bash
    ansible-rulebook --rulebook AIOPS_in_Action/Linux_Service_failed_for_UIworkflow.yml \
                     --controller-url https://<AAP_IP> \
                     --controller-username <USER> \
                     --controller-password <PASS> \
                     -i AIOPS_in_Action/inventory \
                     --controller-ssl-verify no
    ```

## How to Run the Demo

1.  Ensure all services (AAP, Prometheus, Alertmanager, EDA listener) are running and configured correctly.
2.  Verify that the Nginx service is active and the demo webpage is accessible.
3.  On the target Nginx server, simulate a process crash:
    ```bash
    sudo pkill -9 nginx
    ```
4.  Wait for Prometheus to detect the failure (typically within 1-2 minutes).
5.  Observe the alert firing in Alertmanager and the corresponding event being received by the EDA listener.
6.  Navigate to the AAP UI. You will see the `AIOPS_Nginx_self_healing` workflow has been launched and is paused at the approval node.
7.  Check the administrator's email for the RCA and remediation playbook.
8.  Review and approve the workflow in the AAP UI.
9.  Observe as the final job runs, fixes the Nginx service, and the workflow completes successfully.
10. Verify that the Nginx webpage is accessible once again.

# AIOPS-POC/Demo center setup Guide

##	DEMO Scenario
![image](https://github.com/user-attachments/assets/806c835b-d59b-42b3-a98b-9777c83defed)
![image](https://github.com/user-attachments/assets/917b6e6a-8b06-4e9e-acee-19c64397c697)

##	DEMO Expecting results
![image](https://github.com/user-attachments/assets/6bb87e61-a521-4752-8007-6e2492151706)
![image](https://github.com/user-attachments/assets/6f0033ac-01a6-4f75-9218-e9d630fbb22f)
![image](https://github.com/user-attachments/assets/c24ce408-7415-4aad-b516-bf0bae7e570d)
![image](https://github.com/user-attachments/assets/acccaf3a-dc4b-4a5e-8c09-786c54f36fb0)

### Branch 1：Accepted the AI’s resolution 
![image](https://github.com/user-attachments/assets/c41edc1c-230b-42e1-ac1a-6976bb189008)

### Branch 2：Branch 2: Tuning the playbook w/ Gitops
![image](https://github.com/user-attachments/assets/39cdfa2b-92f7-4e57-8e41-466981ab2c30)


## License

This project is provided as a proof-of-concept. Please add an appropriate open-source license if you plan to extend it. An MIT or Apache 2.0 license is recommended.

               








