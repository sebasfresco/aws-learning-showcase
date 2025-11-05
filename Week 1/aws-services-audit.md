# FIRST PRINCIPLES THINKING

### Every AWS service I've used in production.

## S3
All things storage. Files, backups, static content. Pay $0.023/GB/month.

**vs EBS:** EBS is block storage bolted to EC2. Fast, local. S3 is object storage over HTTP. Accessible from anywhere.

**vs EFS:** EFS is NFS for when you need multiple EC2s hitting the same filesystem. Costs $0.30/GB/month (13x more than S3). Only use when you actually need concurrent mounts.

**vs Glacier:** Storage class inside S3. Set lifecycle policies, data moves to Glacier at $0.004/GB/month. Takes hours to retrieve.

**vs Azure/Google:** S3 is oldest, most mature. Every tool expects S3.

**Why I chose it:** Everyone uses it. Default for everything.

---

## Lambda
Run code without managing servers. Pay per 100ms of execution. 15-minute hard limit. No persistent state between runs.

**vs EC2:** EC2 runs 24/7, pay hourly whether you use it or not. Lambda runs on-demand, pay per millisecond. Lambda for event-driven tasks under 15 mins. EC2 for long-running or stateful apps.

**vs Fargate:** Fargate runs containers. Lambda runs functions. Already containerized? Use Fargate. Just deploying code? Lambda.

**vs ECS/EKS:** Full orchestration platforms. Overkill unless running dozens of microservices.

**vs Step Functions:** Step Functions chains Lambdas into workflows. Lambda is the execution unit.

**Why I chose it:** Had a 2-minute job triggered by file uploads. EC2 24/7 = $10/month. Lambda = $0.40/month. Simple cost efficiency.

---

## EC2
Virtual servers. Pick CPU/RAM/disk. Full root access. Any OS you want.

**vs Lambda:** Lambda maxes out at 15 mins and can't hold state. EC2 runs indefinitely with full persistence. Web server 24/7? EC2. Multi-hour batch jobs? EC2.

**vs Lightsail:** Fixed pricing ($3.50, $5, $10/month), simplified UI. Less control. Good for basic WordPress deployments or learning.

**vs ECS/EKS:** Container orchestration. Use if running Docker. Otherwise stick to EC2 for traditional deployments.

**vs Fargate:** ECS without managing instances yourself. More expensive, less control. EC2 gives you everything.

**Why I chose it:** Small web server + MySQL for ~100 users. t3.micro = $7.50/month. Straightforward.

---

## RDS
Managed databases. MySQL, PostgreSQL, SQL Server, Oracle, MariaDB. AWS handles backups, patches, failover.

**vs self-managed on EC2:** You do everything yourself on EC2 - backups, patches, replication. RDS automates it but costs 30-50% more.

**vs Aurora:** AWS proprietary (MySQL/PostgreSQL compatible). 5x faster, auto-failover across 3 AZs, scales to 128TB. Costs 20% more. For high-traffic production only.

**vs DynamoDB:** NoSQL key-value. No joins, no transactions. RDS is relational SQL with full JOIN support. Depends on your data model.

**vs Aurora Serverless:** Auto-scales capacity vs fixed instance sizing. Good for unpredictable traffic, bad for cost predictability.

**vs Redshift:** Data warehouse (OLAP). RDS is transactional (OLTP).

**Why I chose it:** Haven't yet. Running MySQL on EC2. 2GB database, single user. RDS would add $15/month for features I don't need right now. Will migrate when I need HA/automated backups.

---

## IAM
Access control. Create users, assign permissions, manage roles. Controls Console access and API keys.

**vs AWS SSO/Identity Center:** IAM is direct auth. SSO federates with external providers (Google, AD). Small orgs use IAM. Enterprises use SSO.

**vs third-party (Okta, Azure AD):** External IdPs authenticate then pass to IAM for enforcement.

**Why I chose it:** Mandatory. No way around it.

---

## VPC
Private network in AWS. Define CIDR blocks, subnets, route tables, security groups.

**Components:**
- Subnets: Public (internet) or private (isolated)
- Security Groups: Stateful firewall per resource
- Route Tables: Traffic routing

**vs default VPC:** Auto-created (172.31.0.0/16). Fine for testing. Custom VPC for production when you need specific IP ranges or complex networking.

**Why I chose it:** Mandatory. Default works for simple stuff, custom for production.

---

## CloudWatch
Metrics, logs, alarms. Watches CPU, disk, network. Set thresholds to trigger actions (SNS, autoscaling, Lambda).

Basic metrics free. Custom metrics $0.30/metric/month.

**vs third-party (Datadog, New Relic):** CloudWatch free and pre-integrated. Third-party has better UI/dashboards but costs $15+/host/month. CloudWatch good enough for now.

**vs EventBridge:** CloudWatch monitors metrics. EventBridge handles discrete events (S3 upload, state changes).

**vs X-Ray:** X-Ray traces requests through distributed systems. CloudWatch shows aggregate metrics.

**Why I chose it:** Free, already there. Set up billing alarms in 5 mins.

---

## Route 53
DNS hosting and domain registration.

**Two things:**
1. Buy domains ($12/year)
2. Manage DNS records (A, CNAME, MX, etc.)

**vs external DNS (Cloudflare, GoDaddy):** Route 53 integrates with AWS via alias records (point to ALB/CloudFront without IPs). Health checks for auto-failover. More edge locations.

**vs registrar DNS:** Most registrars have free basic DNS. Route 53 has better AWS integration.

**Cost:** $0.50/month per hosted zone + $0.40/million queries. About $0.60/month total.

**Why I chose it:** Everything in AWS. Simpler.

---

## SNS
Pub/sub messaging. One message in, pushes to all subscribers (email, SMS, HTTP, Lambda, SQS).

**My use:** CloudWatch billing alarm → SNS → email.

**vs SES:** SES does real email (HTML, attachments, templates). SNS does plain text alerts. Use SNS for notifications, SES for actual email systems.

**vs SQS:** SQS is pull-based queue. SNS pushes immediately. Use SQS for async processing, SNS for instant notifications.

**vs EventBridge:** EventBridge has complex routing. SNS is simple broadcast.

**Why I chose it:** CloudWatch integrates directly. Done in 2 mins.

---

## Billing and Cost Management
AWS billing console. Invoices, spending breakdowns, payment methods.

**Includes:**
- Invoices with line items
- Cost Explorer for spending graphs
- Budgets for alerts
- Payment methods

**vs Cost Explorer:** Part of Billing, not separate.

**vs CloudWatch billing metrics:** CloudWatch shows real-time estimates. Billing shows finalized invoices.

**vs third-party (CloudHealth):** Enterprise tools for massive bills. Overkill.

**Why I chose it:** Mandatory.

---

## SES / WorkMail
**SES:** Transactional email API. Password resets, order confirmations, marketing. $0.10 per 1,000 emails.

**WorkMail:** Hosted email. Mailboxes at your domain. $4/user/month.

**My setup:** WorkMail for mailbox, SES for automated emails.

**SES vs SNS:** SNS is plain text only. SES has full email features (HTML, attachments, templates, bounce handling).

**WorkMail vs Google/Microsoft:** WorkMail $4/month, Google $6/month. Integrates with AWS (IAM, KMS). No calendar sharing or collab docs. Good for basic email.

**SES vs SendGrid/Mailgun:** SendGrid has easier UI. SES cheaper at scale but needs code.

**Why I chose it:** Own domain, want email without paying Google. WorkMail + SES = $4/month total.