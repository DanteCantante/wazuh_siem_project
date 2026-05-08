# wazuh_siem_project
This project demonstrates the deployment of a Wazuh SIEM (Security Information and Event Management) instance and the subsequent monitoring of a Parrot OS endpoint. The goal was to simulate common adversarial behaviors—such as privilege escalation attempts and SSH brute-forcing—to verify telemetry collection.

## Lab Environment
* **SIEM:** Wazuh Manager (v4.14.4) deployed on Ubuntu Server.
* **Endpoint:** Parrot Security OS (Agent ID: 001) with Wazuh Agent installed.
* **Network:** Private lab environment.

---

## Project Walkthrough

### 1. Deployment and Agent Integration
The Wazuh Manager was deployed on a dedicated Ubuntu server. Following the setup, the Wazuh agent was installed and configured on the Parrot OS "victim" machine.

![Successful connection](<img width="2048" height="1280" alt="01" src="https://github.com/user-attachments/assets/80455ae6-8c2d-47c0-babd-54fc7a58767f" />
)
*Figure 1: Successful connection of the Parrot OS agent (diogenes) to the Wazuh Manager.*

### 2. Manual Telemetry Generation (Sudo Failures)
To test initial telemetry, I performed several failed `sudo` attempts on the Parrot OS machine. This simulates a user or an unauthorized person attempting to gain root access. Eventually I simulated a succesful login attempt. 

| Action | Resulting Telemetry |
| :--- | :--- |
| **Failed Sudo** <br> ![Sudo Attempt](<img width="2048" height="1280" alt="03" src="https://github.com/user-attachments/assets/84dee3d1-06ad-4fdb-bc4f-272e5b7e6625" />
) | **Rule 5403:** First time user executed sudo <br> **Rule 5503:** User login failed <br> ![Telemetry Result](<img width="2048" height="1280" alt="04" src="https://github.com/user-attachments/assets/1db59fe8-74e6-49d0-927e-99f0611a3140" />) |

### 3. Simulating a Brute Force Attack
Using Hydra, I targeted the local SSH service on the Parrot OS machine to simulate a Credential Access attack.

* **Preparation:** Configured and enabled the local SSH service.
![SSH Configuration](<img width="2048" height="1280" alt="05" src="https://github.com/user-attachments/assets/72fca20d-e78a-4746-b65f-c500d40a4367" />)

* **Execution:** Ran Hydra against the local SSH port using the `rockyou.txt` wordlist.
![Hydra Attack Execution](<img width="2048" height="1280" alt="06" src="https://github.com/user-attachments/assets/408cbce8-ebad-4430-a11f-45a761b59b56" />)

### 4. SIEM Analysis & Detection
The SIEM successfully captured the spike in traffic. The telemetry clearly shows the transition from normal background noise to a high-density event period.

![Traffic Spike](<img width="2048" height="1280" alt="07-01" src="https://github.com/user-attachments/assets/730a5a62-8c51-4c8c-bebf-dedb0d277f0e" />) (<img width="2048" height="1280" alt="07-02" src="https://github.com/user-attachments/assets/e68a1b26-f9f4-47a4-99a2-aee4afe77145" />) (<img width="2048" height="1280" alt="07-03" src="https://github.com/user-attachments/assets/76af04f4-a784-4d45-b381-6ed4e82cd01b" />)
*Figure 2: The Events Count Evolution graph showing a sharp spike during the Hydra attack.*

Wazuh mapped these events directly to the **MITRE ATT&CK** framework, specifically identifying **T1110 (Brute Force)** under the **Credential Access** tactic.

### 5. Event Drill-down
By inspecting the specific alerts, we can see the granularity of the detection. The SIEM identified "User missed the password more than one time" with a Level 10 severity rating.

![Event Details](<img width="2048" height="1280" alt="08" src="https://github.com/user-attachments/assets/aeeec35f-510b-4aeb-8929-460c62672486" />)
*Figure 3: Deep dive into Rule ID 2502, showing 17 fired times within a short window.*

---

## Key Findings
* **Alert Accuracy:** The Wazuh default ruleset effectively identified the brute force attempt without requiring custom regex.
* **Mapping:** Seeing the events mapped to MITRE ATT&CK tactics allows for a higher-level understanding of the "kill chain" during an incident.
* **Visibility:** Real-time monitoring provided immediate feedback on the success of the attack simulation.

