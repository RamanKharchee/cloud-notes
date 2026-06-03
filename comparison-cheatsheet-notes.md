<div align="center">

<img src="cheatsheet-logo.svg" width="110" alt="Comparison Cheat-Sheet logo" />

# Comparison Cheat-Sheet — "X vs Y"

**Rapid revision** · Every easy-to-confuse distinction in one place

![Type](https://img.shields.io/badge/Type-Cheat--Sheet-475569?style=flat-square)
![Use](https://img.shields.io/badge/Use-Last--minute%20revision-475569?style=flat-square)
![Format](https://img.shields.io/badge/Format-Compare%20tables-blue?style=flat-square)
![Scope](https://img.shields.io/badge/Covers-All%20notes-lightgrey?style=flat-square)

*Every "this vs that" interviewers love to test — scan this the night before.*

<!--PDF-BUTTON-START-->

[<img src="https://img.shields.io/badge/⬇%20Download%20as%20PDF-475569?style=for-the-badge&logo=adobeacrobatreader&logoColor=white" alt="Download as PDF" height="40">](https://github.com/RamanKharchee/cloud-notes/raw/main/comparison-cheatsheet-notes.pdf)

<sub>Opens the rendered PDF — use your browser's download button to save it.</sub>

<!--PDF-BUTTON-END-->

</div>

---

## 📑 Table of Contents

1. [Storage](#1--storage)
2. [Compute & Containers](#2--compute--containers)
3. [Databases](#3--databases)
4. [Networking](#4--networking)
5. [DNS & CDN](#5--dns--cdn)
6. [Messaging & Events](#6--messaging--events)
7. [Security & Identity](#7--security--identity)
8. [Monitoring & Audit](#8--monitoring--audit)
9. [Git & GitHub](#9--git--github)
10. [Memory Hooks](#10--memory-hooks)
11. [Quick Decision Prompts](#11--quick-decision-prompts)

---

## 1. 💾 Storage

**S3 vs EBS vs EFS**

| | **S3** | **EBS** | **EFS** |
|---|---|---|---|
| Type | Object | Block | File (NFS) |
| Access | HTTP API, anywhere | One EC2 (one AZ) | Many EC2 across AZs |
| Scale | Unlimited | Up to 64 TiB/volume | Auto-scaling |
| Use | Backups, data lake, static sites | Boot/data disk for an instance | Shared filesystem |

**S3 storage classes (when to use)**

| Class | Use |
|---|---|
| Standard | Frequent access |
| Standard-IA / One Zone-IA | Infrequent, instant access (One Zone = 1 AZ, cheaper) |
| Glacier Instant / Flexible / Deep Archive | Archive: ms / minutes-hours / 12-48h retrieval |
| Intelligent-Tiering | Unknown/changing access patterns |

---

## 2. 🖥️ Compute & Containers

**EC2 pricing models**

| Model | Commitment | Discount | Use |
|---|---|---|---|
| On-Demand | None | — | Spiky / short-term |
| Reserved / Savings Plans | 1–3 yr | up to ~72% | Steady-state |
| Spot | None (interruptible) | up to ~90% | Fault-tolerant, batch |
| Dedicated Host | Per host | — | Licensing / compliance |

**Container vs VM**

| | Container | VM |
|---|---|---|
| Isolates | A process (shares host kernel) | A full OS (own kernel) |
| Size / start | MBs / ms | GBs / minutes |
| Isolation | OS-level (namespaces+cgroups) | Hardware-level (stronger) |

**Docker vs Kubernetes** — Docker builds/runs containers on **one host**; Kubernetes **orchestrates** many containers across **many hosts** (scheduling, scaling, self-healing).

**Lambda vs EC2** — Lambda = serverless, event-driven, ≤15 min, pay per ms, auto-scale. EC2 = full VM you manage, always-on, any runtime/length.

---

## 3. 🗃️ Databases

**DynamoDB vs RDS**

| | DynamoDB | RDS / Aurora |
|---|---|---|
| Model | NoSQL key-value/document | Relational (SQL) |
| Schema | Flexible | Fixed |
| Scale | Unlimited, automatic | Vertical + read replicas |
| Queries | By key/index (no joins) | Rich SQL, joins |
| Use | High scale, known patterns | Complex queries, transactions |

**RDS Multi-AZ vs Read Replica**

| | Multi-AZ | Read Replica |
|---|---|---|
| Purpose | High availability | Scale reads |
| Replication | Synchronous | Asynchronous |
| Readable? | ❌ No | ✅ Yes |
| Failover | Automatic, same endpoint | Manual promote |

**DynamoDB GSI vs LSI** — GSI = different partition key, added anytime, own capacity, eventual reads. LSI = same partition key + alt sort key, created at table creation only, strong reads.

---

## 4. 🌐 Networking

**Security Group vs NACL**

| | Security Group | NACL |
|---|---|---|
| Level | Instance / ENI | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow + Deny (ordered) |
| Default | Deny in / allow out | Default allow all |

**ALB vs NLB vs GWLB**

| | ALB | NLB | GWLB |
|---|---|---|---|
| Layer | L7 (HTTP) | L4 (TCP/UDP) | L3 |
| Routes on | Host/path/header | IP + port | IP packets |
| Use | Web apps, microservices | Ultra-fast, static IP | Virtual appliances |

**VPC Peering vs Transit Gateway** — Peering = 1:1, non-transitive. TGW = hub-and-spoke, transitive, scales to many VPCs + on-prem.

**Gateway vs Interface endpoint** — Gateway = free, route-table target, **S3 & DynamoDB only**. Interface (PrivateLink) = ENI + private IP, most services, hourly + per-GB.

**Public vs Private subnet** — Public has a route `0.0.0.0/0 → IGW`; private reaches internet only outbound via **NAT**.

---

## 5. 🛰️ DNS & CDN

**Route 53 Alias vs CNAME**

| | Alias | CNAME |
|---|---|---|
| Points to | AWS resources | Any DNS name |
| Zone apex? | ✅ Yes | ❌ No |
| Cost | Free (to AWS targets) | Charged |

**CloudFront vs S3 vs Global Accelerator**

| | CloudFront | S3 | Global Accelerator |
|---|---|---|---|
| Role | CDN (cache at edge) | Object store | Route TCP/UDP to nearest endpoint |
| Caches? | ✅ Yes | ❌ No | ❌ No |
| Layer | HTTP(S) | Storage | Network (static anycast IPs) |

**CloudFront Functions vs Lambda@Edge** — Functions = lightweight JS, sub-ms, viewer-side (header/redirect). Lambda@Edge = full runtime, origin-side too, heavier logic.

---

## 6. 📨 Messaging & Events

**SQS vs SNS vs EventBridge vs Kinesis**

| | SQS | SNS | EventBridge | Kinesis |
|---|---|---|---|---|
| Model | Queue (pull) | Pub/sub (push) | Event bus (routing) | Stream |
| Consumers | 1 per message | Many subscribers | Rule targets | Many, replayable |
| Use | Decouple + buffer | Fan-out | Route by content/source | Real-time analytics, replay |

**SQS Standard vs FIFO** — Standard = high throughput, at-least-once, best-effort order. FIFO = strict order + exactly-once, lower throughput (`.fifo`).

**SQS vs Kinesis** — SQS deletes a message once consumed (work queue). Kinesis is an ordered, **replayable** stream multiple consumers can re-read.

---

## 7. 🔐 Security & Identity

**IAM User vs Role**

| | User | Role |
|---|---|---|
| Credentials | Long-lived (keys/password) | Temporary (STS) |
| For | A person | Workloads, cross-account, federation |
| Preferred? | Avoid for apps | ✅ Yes |

**SCP vs Permission Boundary** — both only **cap** (never grant). SCP = org/OU/account-wide guardrail. Boundary = max for one user/role (safe delegation).

**Identity-based vs Resource-based policy** — identity-based attaches to a user/role; resource-based attaches to a resource and names a **Principal** (enables cross-account).

**IAM evaluation:** default deny → explicit **Deny wins** → must be allowed by SCP ∩ boundary ∩ identity/resource policy.

---

## 8. 📊 Monitoring & Audit

**CloudWatch vs CloudTrail**

| | CloudWatch | CloudTrail |
|---|---|---|
| Answers | How is it **performing**? | **Who did what**, when? |
| Data | Metrics, logs, alarms | API call audit log |
| Use | Monitoring, alerting | Auditing, compliance, forensics |

**EC2 status checks:** System check = AWS infra (fix: stop/start). Instance check = the OS/instance (fix: reboot/fix OS).

---

## 9. 🔀 Git & GitHub

**Merge vs Rebase**

| | Merge | Rebase |
|---|---|---|
| History | Preserved (branchy) | Rewritten (linear) |
| Merge commit | Yes | No |
| Safe on shared branches? | ✅ Yes | ❌ No |

**git fetch vs pull** — fetch downloads without merging; pull = fetch + merge (or `--rebase`).

**Fork vs Clone vs Branch** — fork = your server-side copy; clone = a local copy; branch = a line of work in a repo.

---

## 10. 🧠 Memory Hooks

One-liners worth memorizing verbatim:

- **CloudWatch = performance/health · CloudTrail = audit/who-did-what.**
- **Rebase private history, merge public history** ("rebase to clean up, merge to combine").
- **Multi-AZ = availability (sync, not readable) · Read Replica = scaling (async, readable).**
- **Alias = AWS targets + works at apex + free · CNAME = any name, not at apex.**
- **SG = stateful/instance/allow-only · NACL = stateless/subnet/allow+deny.**
- **SQS = queue (one consumer) · SNS = pub/sub (many subscribers) · fan-out = SNS → many SQS.**
- **Explicit Deny always wins** in IAM; boundaries & SCPs **cap, never grant**.
- **CloudFront caches; Global Accelerator routes (no cache).**
- **Container shares the host kernel; a VM ships its own.**
- **Spot = cheapest + interruptible (2-min notice); Reserved/Savings = steady; On-Demand = flexible.**

---

## 11. 🎯 Quick Decision Prompts

"Which service?" gut-checks:

- **Need a static IP on a load balancer?** → NLB (or Global Accelerator).
- **Route by URL path / host?** → ALB.
- **Decouple two services + buffer spikes?** → SQS.
- **Notify many subscribers at once?** → SNS (fan-out to SQS).
- **Key-value at massive scale, single-digit ms?** → DynamoDB.
- **Complex joins / transactions / reporting?** → RDS/Aurora.
- **Cache content near users / front S3 with HTTPS?** → CloudFront.
- **Run code on an event, no servers, ≤15 min?** → Lambda.
- **Give an EC2 app AWS permissions?** → IAM role (never hard-coded keys).
- **Private access to S3 from a VPC?** → Gateway VPC endpoint (free).
- **Highly available web tier?** → ALB + Auto Scaling across ≥2 AZs.
- **Audit who deleted a resource?** → CloudTrail. **Why is CPU high?** → CloudWatch.

---

<div align="center">

*📝 Notes compiled as a quick-revision comparison cheat-sheet.*

</div>
