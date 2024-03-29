# Wireguard Server Set Up with Ansible
This Ansible Playbook offers a streamlined solution for automating the setup and management of an Wireguard server on a Linux system. Leveraging Ansible introduces the powerful principle of idempotency, guaranteeing that the results remain consistent, no matter how frequently the Playbook is executed. This inherent characteristic simplifies updates, especially when fine-tuning specific sections of the Playbook.

This Playbook will be ran on the Control Node (Device controlling Servers).

Before running our Playbook, we would need to set up our environment:
```
sudo apt update && sudo apt upgrade -y
sudo apt install ansible sshpass -y
sudo mkdir /etc/ansible
```

# Table of Contents
1. [Part I: Source Files](#part-i-source-files)
2. [Part II: Usage](#part-ii-usage)
3. [Menu](#part-iii-menu)
4. [Roles](#part-iv-roles)
    1. [serverSetUp](#1-serversetup)
    2. [createClientCertificate](#2-createclientconfig)
    3. [revokeClientAccess](#3-revokeclientaccess)

## Part I: Source Files
Files needed in order for the Ansible Script to Work:
```
├── /etc/ansible/                                       # Ansible Folder where all Playbooks are Stored
│   ├── /main.yml                                       # Main Playbook to be Ran
│   ├── /hosts                                          # Host File to Specify servers to be affected by Ansible
│   ├── /ansible.cfg                                    # Ansible Configuration Settings
│   ├── /roles                                          # Roles Folder containing Roles
│   │   ├── /serverSetUp/                               # Folder containing Files related to the serverSetUp role
│   │   |   ├── tasks/                                  # Folder containing Tasks called by the serverSetUp role
|   |   |   |   ├── /main.yml                           # Main Task 
|   |   |   |   ├── /setUpIPTables.yml                  # Task File to establish IPTable Rules
|   |   |   |   ├── /config.j2                          # Jinja2 Template File
│   │   |   ├── vars/                                   # Folder containing files that define variables
│   │   |   |   ├── /main.yml                           # Main vars file to define variables
│   │   |   ├── handlers/                               # Folder to contain handlers
│   │   |   |   ├── /main.yml                           # Main handler file
│   │   ├── /createClientConfig/                        # Folder containing Files related to the createClientCertificate role
│   │   |   ├── tasks/                                  # Folder containing Tasks called by the serverSetUp role
│   │   |   |   ├── /main.yml                           # Main Task File
│   │   |   ├── vars/                                   # Vars Folder
│   │   |   |   ├── /main.yml                           # Main Vars File
│   │   |   ├── handlers/                               # Folder to contain handlers
│   │   |   |   ├── /main.yml                           # Main handler file
│   │   ├── /revokeClientCertificate/                   # Folder containing Files related to the revokeClientCertificate role
│   │   |   ├── tasks/                                  # Task Folder
│   │   |   |   ├── /main.yml                           # Main Task File
│   │   |   ├── vars/                                   # Vars Folder
│   │   |   |   ├── /main.yml                           # Main Vars File
│   │   |   ├── handlers/                               # Handlers Folder
│   │   |   |   ├── /main.yml                           # Main Handlers File
```

## Part II: Usage
Ansible should be ran from the `/etc/ansible/main.yml` playbook.

Command to Run Ansible Playbook:
```
sudo ansible-playbook main.yml
```

Prior to executing the script, it is highly encouraged to perform a system update and upgrade, ensuring that your system has the latest software versions. This proactive step not only saves time but also optimizes resource utilization for a smoother script execution. It's worth noting that the script itself will handle system update and upgrade if it hasn't been done already, making this step optional.

Run the following command to Update and Upgrade the System:
```
sudo apt update && sudo apt upgrade -y
```

## Part III: Menu
The Playbook incorporates a user-friendly interface, allowing users to make selections tailored to their specific needs or purposes.

```
WIREGUARD SERVER:
=====================================
1. Server Set Up
2. Create Client Config
3. Revoke Client Access
Enter Choice > 
```

Each user selection within the script directs the user to a specific role, each designed to fulfill a unique purpose. These roles encapsulate distinct functionalities, enabling a modular and organized approach to the automation process. Whether it's server setup, client certificate creation, or other tasks, the user is guided to the appropriate role for seamless and purpose-driven execution.

## Part IV: Roles
1. [serverSetUp](#1-serversetup)
2. [createClientCertificate](#2-createclientcertificate)
3. [revokeClientAccess](#3-revokeclientaccess)

### 1. serverSetUp
This role performs the following steps to set up the Wireguard Server:
1. [Updating and Upgrading of System](#1-updating-and-upgrading-of-system)
2. [Installation of Packages](#2-installation-of-packages)
3. [Generation of Keys](#3-generation-of-keys)
4. [Creation of Server Configuration](#4-creation-of-server-configuration)
5. [Server Hardening](#5-server-hardening)

<u>_Variables_</u>

Certain Variables can be changed as per your needs. Relevant description on the different variables will be displayed below.

| Variable Name | Description |
| :- | :- |
| wireguardDirectory | Wireguard Server Directory |
| configPath | Full Path of Wireguard Server Configuration File |
| interface | WAN Interface of System |
| serverIP | IP of Server in Tunnel |
| serverPort | Port to set Wireguard to Run On |
| sshPort | Port to set SSH to Run On |
| chains | INPUT and OUTPUT chains (edited in Playbook) |

<u>_Features (Tasks)_</u>

##### 1. Updating and Upgrading of System
As the first step, the Playbook initiates the updating and upgrading of the system. This essential process ensures that both the system and its package manager are brought up to the latest versions, providing improved security, bug fixes, and access to the latest features and enhancements.

##### 2. Installation of Packages
Necessary packages required for setting up the Wireguard Server are installed.

List of Packages Installed by Script:
- wireguard
- iptables
- iptables-persistent
- rsyslog

**_NOTE:_**  The installation of `rsyslog` is included to revert back to using `/var/log/auth.log` on Linux systems (such as Debian) that utilize journalctl to store authorization information. If your system already uses `/var/log/auth.log` for authorization information, the installation of this package is not necessary.

##### 3. Generation of Keys
Both Public and Private Keys are generated with the `wg` utility and saved within Wireguard's Directory specified by the `wireguardDirectory` variable. These Keys will be added into the Wireguard Configuration File for authentication purposes.

##### 4. Creation of Server Configuration
After successfully generating the necessary server keys for WireGuard, the next crucial step is to seamlessly integrate them into the WireGuard server configuration file, accompanied by other essential parameters. This process ensures the secure and efficient operation of the WireGuard VPN setup.

##### 5. Server Hardening
The server undergoes a robust hardening process, incorporating SSH Hardening, and the establishment of IPTable Rules. These measures collectively enhance the security posture by fortifying the SSH configuration, and defining strict IP filtering rules.

### 2. createClientConfig
This role facilitates the [Creation of Client Configuration](#creation-of-client-configuration) based off the Keys and Information provided by the Client through prompts.

Before running the Playbook, the Client needs to install the necessary packages and generate the keys:
```
sudo apt update && sudo apt upgrade -y
sudo apt install wireguard openresolv -y
cd /etc/wireguard
wg genkey | tee privatekey | wg pubkey > publickey
```

<u>_Variables_</u>

Certain Variables can be changed as per your needs. Relevant description on the different variables will be displayed below.

| Variable Name | Description |
| :- | :- |
| wireguardDirectory | Wireguard Server Directory |
| serverConfigPath | Full Path of Wireguard Server Configuration File |
| serverIP | Wireguard Server's IP for Client to Connect To (WAN Interface) |
| serverPort | Port to set Wireguard to Run On |
| dnsServerIP | IP Address of DNS Server used by Client |

<u>_Features (Tasks)_</u>

##### Creation of Client Configuration
Based off the information given by the Client through prompts, the Wireguard server crafts up a Client Configuration that allows the Client to connect to the Wireguard Server. After successful creation and configuraiton, the Client Configuration File is then relocated to the `/tmp` folder, facilitating easy retrieval by the client through transfer methods like using `scp`.

The Client will then be set as a peer in the Wireguard Configuration File.

<u>_Connecting to the OpenVPN Server_</u>

Before connecting to the OpenVPN Server, it is best for the client to Update and Upgrade the System by Running:
```
sudo apt update && sudo apt upgrade -y
```

Clients can do a quick start up of the connection/tunnel using:
```
sudo wg-quick up wg0
```

Clients can also set their system such that it connects to the system upon startup using:
```
sudo systemctl enable --now wg-quick@wg0
```

### 3. revokeClientAccess
This role facilitates the [Revoking of Client Access](#revoking-of-client-access) to the Wireguard Server.

<u>_Variables_</u>

Certain Variables can be changed as per your needs. Relevant description on the different variables will be displayed below.

| Variable Name | Description |
| :- | :- |
| configFileLocation | Full Path of Wireguard Configuration File |

<u>_Features (Tasks)_</u>

##### Revoking of Client Access
Client access is removed through removing the `[Peer]` portion where the Client's Public Key matches the Public Key specified. The Wireguard Service would need to be stopped before the removal to ensure successful removal.