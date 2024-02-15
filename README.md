# Wifimon Analysis Server and Wifimon Test Server Installation Guide (Ansible)

## Introduction
**The WiFiMon Analysis Server (WAS)** is the core component of WiFiMon which gathers and processes all the measurement data. The WAS receives the following data:

- Results of crowdsourced measurements streamed from end users (**WiFiMon Software Probes**, WSP's) within the monitored WiFi networks.
- Results of measurements from fixed points within the Wi-Fi network, which are streamed from **WiFiMon Hardware Probes** (WHP's) within the monitored networks.
- RADIUS and DHCP logs from RADIUS and DHCP servers respectively.
- Wireless network performance metrics streamed from WHP's.
- Two-Way Active Measurement Protocol (TWAMP) measurements between WHP's and the **WiFiMon Test Server** (WTS).
- System metrics from WHP's, such statistics about the CPU of the probes as well as the available memory and storage space.

The WAS mainly consists of two software components: A - The WiFiMon Agent and B - The WiFiMon GUI.

**A - WiFiMon Agent**

The WiFiMon Agent is responsible for performing the following actions: 

- The analysis of crowdsourced measurements. These measurements are received from end users and WAS correlates them with information received from RADIUS and DHCP Logs when/if this information is available.
- The analysis of hardware probe measurements. These measurements are received from WHP's and  WAS correlates them with information received from RADIUS and DHCP Logs when/if this information is  available.
- The analysis of wireless network metrics received from WHP's.
- The analysis of TWAMP measurement results received from WHP's.
- Storing the results of analysis and correlation.
- Detecting anomalies in WiFiMon performance measurements

WiFiMon Agent operates over HTTPS, i.e. measurements are streamed over HTTPS.

**B - WiFiMon GUI**

 The WiFiMon GUI provides a graphical representation of the measurement results and various analyses as described above.

## Requirements

Supported OS
- Ubuntu 22.04
- Debian 11

Minimal Hardware Requirements
- 4 CPU Core
- 8 GB RAM
- 10 GB free space

Software Requirements
- Ansible 2.10.0 or higher
- Python 3.10 or higher

## Single Node Installation (All in One)

WiFiMon Ansible Playbook may be used to easily install the WiFiMon Analysis Server (WAS) and Wifimon Test Server (WTS).

1. Clone the project by running:
   ~~~
   git clone https://gitlab.grena.ge/nugzar/wifimon-ansible.git
   ~~~

2. CD to project directory:
   ~~~
   cd wifimon-ansible
   ~~~

3. Create a DNS record (A) for the following names. They should point to the same IP.
   - <hostname>.<suffix>
   - <hostname>-ui.<suffix>
   - <hostname>-flask.<suffix>
   - <hostname>-anomaly.<suffix>
   - <hostname>-elastic.<suffix>
   - <hostname>-kibana.<suffix>
   - <hostname>-wts.<suffix>
   
   > **Note:** This playbook includes installation of  **WiFiMon Test Server** (WTS) on `<hostname>-wts.<suffix>` (that you'll set in `vars/main.yml`) 

4. Ensure that all of them have the same public IP address and their TCP ports 80,443 are accessible.


5. In `hosts.cfg` file replace IP addresses with the IP address (or FQDN) of the server on which you plan to install WAS.


6. Add your SSH public key to `~/.ssh/authorized_keys`
   ~~~
   ssh-keygen -o
   cat ~/.ssh/id_rsa.pub
   ~~~
7. Ensure that your ansible machine has the root access over SSH to the server where you want to install WAS and WTS.
   ~~~
   ansible -m ping WAS
   ~~~
   
8. Adjust variables in `vars/main.yml`. **Do not use the default passwords in production! Set your own secure passwords!**

   > **Note:** Be aware to use the following values:
   > - ELK stack version: 8.8
   > - PostgreSQL version: 15 
   > - WiFiMon agent version: 2.2.0

9. Run the playbook
   ~~~
   ansible-playbook wifimon.yml
   ~~~
10. Wait a few minutes (approximately 15 mins)

11. Access the WiFiMon web-ui `<hostname>-ui.<suffix>`.
    Credentials are defined in `vars/main.yml`:
   ```
   # Credentials for first login window (wifimon)
   email: <wifimon_admin_email>
   password: <wifimon_admin_pass>
   # Credentials for second login window (kibana)
   username: elastic
   password: <elastic_elasticsearch_password>
   ```

12. To send the logs to the Logstash using Filebeat, configure output section in `/etc/filebeat/filebeat.yml` as follow:
   ~~~
   output.logstash:
      hosts: ["<hostname>-elastic.<suffix>:5044"]
      ssl.certificate_authorities:
            - /etc/ssl/certs/ca-certificates.crt   
   ~~~
