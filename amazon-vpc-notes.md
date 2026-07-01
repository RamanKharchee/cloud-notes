<div align="center">

<img src="vpc-logo.svg" width="110" alt="Amazon VPC logo" />

# Amazon VPC — Complete Notes

**Virtual Private Cloud** · Your isolated network on AWS

![AWS](https://img.shields.io/badge/AWS-VPC-8C4FFF?style=flat-square&logoColor=white)
![Type](https://img.shields.io/badge/Service-Networking-8C4FFF?style=flat-square)
![Scope](https://img.shields.io/badge/Scope-Regional-blue?style=flat-square)
![Isolation](https://img.shields.io/badge/Network-Isolated-lightgrey?style=flat-square)

*A reader-friendly, all-in-one reference for Amazon VPC.*

<!--PDF-BUTTON-START-->

[<img src="https://img.shields.io/badge/⬇%20Download%20as%20PDF-8C4FFF?style=for-the-badge&logo=adobeacrobatreader&logoColor=white" alt="Download as PDF" height="40">](https://github.com/RamanKharchee/cloud-notes/raw/main/amazon-vpc-notes.pdf)

<sub>Opens the rendered PDF — use your browser's download button to save it.</sub>

<!--PDF-BUTTON-END-->

</div>

---

## 📑 Table of Contents

1. [What is a VPC](#1--what-is-a-vpc)
2. [Core Components](#2--core-components)
3. [CIDR & IP Addressing](#3--cidr--ip-addressing)
4. [Subnets (Public vs Private)](#4--subnets-public-vs-private)
5. [Route Tables](#5--route-tables)
6. [Gateways (IGW, NAT, etc.)](#6--gateways-igw-nat-etc)
7. [Security Groups vs NACLs](#7--security-groups-vs-nacls)
8. [VPC Connectivity Options](#8--vpc-connectivity-options)
9. [VPC Endpoints & PrivateLink](#9--vpc-endpoints--privatelink)
10. [DNS in a VPC](#10--dns-in-a-vpc)
11. [VPC Flow Logs & Monitoring](#11--vpc-flow-logs--monitoring)
12. [Common CLI Commands](#12--common-cli-commands)
13. [Key Limits & Numbers](#13--key-limits--numbers)
14. [Design Best Practices](#14--design-best-practices)
15. [Quick Mental Model](#15--quick-mental-model)
16. [Common Interview Questions](#16--common-interview-questions)
17. [Detailed Interview Q&A and When to Use](#17--detailed-interview-qa-and-when-to-use)

---

## 1. 🌐 What is a VPC

Amazon VPC lets you provision a **logically isolated virtual network** within AWS where you launch resources (EC2, RDS, Lambda, etc.). You fully control the IP address range, subnets, route tables, gateways, and firewall rules — like running your own data-center network, but software-defined.

> 💡 **Mental model:** a VPC is your **private slice of the AWS cloud**. It's **regional** (spans all AZs in one region); subnets live in a single AZ.

**Why it matters:** isolation, segmentation (public vs private tiers), fine-grained traffic control, and secure connectivity to on-prem and other VPCs.

---

## 2. 🧩 Core Components

| Component | Role |
|---|---|
| **VPC** | The network container with a CIDR block; regional. |
| **Subnet** | A slice of the VPC CIDR, bound to one AZ. |
| **Route Table** | Rules deciding where subnet traffic goes. |
| **Internet Gateway (IGW)** | Allows internet access for public subnets. |
| **NAT Gateway** | Lets private subnets reach the internet outbound only. |
| **Security Group** | Stateful firewall at the instance/ENI level. |
| **Network ACL** | Stateless firewall at the subnet level. |
| **ENI** | Virtual network interface attached to resources. |
| **Elastic IP** | Static public IPv4 you own. |
| **VPC Endpoint** | Private access to AWS services without the internet. |
| **Peering / TGW / VPN / DX** | Ways to connect VPCs and on-prem networks. |

---

## 3. 🔢 CIDR & IP Addressing

- A VPC is defined by a **CIDR block**, e.g. `10.0.0.0/16` (65,536 addresses).
- Allowed VPC CIDR size: **/16 (max) to /28 (min)**. Use **private ranges** (RFC 1918): `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.
- Subnets carve the VPC CIDR into smaller blocks, e.g. `10.0.1.0/24` (256 addresses).
- **AWS reserves 5 IPs per subnet** (first 4 + last 1): network, VPC router, DNS, future use, broadcast. So a `/24` gives **251 usable** IPs.
- You can add **secondary CIDR blocks** to a VPC and enable **IPv6** (AWS assigns a `/56`, subnets get `/64`).

> ⚠️ Plan CIDRs so VPCs that may peer or connect to on-prem **don't overlap** — overlapping ranges can't be routed between.

---

## 4. 🏗️ Subnets (Public vs Private)

| | **Public Subnet** | **Private Subnet** |
|---|---|---|
| Route to IGW | ✅ Yes | ❌ No |
| Internet inbound | Possible (with public IP) | No |
| Internet outbound | Direct via IGW | Via **NAT Gateway** only |
| Typical use | Load balancers, bastions, NAT GW | App servers, databases |

- A subnet is **public** when its route table has a route `0.0.0.0/0 → IGW`.
- Spread subnets across **multiple AZs** for high availability.
- Common 3-tier layout: public (web/LB) · private-app · private-data, each replicated per AZ.

---

## 5. 🗺️ Route Tables

- Each subnet associates with **exactly one** route table; a route table can serve many subnets.
- The **main route table** is the default; create **custom** ones per tier.
- Routes map a **destination CIDR → target** (IGW, NAT GW, ENI, peering, TGW, endpoint, etc.).
- The **local route** (`VPC CIDR → local`) is always present and enables intra-VPC routing — it can't be removed.
- **Most specific match wins** (longest prefix).

```
Destination        Target
10.0.0.0/16        local            (intra-VPC, implicit)
0.0.0.0/0          igw-xxxx         (public subnet → internet)
0.0.0.0/0          nat-xxxx         (private subnet → NAT)
```

---

## 6. 🚪 Gateways (IGW, NAT, etc.)

| Gateway | Direction | Notes |
|---|---|---|
| **Internet Gateway (IGW)** | In + out | One per VPC; horizontally scaled, no bandwidth limit. Required for public internet. |
| **NAT Gateway** | Outbound only | Managed, per-AZ, needs an EIP. Lets private subnets pull updates/reach internet. Charged hourly + per GB. |
| **NAT Instance** | Outbound only | Legacy DIY (an EC2 doing NAT). Cheaper but you manage it. |
| **Egress-Only IGW** | Outbound only (IPv6) | The IPv6 equivalent of a NAT gateway. |
| **Virtual Private Gateway (VGW)** | VPN | AWS side of a Site-to-Site VPN. |
| **Transit Gateway (TGW)** | Hub | Connects many VPCs + on-prem at scale. |

> 💸 NAT Gateways cost per hour **and** per GB processed — a common surprise bill. Use **VPC Gateway Endpoints** for S3/DynamoDB to avoid NAT data charges.

---

## 7. 🛡️ Security Groups vs NACLs

A **Security Group** is a **virtual, stateful firewall** attached to a resource's network interface (ENI) inside the VPC. It uses **allow rules** that match a **protocol + port + source/destination** (an IP/CIDR or another security group). Being **stateful**, allowed inbound traffic gets its return path opened automatically. By default it **denies all inbound** and **allows all outbound**; a resource can carry several groups whose rules combine. It's the instance-level control, while a **Network ACL** is the stateless, subnet-level layer around it.

| | **Security Group** | **Network ACL** |
|---|---|---|
| Level | Instance / ENI | Subnet |
| State | **Stateful** (return traffic auto-allowed) | **Stateless** (allow both directions) |
| Rules | Allow only | Allow **and** Deny |
| Evaluation | All rules together | Numbered order, first match wins |
| Default | Deny inbound / allow outbound | Default allows all; custom denies all |

> ✅ Use **Security Groups** as your primary control (reference other SGs as sources for tier-to-tier rules). Use **NACLs** for coarse subnet-wide allow/deny (e.g. block a bad IP range).

---

## 8. 🔗 VPC Connectivity Options

| Option | Connects | Notes |
|---|---|---|
| **VPC Peering** | VPC ↔ VPC (1:1) | Non-transitive; simple; can be cross-region/cross-account. CIDRs must not overlap. |
| **Transit Gateway (TGW)** | Many VPCs + VPN/DX (hub-spoke) | Scales to thousands of VPCs; transitive routing. |
| **Site-to-Site VPN** | On-prem ↔ VPC over internet | Encrypted IPsec tunnels via VGW or TGW. |
| **Direct Connect (DX)** | On-prem ↔ AWS dedicated line | Private, consistent bandwidth/latency; not over internet. |
| **PrivateLink / Endpoints** | VPC ↔ AWS or partner service | Private, no IGW/NAT (see next section). |

> Peering is **not transitive**: if A↔B and B↔C peer, A can't reach C through B. Use a **Transit Gateway** for hub-and-spoke at scale.

---

## 9. 🔌 VPC Endpoints & PrivateLink

Reach AWS services **privately**, without an IGW, NAT, or public IPs.

| Endpoint type | How | Services |
|---|---|---|
| **Gateway Endpoint** | A route-table target | **S3 and DynamoDB only**. Free. |
| **Interface Endpoint (PrivateLink)** | An ENI with a private IP in your subnet | Most AWS services + partner/SaaS + your own services. Hourly + per-GB cost. |

> Benefits: traffic stays on the AWS network (more secure), avoids NAT data charges (Gateway Endpoint), and enables private SaaS connectivity (PrivateLink).

---

## 10. 🧭 DNS in a VPC

- AWS provides DNS at the **VPC base +2 address** (e.g. `10.0.0.2`) — the **Route 53 Resolver**.
- Two VPC attributes: `enableDnsSupport` (resolution on) and `enableDnsHostnames` (assign public DNS names).
- **Route 53 Private Hosted Zones** give internal domain names resolvable inside the VPC.
- **Resolver endpoints** (inbound/outbound) bridge DNS between on-prem and VPC.

---

## 11. 📊 VPC Flow Logs & Monitoring

- **Flow Logs** capture IP traffic metadata (src/dst, ports, bytes, ACCEPT/REJECT) at VPC, subnet, or ENI level.
- Destinations: **CloudWatch Logs**, **S3**, or **Kinesis Firehose**. Great for troubleshooting connectivity and security forensics.
- Flow Logs do **not** capture packet contents (use traffic mirroring for that).
- **VPC Reachability Analyzer** and **Network Access Analyzer** help validate/troubleshoot paths and exposure.

---

## 12. ⌨️ Common CLI Commands

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 create-subnet --vpc-id vpc-123 --cidr-block 10.0.1.0/24 --availability-zone us-east-1a
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-123 --internet-gateway-id igw-123
aws ec2 create-route-table --vpc-id vpc-123
aws ec2 create-route --route-table-id rtb-123 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-123
aws ec2 create-nat-gateway --subnet-id subnet-123 --allocation-id eipalloc-123
aws ec2 describe-vpcs
aws ec2 create-flow-logs --resource-type VPC --resource-ids vpc-123 \
  --traffic-type ALL --log-destination-type cloud-watch-logs --log-group-name vpc-flow
```

---

## 13. 📌 Key Limits & Numbers

| Item | Value |
|---|---|
| VPCs per region | 5 (default, adjustable) |
| VPC CIDR size | /16 (max) to /28 (min) |
| Secondary CIDRs per VPC | up to 5 (adjustable) |
| Subnets per VPC | 200 (default) |
| Reserved IPs per subnet | **5** (first 4 + last 1) |
| Route tables per VPC | 200 |
| Routes per route table | 50 (non-propagated) |
| Security groups per ENI | 5 (up to 16) |
| Rules per security group | 60 in + 60 out |
| Internet Gateways | 1 per VPC |

---

## 14. 🧱 Design Best Practices

- **Plan non-overlapping CIDRs** across all VPCs and on-prem from day one.
- **Multi-AZ subnets** for every tier → high availability.
- **Public/private separation**: only LBs, bastions, and NAT in public subnets; everything else private.
- **Least privilege** Security Groups; reference SGs, not raw IPs, for tier-to-tier traffic.
- Use **Gateway Endpoints** (S3/DynamoDB) and **Interface Endpoints** to cut NAT cost and stay private.
- Centralize multi-VPC connectivity with a **Transit Gateway**, not a mesh of peerings.
- Turn on **Flow Logs** for security and troubleshooting.

---

## 15. 🧠 Quick Mental Model

- **VPC = your private network in AWS**, defined by a CIDR, regional in scope.
- **Subnets** split it per AZ; **public** = route to IGW, **private** = route to NAT.
- **Route tables** decide where traffic goes; **most specific route wins**.
- **Security Groups** (stateful, instance) + **NACLs** (stateless, subnet) filter traffic.
- Connect out: **IGW** (public), **NAT** (private outbound), **VPN/DX** (on-prem), **Peering/TGW** (VPC-to-VPC).
- Reach AWS services privately with **Endpoints / PrivateLink**.
- **Don't overlap CIDRs**, spread across AZs, keep data tiers private, and watch **NAT data costs**.

---

## 16. ❓ Common Interview Questions

Rapid-fire questions interviewers ask about VPC:

- **Q: Public vs private subnet?** — A subnet is *public* when its route table has `0.0.0.0/0 → Internet Gateway`. Private subnets reach the internet only outbound via a **NAT Gateway**.
- **Q: Security Group vs NACL?** — SG = stateful, instance/ENI level, allow-only. NACL = stateless, subnet level, ordered allow + deny rules.
- **Q: How many IPs are usable in a /24 subnet?** — 251. AWS reserves 5 per subnet (network, router, DNS, future, broadcast).
- **Q: How do private instances get internet for updates?** — Through a NAT Gateway in a public subnet (outbound only); inbound from the internet stays blocked.
- **Q: VPC Peering vs Transit Gateway?** — Peering is 1:1 and **non-transitive**. Transit Gateway is a hub connecting many VPCs + on-prem with transitive routing (use at scale).
- **Q: Gateway vs Interface endpoint?** — Gateway endpoint = free, route-table target, **S3 & DynamoDB only**. Interface endpoint (PrivateLink) = ENI with a private IP, most services, hourly + per-GB cost.
- **Q: Why avoid overlapping CIDRs?** — Overlapping ranges can't be routed between peered/connected networks.
- **Q: How to reduce NAT cost?** — Use VPC Gateway Endpoints for S3/DynamoDB so that traffic skips the NAT (which charges per GB).

---

## 17. 🎯 Detailed Interview Q&A and When to Use

> Try each question first, then open its answer below. Every answer opens with a plain-English *In plain terms* line, then the technical detail and **when to use** guidance — aimed at readers newer to AWS/DevOps.

### Questions

**Basics**

- [Q1. What is a VPC, and how do you size the CIDR?](#q1-what-is-a-vpc-and-how-do-you-size-the-cidr)
- [Q2. Public vs private subnet — what actually makes a subnet public?](#q2-public-vs-private-subnet--what-actually-makes-a-subnet-public)
- [Q3. IGW vs NAT Gateway vs NAT instance?](#q3-igw-vs-nat-gateway-vs-nat-instance)
- [Q4. Security Group vs NACL?](#q4-security-group-vs-nacl)
- [Q5. Gateway vs Interface VPC endpoint?](#q5-gateway-vs-interface-vpc-endpoint)
- [Q6. How does route-table evaluation pick a path?](#q6-how-does-route-table-evaluation-pick-a-path)

**Intermediate**

- [Q7. NAT Gateway high availability and cost?](#q7-nat-gateway-high-availability-and-cost)
- [Q8. VPC Peering vs Transit Gateway vs PrivateLink?](#q8-vpc-peering-vs-transit-gateway-vs-privatelink)
- [Q9. DNS inside a VPC?](#q9-dns-inside-a-vpc)
- [Q10. VPC Flow Logs — what do they capture?](#q10-vpc-flow-logs--what-do-they-capture)
- [Q11. Reserved IPs and subnet sizing?](#q11-reserved-ips-and-subnet-sizing)
- [Q12. How does a private instance reach the internet vs AWS services?](#q12-how-does-a-private-instance-reach-the-internet-vs-aws-services)

**Advanced & Scenario**

- [Q13. Scenario — private EC2 can't reach S3 / Secrets Manager.](#q13-scenario--private-ec2-cant-reach-s3--secrets-manager)
- [Q14. Scenario — two VPCs with overlapping CIDRs must communicate.](#q14-scenario--two-vpcs-with-overlapping-cidrs-must-communicate)
- [Q15. Scenario — design the VPC for a 3-tier app.](#q15-scenario--design-the-vpc-for-a-3-tier-app)
- [Q16. Keep a database unreachable from the internet — how?](#q16-keep-a-database-unreachable-from-the-internet--how)
- [Q17. Why one NAT Gateway per AZ?](#q17-why-one-nat-gateway-per-az)
- [Q18. Centralized egress / inspection at scale?](#q18-centralized-egress--inspection-at-scale)

---

### Answers

#### Q1. What is a VPC, and how do you size the CIDR?

*In plain terms:* Your own private network inside AWS; you pick an address range (like 10.0.0.0/16) and carve it into subnets — size it big up front for room to grow.

A logically isolated virtual network in AWS where you control subnets, routing, and gateways. You pick a private CIDR block (e.g. `10.0.0.0/16`) and carve subnets from it.
**When to use a larger block:** `/16` for room to grow / many subnets across AZs; smaller `/24` per subnet. Plan non-overlapping ranges up front if you'll ever peer or connect on-prem.

[↩ Back to questions](#questions)

#### Q2. Public vs private subnet — what actually makes a subnet public?

*In plain terms:* A subnet is 'public' only if it has a route to the internet gateway; 'private' subnets don't — that's the whole difference.

A subnet is "public" only if its route table has a route to an **Internet Gateway**; private subnets don't.
**When to use:** Public → load balancers, NAT gateways, bastion. Private → app servers, databases, anything that shouldn't be internet-reachable.

[↩ Back to questions](#questions)

#### Q3. IGW vs NAT Gateway vs NAT instance?

*In plain terms:* Internet Gateway = two-way internet for public subnets; NAT Gateway = outbound-only internet for private subnets; NAT instance = a DIY version.

IGW = bidirectional internet for public subnets. NAT Gateway = managed, HA outbound-only internet for private subnets. NAT instance = a self-managed EC2 doing the same.
**When to use:** IGW → public subnets. NAT Gateway → private-subnet egress (default). NAT instance → tiny/cost-sensitive or when you need port-forwarding/custom firewall.

[↩ Back to questions](#questions)

#### Q4. Security Group vs NACL?

*In plain terms:* Security Group = smart firewall on the server that remembers connections; NACL = a plain firewall on the subnet that checks each direction.

SG = stateful, instance/ENI-level, allow-only. NACL = stateless, subnet-level, ordered allow + explicit deny.
**When to use:** SG → primary instance firewall (always). NACL → coarse subnet rules or an explicit deny (block a CIDR).

[↩ Back to questions](#questions)

#### Q5. Gateway vs Interface VPC endpoint?

*In plain terms:* Private doorways to AWS services without using the internet — Gateway (free, S3/DynamoDB only) or Interface (an ENI, most services, costs a bit).

Gateway endpoint = free, a route-table target, **S3 & DynamoDB only**. Interface endpoint (PrivateLink) = an ENI with a private IP for most other AWS/partner services, billed hourly + per GB.
**When to use:** Gateway → private S3/DynamoDB access (free, do it always). Interface → private access to other services (Secrets Manager, SQS, ECR, etc.) without a NAT.

[↩ Back to questions](#questions)

#### Q6. How does route-table evaluation pick a path?

*In plain terms:* Routing picks the most specific matching rule — local traffic stays inside, and a catch-all route sends the rest to the internet gateway or NAT.

Most-specific (longest-prefix) matching route wins; `local` covers in-VPC traffic, and a `0.0.0.0/0` default route sends the rest to IGW/NAT/endpoint as configured.

[↩ Back to questions](#questions)

#### Q7. NAT Gateway high availability and cost?

*In plain terms:* A NAT lives in one zone, so run one per zone for resilience and to avoid cross-zone data charges — a single NAT is fine only for dev.

A NAT Gateway lives in one AZ; for resilience you deploy **one per AZ** and point each private subnet's route at its own-AZ NAT.
**When to use one-per-AZ:** production — it avoids an AZ failure killing egress and avoids cross-AZ data-transfer charges. A single NAT is fine only for dev.

[↩ Back to questions](#questions)

#### Q8. VPC Peering vs Transit Gateway vs PrivateLink?

*In plain terms:* Connect networks — peering (just two, no pass-through), Transit Gateway (a hub for many), or PrivateLink (share one service, even with clashing address ranges).

Peering = 1:1, non-transitive. Transit Gateway = a hub for many VPCs + on-prem with transitive routing. PrivateLink = exposes a single service privately (not whole-network routing).
**When to use:** Peering → a couple of VPCs. Transit Gateway → many VPCs/accounts at scale. PrivateLink → share one service across VPCs/accounts, even with overlapping CIDRs.

[↩ Back to questions](#questions)

#### Q9. DNS inside a VPC?

*In plain terms:* Turn on the built-in DNS so names resolve inside the VPC; use the Route 53 Resolver to bridge names with your on-prem network.

`enableDnsSupport` + `enableDnsHostnames` enable the Amazon-provided resolver; the **Route 53 Resolver** (and inbound/outbound endpoints) handles hybrid DNS with on-prem.
**When to use Resolver endpoints:** hybrid setups where on-prem must resolve AWS private zones (or vice versa).

[↩ Back to questions](#questions)

#### Q10. VPC Flow Logs — what do they capture?

*In plain terms:* Flow Logs record who talked to whom (addresses, ports, allowed/denied) but not the actual data — great for debugging connectivity and security.

Metadata about IP traffic (src/dst, ports, protocol, bytes, ACCEPT/REJECT) to CloudWatch/S3 — **not packet payloads**.
**When to use:** debugging connectivity (is traffic being REJECTed by SG/NACL?), security forensics, and traffic auditing.

[↩ Back to questions](#questions)

#### Q11. Reserved IPs and subnet sizing?

*In plain terms:* AWS keeps 5 addresses in every subnet, so a /24 gives 251 usable — leave headroom because Lambda/EKS eat IPs fast, and spread across zones.

AWS reserves **5 IPs per subnet** (network, VPC router, DNS, future use, broadcast), so a `/24` yields 251 usable. Spread subnets across ≥2–3 AZs for HA and leave headroom (Lambda/EKS ENIs eat IPs fast).

[↩ Back to questions](#questions)

#### Q12. How does a private instance reach the internet vs AWS services?

*In plain terms:* To the internet: route out through a NAT. To AWS services: use VPC endpoints so the traffic stays on AWS's network and skips the paid NAT.

Internet (outbound): route `0.0.0.0/0` → NAT Gateway → IGW. AWS services: **VPC endpoints** (Gateway for S3/DynamoDB, Interface for the rest) keep traffic on the AWS backbone and off the NAT.

[↩ Back to questions](#questions)

#### Q13. Scenario — private EC2 can't reach S3 / Secrets Manager.

*In plain terms:* Figure out if it's network (timeout means a missing route/endpoint) or permission (403 means IAM/policy), then fix the right layer.

Distinguish **timeout vs 403**: timeout = network (no route — needs a Gateway Endpoint for S3 or NAT/Interface Endpoint for Secrets Manager, plus SG egress and the endpoint's private DNS); 403 = it reached the service but the IAM role / endpoint policy / bucket policy denied it. Check route table, endpoint type, DNS, SG, then permissions.

[↩ Back to questions](#questions)

#### Q14. Scenario — two VPCs with overlapping CIDRs must communicate.

*In plain terms:* You can't peer networks with overlapping address ranges (ambiguous routing) — re-number one, or expose just the service via PrivateLink.

You **can't peer overlapping ranges** (routing is ambiguous). Options: re-IP one VPC, or expose only the needed service via **PrivateLink** (which sidesteps routing entirely), or front it with a NAT/proxy doing address translation.

[↩ Back to questions](#questions)

#### Q15. Scenario — design the VPC for a 3-tier app.

*In plain terms:* Public subnets for the load balancer and NAT, private for the app, private for the DB; private servers only reach out, and use a free endpoint for S3.

Public subnets: ALB + NAT (per AZ). Private app subnets: EC2/containers. Private DB subnets: RDS in a DB subnet group, `PubliclyAccessible=false`. Routes: public→IGW, private→NAT; **S3/DynamoDB via Gateway Endpoints**. SGs chained ALB→app→DB; NACLs as a coarse backstop.

[↩ Back to questions](#questions)

#### Q16. Keep a database unreachable from the internet — how?

*In plain terms:* Keep the database in a private subnet with no internet route and a firewall that allows only the app — it simply has no public path.

Place RDS in **private subnets** with `PubliclyAccessible=false`, a DB SG that allows the port **only from the app SG** (not a CIDR), no IGW route in the DB subnets, and access secrets via Secrets Manager. The DB has no public path at all.

[↩ Back to questions](#questions)

#### Q17. Why one NAT Gateway per AZ?

*In plain terms:* One NAT per zone means a zone outage doesn't cut off the others, and traffic stays in-zone to avoid cross-zone data charges.

Resilience (an AZ outage doesn't sever egress for the surviving AZs) and cost (keeps each AZ's traffic local, avoiding cross-AZ data-transfer charges through a single NAT).

[↩ Back to questions](#questions)

#### Q18. Centralized egress / inspection at scale?

*In plain terms:* At scale, route all networks' outbound through a shared 'egress VPC' with one NAT and a firewall (via Transit Gateway) instead of a NAT everywhere.

Route many VPCs' egress through a **Transit Gateway** to a shared **egress/inspection VPC** (NAT + firewall/Network Firewall) instead of a NAT per app VPC.
**When to use:** many accounts/VPCs needing consistent egress controls, traffic inspection, and lower aggregate NAT cost.

[↩ Back to questions](#questions)

---

<div align="center">

*📝 Notes compiled as a quick reference for Amazon VPC.*

</div>
