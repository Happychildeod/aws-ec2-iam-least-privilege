# Cloud Security in AWS: IAM Least Privilege for EC2

## Overview

This project secures Amazon EC2 instances using **AWS Identity and Access Management (IAM)** and the **principle of least privilege**.

Scenario:

- There are **two EC2 instances**:
  - `audit` – critical instance that must not be stopped or started by regular users.
  - `sales` – application instance that standard users are allowed to start/stop.
- IAM users **should be able** to start/stop only the `sales` instance.
- IAM users **must be blocked** from starting/stopping the `audit` instance.

This is implemented using **EC2 tags + IAM policies**.

**Created by:** Emmanuel Oladeinde  
**Role:** Cyber Security Analyst  
**Date:** November 2025  

---

## Architecture & Scope

**AWS Services Used**

- **AWS IAM**
  - Users, Groups, Policies, Account Alias
- **Amazon EC2**
  - Instance creation, tagging, start/stop actions
- **IAM Policy Language (JSON)**
  - Effect / Action / Resource / Condition

**High-Level Design**

1. Launch two EC2 instances: `audit` and `sales`.
2. Tag each instance with `Environment`:
   - `audit` → `Environment=Audit`
   - `sales` → `Environment=Sales`
3. Create an IAM policy that:
   - Allows read-only EC2 access.
   - Allows start/stop only when `Environment=Sales`.
4. Attach the policy to an IAM group.
5. Add IAM users to the group.
6. Log in as an IAM user and test allowed vs denied actions.

---

## Tagging Strategy

| Instance | Tag Key     | Tag Value |
|----------|------------|-----------|
| audit    | Environment | Audit     |
| sales    | Environment | Sales     |

The IAM policy uses these tags to control which instances can be started/stopped.

---

## IAM Policy (JSON)

Save this as, for example, `policies/PurelyessAuditEnvPolicy.json` in the repo and as a customer-managed policy in AWS:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadOnlyEC2",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus",
        "ec2:DescribeTags"
      ],
      "Resource": "*"
    },
    {
      "Sid": "AllowStartStopSalesOnly",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Environment": "Sales"
        }
      }
    }
  ]
}
