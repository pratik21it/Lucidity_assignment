# Multi-Account AWS EC2 Disk Utilization Monitoring

This project implements a secure, multi-account monitoring framework using Ansible to track disk utilization across an enterprise AWS environment. It automates the collection and aggregation of metrics into a consolidated HTML dashboard. The architecture is designed for extensibility, producing structured output that can be ingested by AWS Athena to generate queryable data tables.

## Key Components

1. Ansible Control Node: The central orchestration server.

2. AWS IAM Cross-Account Roles: Ensures secure access without managing long-lived access keys.

3. Dynamic Inventory (aws_ec2): Automatically discovers instances across all accounts.

4. Jinja2 Templating: Aggregates raw data into a user-friendly HTML dashboard.


## Solution Featrures

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

## Setup and Configuration

1. IAM Configuration (Security)
    In Member Accounts (Target): Create a role named CrossAccountMonitorRole with the following Trust Policy:

    {
    "Version": "2012-10-17",
    "Statement": [
        {
        "Effect": "Allow",
        "Principal": { "AWS": "arn:aws:iam::MANAGEMENT_ACCOUNT_ID:root" },
        "Action": "sts:AssumeRole"
        }
    ]
    }
    In Management Account (Source): Attach a policy to the Ansible Control Node allowing it to assume the roles:

    {
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Action": "sts:AssumeRole",
        "Resource": "arn:aws:iam::*:role/CrossAccountMonitorRole"
    }
    }


2. Inventory Configuration
    Update inventory/aws_ec2.yml with relevant regions or tag filters.

## Usage
To run the monitoring solution and generate the report:

### Clone the repository:

    git clone https://github.com/pratik21it/Lucidity_assignment.git

    cd LUCIDITY

### Run the Playbook:

    ansible-playbook -i inventory/aws_ec2.yml playbook.yml

### View the Report: 
    Open the generated Disk_Utilization_Report.html in any web browser.