# VPC Architecture Decision Document  

This document captures the architectural reasoning behind building a production-aligned VPC from scratch.  
Engineering leaders document decisions so stakeholders understand **why** a design exists — not just what was built.

This ADR-style record covers:

- **Problem Statement**
- **Options Considered**
- **Decision**
- **Trade-offs**
- **Cost Impact**
- **Final Architecture Summary**
- **Lessons Learned**

---

# 1. Problem Statement

Design a foundational AWS network that aligns with real production environments:

- Segmented **public** and **private** subnets  
- Public ingress via Internet Gateway  
- **Outbound-only** internet for private workloads (updates, package installs, API calls)  
- Layered security (Security Groups + NACLs)  
- Explicit routing decisions  
- Room to expand into ALB → App → DB multi-tier patterns  

The design must emphasize **security**, **clarity**, and **cost awareness**.

---

# 2. Options Considered

## **Option A — Single Subnet (Flat Network)**

**Pros**
- Minimal components  
- Fast to deploy  

**Cons**
- No network segmentation  
- Public and private workloads mixed  
- No blast-radius control  
- Not realistic for production  

**Status:** ❌ Rejected

---

## **Option B — Public + Private Subnets Without NAT**

**Pros**
- True tier separation  
- Private subnet remains completely isolated  

**Cons**
- Private instances cannot:  
  - Install OS updates  
  - Pull packages  
  - Reach external APIs  
- Operationally restrictive  

**Status:** ❌ Rejected

---

## **Option C — Public + Private Subnets With NAT Gateway** *(Chosen)*

**Pros**
- Strong segmentation  
- Private tier remains non-public  
- NAT Gateway provides outbound-only internet  
- Scalable, production-aligned architecture  

**Cons**
- NAT Gateway introduces recurring cost  
- Requires additional routing and security configuration  

**Status:** ✅ Selected

---

# 3. Decision

Implement a **two-subnet VPC** with:

- **Public Subnet**  
  - Route → Internet Gateway (IGW)  
  - Handles ingress/egress for public resources  

- **Private Subnet**  
  - Route → NAT Gateway  
  - Internal workloads only  

**Supporting Components:**
- **Internet Gateway (IGW)**  
- **NAT Gateway**  
- **Security Groups (SGs)** — stateful instance-level filtering  
- **Network ACLs (NACLs)** — stateless subnet-level guardrails  
- **Separate Route Tables** for public and private tiers  

This structure mirrors the most common production VPC patterns.

---

# 4. Trade-offs

## **Security vs. Simplicity**
- Two subnets + NAT increases complexity  
- But provides strong isolation and reduces exposure  

## **Cost vs. Reliability**
- NAT Gateway is more expensive than NAT Instance  
- But fully managed: no patching, scaling, or failover work  

## **Flexibility vs. Restriction**
- Private subnet cannot accept inbound connections  
- Forces correct architectural behavior (bastion, ALB, SSM Session Manager)  

## **Operational Complexity**
- More components (two route tables, SGs, NACL rules)  
- But far clearer traffic flow and easier auditing  

---

# 5. Cost Impact

| Component | Approx. Monthly Cost | Notes |
|----------|----------------------|-------|
| **NAT Gateway** | ~$33 | Major recurring cost; egress billed separately |
| **EC2 t3.micro (public)** | ~$8 | Bastion or public-facing workload |
| **EC2 t3.micro (private)** | ~$8 | Internal app tier / DB placeholder |
| **IGW, Route Tables, NACLs** | Free | No direct cost |
| **Data Transfer (through NAT)** | Variable | Charged per GB |

**Key Insight:**  
Cloud networking is a technical and financial design exercise.  
Architecture choices directly influence the AWS bill.

---

# 6. Final Architecture Summary

- **VPC** (custom CIDR)  
- **Public Subnet**  
- **Private Subnet**  
- **Internet Gateway (IGW)**  
- **NAT Gateway**  
- **Public Route Table** → IGW  
- **Private Route Table** → NAT  
- **Security Groups** for instance-level permissions  
- **NACLs** for subnet guardrails  
- **EC2 Public** for ingress/bastion  
- **EC2 Private** for internal workloads  

This represents a minimal but production-minded VPC.

---

# 7. Lessons Learned

- **Nothing is implicit** in cloud networking — everything must be defined.  
- **SGs and NACLs serve different purposes:**  
  - SGs = stateful, precise  
  - NACLs = stateless, subnet-level guardrails  
- **Route tables are the real traffic controller** of the VPC.  
- **IGW + NAT = controlled public vs private boundaries.**  
- **Cost awareness** is part of the role — NAT, endpoints, and multi-AZ design all change the monthly bill.  
- **Architecture > clicking buttons.**  
  Documenting *why* decisions were made is the actual skill.