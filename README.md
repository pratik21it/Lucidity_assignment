# Multi-Account AWS EC2 Disk Utilization Monitoring

## 📖 Project Overview
This project provides a **centralized, scalable, and secure** solution to monitor disk utilization across multiple AWS accounts using **Ansible**.

Designed for an enterprise environment managing three distinct AWS accounts, this solution leverages **Dynamic Inventory** and **IAM Cross-Account Roles** to automatically discover instances and report their disk usage without installing agents.

### Key Features
* **Agentless Architecture:** Uses standard SSH and Python; no agents required on target servers.
* **Zero-Touch Scalability:** Automatically detects new instances and accounts via AWS API.
* **Dynamic Key Mapping:** Automatically selects the correct SSH Private Key for each instance based on its AWS Key Pair name.
* **Consolidated Reporting:** Generates a single HTML dashboard summarizing metrics from all accounts.

---

## 🏗 Architecture Design

The solution utilizes a **Hub-and-Spoke** security model:

* **The Hub (Management Account):** Hosts the Ansible Control Node.
* **The Spokes (Member Accounts):** Contain the target workloads.
* **Security Layers:**
    1.  **Discovery Layer:** Uses **IAM Roles** (No long-lived Access Keys) to query the AWS API.
    2.  **Connection Layer:** Uses **SSH Keys** (stored securely) to connect to instances.

## Key Components

1. Ansible Control Node: The central orchestration server.

2. AWS IAM Cross-Account Roles: Ensures secure access without managing long-lived access keys.

3. Dynamic Inventory (aws_ec2): Automatically discovers instances across all accounts.

4. Jinja2 Templating: Aggregates raw data into a user-friendly HTML dashboard.


## Solution Features

1. Centralized Access & Management 

    Static inventory files and shared private keys are avoided. Instead, AWS IAM Roles are used.

    A uniform role (CrossAccountMonitorRole) is deployed to all member accounts.

    The Ansible Control Node assumes this role using AWS STS (Security Token Service) to execute commands.

2. Data Aggregation 

    Parallel Execution: Ansible collects disk metrics (df -h) from all instances simultaneously.

    Reporting: A local task aggregates the "facts" gathered from all hosts and renders them into a color-coded HTML report using Jinja2, highlighting instances with usage >80%.

3. Scalability 

    The solution is designed for rapid acquisition integration.

    Auto-Discovery: The aws_ec2 dynamic inventory plugin is used.

    Future Proof: If a new company/account is acquired, the new Account ID can be simply added to the inventory config. No changes to the playbook code are required.


## Implementation Guide (Step-by-Step)
### Step 1: IAM Security Setup
To avoid using hardcoded AWS Access Keys/Secret Keys, we use IAM Roles.

In Member Accounts (Targets):

Create a Role: CrossAccountMonitorRole.

Trust Policy: Allow the Management Account ID to assume this role.

Permissions: AmazonEC2ReadOnlyAccess.

In Management Account (Hub):

Create a Role: AnsibleControllerRole.

Permissions: Allow sts:AssumeRole on the Member Account roles.

Attach this role to the EC2 instance running Ansible.

### Step 2: Secure Key Management
Since different accounts/regions use different SSH keys, we created a centralized, secured key store.

Created a directory: /home/ec2-user/keys/.

Uploaded all necessary .pem files (prat-ansible.pem, target1.pem) to this directory.

Secured the directory:

Bash

sudo chown -R ec2-user:ec2-user /home/ec2-user/keys/
chmod 400 /home/ec2-user/keys/*.pem
### Step 3: Directory Structure & Inventory
We use the "Directory as Inventory" approach. Ansible reads every YAML file in the inventory/ folder and merges them.

Plaintext

.
├── ansible.cfg            # Global Config
├── playbook.yml           # Automation Logic
├── keys/                  # (Gitignored) Contains .pem files
├── templates/
│   └── report.html.j2     # HTML Report Template
└── inventory/
    ├── 01_master.aws_ec2.yml    # Config for Management Account
    └── 02_member.aws_ec2.yml    # Config for Member Accounts
### Step 4: Dynamic Key Mapping (Crucial Configuration)
To solve the "Permission Denied" issues caused by mismatched keys, we implemented a dynamic Jinja2 expression in the inventory compose section.

Code Snippet from inventory/01_master.aws_ec2.yml:

YAML

compose:
  ansible_host: public_ip_address
  # Automatically picks the key file that matches the AWS Key Pair Name
  ansible_ssh_private_key_file: "'/home/ec2-user/keys/' + key_name + '.pem'"
This ensures that if an instance uses target1 key, Ansible automatically looks for /home/ec2-user/keys/target1.pem.

## Usage
### Prerequisites
* Ansible installed on Control Node.

* Python 3 with boto3 and botocore.

* SSH connectivity (Port 22) open from Control Node to Targets.

### Running the Solution
Run the playbook using the ec2-user. Do not pass a private key flag; the inventory handles it automatically.
Command for running the playbook: 

ansible-playbook -i inventory/ playbook.yml -u ec2-user

### Results
* Ansible scans all accounts.

* Connects via SSH using the correct keys.

* Generates an HTML report: Disk_Utilization_Report.html.