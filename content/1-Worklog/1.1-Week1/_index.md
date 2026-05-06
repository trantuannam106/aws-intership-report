---
title: "Week 1 Worklog"
date: 2026-04-20
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:

- Get familiar with the internship environment and members of the First Cloud Journey program.
- Grasp basic concepts of Cloud Computing and the AWS service ecosystem.
- Understand AWS Global Infrastructure, management tools, and cost optimization.
- Create an AWS Free Tier 2025 account and successfully receive the $200 credit.
- Complete hands-on labs regarding account setup, security, and budget management.

---

### Tasks to be Implemented This Week:

| Day | Task | Start Date | Completion Date | Documentation/Source |
| :--- | :--- | :--- | :--- | :--- |
| Mon | - Meet and greet FCJ members <br> - Read and note down regulations and rules at the internship unit | 2026-04-20 | 2026-04-20 | [Notion Group Description](https://www.notion.so/Group-description-TP-HCM-347df829a730809a8f63d39505644917) |
| Tue | - Module 01-01: What is Cloud Computing? <br> - Module 01-02: What Makes AWS Different? <br> - Module 01-03: How to Start the Journey to the Cloud <br> - Module 01-04: AWS Global Infrastructure | 2026-04-21 | 2026-04-21 | [AWS Cloud Journey](https://cloudjourney.awsstudygroup.com/) |
| Wed | - Module 01-05: AWS Service Management Tools <br> - Module 01-06: Cost Optimization on AWS and Working with AWS Support <br> - Module 01-07: Practice and Additional Research <br> - **Hands-on Practice:** <br>&emsp;+ Lab01-01: Create an AWS Account <br>&emsp;+ Lab01-02: Set up Virtual MFA Device <br>&emsp;+ Lab01-03: Create Admin Group and Admin User <br>&emsp;+ Lab01-04: Account Verification Support | 2026-04-22 | 2026-04-22 | [Lab Source 1](https://000001.awsstudygroup.com/) <br> [Lab Source 2](https://000002.awsstudygroup.com/) |
| Thu | - **Budget Practice:** <br>&emsp;+ Lab07-01: Create Budget using Templates <br>&emsp;+ Lab07-02: Cost Budget Tutorial <br>&emsp;+ Lab07-03: Create Usage Budget <br>&emsp;+ Lab07-04: Create Reservation Instance (RI) Budget <br>&emsp;+ Lab07-05: Create Savings Plans Budget <br>&emsp;+ Lab07-06: Budget Cleanup | 2026-04-23 | 2026-04-23 | [Lab Source 1](https://000001.awsstudygroup.com/) |
| Fri | - Complete 5 tasks to receive $200 credit: <br>&emsp;+ Launch EC2 Instance (+$20) <br>&emsp;+ Amazon Bedrock Playground (+$20) <br>&emsp;+ Set up AWS Budgets (+$20) <br>&emsp;+ Create Lambda Web App (+$20) <br>&emsp;+ Create RDS Database (+$20) <br> - Research: AWS Well-Architected Framework | 2026-04-24 | 2026-04-24 | [Lab Source 1](https://000001.awsstudygroup.com/) <br> [AWS Well-Architected](https://docs.aws.amazon.com/wellarchitected/) |

---

### Week 1 Achievements:

#### Knowledge

**Module 01-01 — What is Cloud Computing?**

- Understood Cloud Computing concepts: on-demand, pay-as-you-go.
- Distinguished between service models: IaaS, PaaS, SaaS.
- Distinguished between deployment models: Public Cloud, Private Cloud, Hybrid Cloud.

**Module 01-02 — What Makes AWS Different?**

- Recognized AWS as the world's largest cloud provider with over 200 services.
- Key advantages: High reliability, security, flexibility, and cost-effectiveness.
- Gained knowledge of main service groups:
  - Compute (EC2, Lambda)
  - Storage (S3, EBS)
  - Networking (VPC, Security Group)
  - Database (RDS, Aurora)
  - AI/ML (Amazon Bedrock)

**Module 01-03 — How to Start Your Cloud Journey**

- Understood the AWS learning roadmap from basic to advanced.
- Gained details on the AWS Free Tier 2025 program:
  - $100 credit immediately upon account creation.
  - An additional $100 from 5 practical tasks.
  - Two options: Free Plan (6-month protection) and Paid Plan (Full service access).

**Module 01-04 — AWS Global Infrastructure**

- Learned about AWS presence across multiple Regions and Availability Zones (AZs) worldwide.
- Understood the concepts of Region, AZ, and Edge Location.
- Learned how to choose the right Region based on latency, cost, and legal compliance.

**Module 01-05 — AWS Service Management Tools**

- AWS Management Console (Web interface).
- AWS CLI (Command line).
- AWS SDK (Code integration).
- AWS CloudShell.

**Module 01-06 — Cost Optimization and AWS Support**

- Cost management tools: AWS Budgets, Cost Explorer, Cost & Usage Report.
- Saving strategies: Reserved Instances, Savings Plans, Spot Instances.
- AWS Support plans: Basic, Developer, Business, Enterprise.
- How to create and manage support cases.

**Additional Research — AWS Well-Architected Framework**

- 6 Pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability.

---

#### Practice

**Lab01-01 — Create AWS Account:**

- Accessed [aws.amazon.com/free](https://aws.amazon.com/free) and created a new account.
- Selected **Paid Plan** for full service access.
- Received $100 credit automatically after successful creation.
- ![Proof: AWS Console screen after first login](/images/1-Worklog/1.1-Week1/aws-billing-100-credit.png)

**Lab01-02 — Set up Virtual MFA Device:**

- Activated MFA for the root account using an Authenticator app.
- ![Proof: MFA activation confirmation screen](/images/1-Worklog/1.1-Week1/mfa-activated.png)

**Lab01-03 — Create Admin Group and Admin User:**

- Created IAM Group `AdminGroup` with `AdministratorAccess` policy.
- Created IAM User and assigned it to the group.
- ![Proof: List of IAM Users and Groups in Console](/images/1-Worklog/1.1-Week1/iam-users-groups-console.png)

**Lab01-04 — Account Verification Support:**

- Practiced contacting AWS Support for account verification when necessary.
- ![Proof: Support case creation success screen](/images/1-Worklog/1.1-Week1/support-case-success.png)

**Lab07 — AWS Budgets Practice:**

- Lab07-01: Created a quick Budget using existing Templates.
- Lab07-02: Created a Cost Budget with email alert thresholds.
- Lab07-03: Created a Usage Budget based on usage units (hours, GB...).
- Lab07-04: Created an RI Budget to track Reserved Instances.
- Lab07-05: Created a Savings Plans Budget.
- Lab07-06: Deleted all Budgets after practice (cleanup).
- 📸 _Proof: List of created Budgets in the AWS Budgets Console._

**5 Tasks to Receive $200 Credit:**

- **Task 1 — Launch EC2 Instance (+$20):**
  - Created an EC2 instance named `Test Instance`, selected AMI, and configured Security Group.
  - Created key pair `first-kp` (RSA, .pem).
  - Terminated instance after completion (cleanup).
  - 📸 _Proof: EC2 instance in 'running' state._

- **Task 2 — Amazon Bedrock Playground (+$20):**
  - Accessed Bedrock Console, selected **Claude 3 Haiku** model.
  - Submitted use case details and tested a prompt.
  - 📸 _Proof: Response results from Bedrock Playground._

- **Task 3 — Set up AWS Budgets (+$20):**
  - Created a cost budget with email notifications.
  - 📸 _Proof: Budget successfully created._

- **Task 4 — Create Lambda Web App (+$20):**
  - Created Lambda function `http-function-url-tutorial` from a blueprint.
  - Deleted function after completion (cleanup).
  - 📸 _Proof: Lambda function successfully created._

- **Task 5 — Create RDS Database (+$20):**
  - Created Aurora (PostgreSQL Compatible) database using 'Easy Create'.
  - Deleted DB instance and cluster after completion (cleanup).
  - 📸 _Proof: RDS database in 'Available' state._

- 📸 _Overall Proof: AWS Billing Console showing the full $200 credit._

---

#### Challenges and Solutions

**Challenges:**

- Did not fully understand the difference between Free Plan and Paid Plan during account creation.
- Authorization error when activating Amazon Bedrock (Claude 3 Haiku).
- Initially unfamiliar with the AWS Console interface.
- Confusion between different Budget types (Cost, Usage, RI, Savings Plans).

**Solutions:**

- Carefully read the Free Plan vs. Paid Plan comparison before registering.
- Submitted an AWS Support case (Account & billing > Bedrock Allowlisting) and waited 24h for approval.
- Practiced repeatedly on the Console to get used to the UI.
- Re-read the documentation for each Budget type and followed the labs step-by-step.

---

#### Lessons Learned

- Understood the Cloud Computing model and why AWS is currently the most popular platform.
- Always **enable MFA** and use an IAM User instead of the root account for daily tasks.
- **Set up Budgets immediately** after account creation to avoid unexpected costs.
- Always **clean up resources** after practice (terminate EC2, delete RDS, delete Lambda).
- The AWS Well-Architected Framework is a vital guide when designing systems on the cloud.