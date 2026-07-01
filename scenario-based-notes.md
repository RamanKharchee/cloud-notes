<div align="center">

<img src="scenarios-logo.svg" width="110" alt="Scenario interview logo" />

# Cloud & DevOps — Scenario-Based Interview Questions

**Cross-service scenarios** · EC2 · S3 · RDS · Lambda · SQS/SNS · Docker · VPC · Security · RBAC

![Type](https://img.shields.io/badge/Interview-Scenario%20Based-E11D48?style=flat-square)
![Scope](https://img.shields.io/badge/Scope-Cross%20Service-527FFF?style=flat-square)
![Level](https://img.shields.io/badge/Level-Intermediate%20to%20Senior-9f1239?style=flat-square)

*Real-world, multi-service interview scenarios with model answers — designed to be solved by combining services, not reciting one.*

</div>

---

## 📑 Table of Contents

1. [How to Use These Scenarios](#-how-to-use-these-scenarios)
2. [Compute, Storage & Delivery](#1--compute-storage--delivery)
3. [Databases & Data Layer](#2--databases--data-layer)
4. [Serverless & Event-Driven](#3--serverless--event-driven)
5. [Containers & CI/CD](#4--containers--cicd)
6. [Networking & Connectivity (VPC)](#5--networking--connectivity-vpc)
7. [Security, IAM & RBAC](#6--security-iam--rbac)
8. [Cross-Service Incident War-Rooms](#7--cross-service-incident-war-rooms)
9. [Rapid-Fire Combo Prompts](#8--rapid-fire-combo-prompts)
10. [Answers](#-answers)

---

## 🧭 How to Use These Scenarios

> These are deliberately **multi-service**. A good answer names the services, draws the data/permission flow, states the trade-off, and calls out failure modes — it does not just define one service. Each scenario gives a **Setup** and a **Question** — try it yourself first, then follow **📖 Show model answer** to the worked answer (Strong answer, Services in play, follow-ups) in the [Answers](#-answers) section below. Reason out loud; an interviewer scores the path, not just the destination.

---

## 1. 🖥️ Compute, Storage & Delivery

### Scenario 1 — User uploads, processed and served globally

**Setup.** A web app on EC2 (behind an ALB, in an Auto Scaling Group) lets users upload profile images. You must store them durably, generate thumbnails, serve them worldwide with low latency, keep cost low, and never expose the bucket publicly.

**Question.** Design the upload → process → serve path end to end.

**[📖 Show model answer ↓](#answer-1--user-uploads-processed-and-served-globally)**

### Scenario 2 — EC2 fleet reads a large dataset from S3, privately

**Setup.** An analytics fleet in private subnets reads tens of TB from S3 all day. Finance flags a large NAT data-processing bill.

**Question.** How do you cut cost and keep the path private and fast?

**[📖 Show model answer ↓](#answer-2--ec2-fleet-reads-a-large-dataset-from-s3-privately)**

### Scenario 3 — Cut compute cost 50% without losing HA

**Setup.** A stateless web tier runs 20 On-Demand instances 24/7 behind an ALB. You're asked to cut cost ~50% while keeping high availability and tolerating an AZ loss.

**[📖 Show model answer ↓](#answer-3--cut-compute-cost-50-without-losing-ha)**


---

## 2. 🗄️ Databases & Data Layer

### Scenario 4 — Secure, highly-available 3-tier data layer

**Setup.** A classic 3-tier app: ALB → app on EC2 (private) → RDS PostgreSQL. Security wants no public DB, rotated credentials, and a tested recovery story.

**[📖 Show model answer ↓](#answer-4--secure-highly-available-3-tier-data-layer)**

### Scenario 5 — Reporting load is crushing the primary

**Setup.** Heavy analytical queries during business hours spike primary CPU and slow the transactional app. Connections also occasionally hit `too many connections`.

**[📖 Show model answer ↓](#answer-5--reporting-load-is-crushing-the-primary)**

### Scenario 6 — Migrate MySQL-on-EC2 to RDS, minimal downtime

**Setup.** A self-managed MySQL on EC2 must move to RDS with near-zero downtime, then be hardened.

**[📖 Show model answer ↓](#answer-6--migrate-mysql-on-ec2-to-rds-minimal-downtime)**


---

## 3. ⚡ Serverless & Event-Driven

### Scenario 7 — Reliable S3-triggered processing at spiky scale

**Setup.** Every uploaded file must be processed exactly once-ish, must survive a downstream outage, must fan out to two independent consumers (thumbnailer + virus scanner), and must absorb 50× traffic spikes.

**[📖 Show model answer ↓](#answer-7--reliable-s3-triggered-processing-at-spiky-scale)**

### Scenario 8 — Lambda is exhausting RDS connections

**Setup.** A Lambda that writes to RDS works fine normally, but under a traffic spike the DB hits max connections and the app 500s.

**[📖 Show model answer ↓](#answer-8--lambda-is-exhausting-rds-connections)**

### Scenario 9 — Order pipeline needs ordering and no duplicates

**Setup.** An order pipeline must process events **in order per customer** and must not double-charge on retries.

**[📖 Show model answer ↓](#answer-9--order-pipeline-needs-ordering-and-no-duplicates)**


---

## 4. 🐳 Containers & CI/CD

### Scenario 10 — Containerize and ship a monolith safely

**Setup.** Take a monolith to containers with a build pipeline, a registry, deploys, secrets, image scanning, and fast rollback.

**[📖 Show model answer ↓](#answer-10--containerize-and-ship-a-monolith-safely)**

### Scenario 11 — "Works in staging, breaks in prod"

**Setup.** The same image (`:latest`) runs in staging but crash-loops in prod.

**[📖 Show model answer ↓](#answer-11--works-in-staging-breaks-in-prod)**

### Scenario 12 — Secure CI/CD with no long-lived AWS keys

**Setup.** Your GitHub Actions pipeline deploys to AWS. Security bans long-lived access keys in CI.

**[📖 Show model answer ↓](#answer-12--secure-cicd-with-no-long-lived-aws-keys)**


---

## 5. 🌐 Networking & Connectivity (VPC)

### Scenario 13 — Design the VPC for a 3-tier app

**Setup.** Lay out a VPC for ALB (public) → app (private) → RDS (private), internet egress for patches, and no public DB.

**[📖 Show model answer ↓](#answer-13--design-the-vpc-for-a-3-tier-app)**

### Scenario 14 — Lambda must reach RDS, S3, and a third-party API

**Setup.** A VPC-attached Lambda needs the private RDS, plus S3, plus an external HTTPS API.

**[📖 Show model answer ↓](#answer-14--lambda-must-reach-rds-s3-and-a-third-party-api)**

### Scenario 15 — EC2 in a private subnet can't reach S3 / Secrets Manager

**Setup.** A freshly launched private EC2 times out calling S3 and Secrets Manager.

**[📖 Show model answer ↓](#answer-15--ec2-in-a-private-subnet-cant-reach-s3--secrets-manager)**


---

## 6. 🔐 Security, IAM & RBAC

### Scenario 16 — Least privilege across many teams and accounts

**Setup.** A fast-growing org has many teams in one (then several) AWS accounts. You must enforce least privilege, separate prod from dev, and make public S3 impossible.

**[📖 Show model answer ↓](#answer-16--least-privilege-across-many-teams-and-accounts)**

### Scenario 17 — RBAC so a compromised EC2 can't pivot

**Setup.** An EC2 app needs S3, RDS, and SQS access. Assume the instance gets popped — limit the blast radius.

**[📖 Show model answer ↓](#answer-17--rbac-so-a-compromised-ec2-cant-pivot)**

### Scenario 18 — Developer RBAC: read prod, write dev, break-glass on-call

**Setup.** Design access so devs have read-only prod, full dev, and on-call can elevate during incidents — auditable.

**[📖 Show model answer ↓](#answer-18--developer-rbac-read-prod-write-dev-break-glass-on-call)**

### Scenario 19 — Kill secret sprawl across EC2, Lambda, and containers

**Setup.** DB creds and API keys are scattered in env vars, AMIs, and task defs. Centralize, rotate, and tightly scope reads.

**[📖 Show model answer ↓](#answer-19--kill-secret-sprawl-across-ec2-lambda-and-containers)**


---

## 7. 🚨 Cross-Service Incident War-Rooms

### Scenario 20 — "Checkout is down"

**Setup.** Customers can't check out. Path: CloudFront → ALB → ECS service → RDS, with an async SQS → Lambda side-effect.

**[📖 Show model answer ↓](#answer-20--checkout-is-down)**

### Scenario 21 — S3 bill tripled and the app got slow

**Setup.** Storage volume barely changed, but the S3 bill tripled and latency rose.

**[📖 Show model answer ↓](#answer-21--s3-bill-tripled-and-the-app-got-slow)**

### Scenario 22 — Connection storm after an RDS failover

**Setup.** Right after a Multi-AZ failover, the app throws a wave of errors and the recovered DB gets hammered.

**[📖 Show model answer ↓](#answer-22--connection-storm-after-an-rds-failover)**

### Scenario 23 — Security alert: public object + leaked key in an image

**Setup.** A scanner finds a public S3 object and an AWS access key baked into a container image in ECR.

**[📖 Show model answer ↓](#answer-23--security-alert-public-object--leaked-key-in-an-image)**


---

## 8. 🎛️ Rapid-Fire Combo Prompts

> Short, open-ended prompts that each force two or more services together. Use them for quick drills — say the architecture out loud in 60–90 seconds.

- Design a **file-drop integration**: a partner drops CSVs in S3; you validate, load into RDS, and notify on failure. (S3 + Lambda + SQS + SNS + RDS)
- A **scheduled job** must run nightly, read from RDS, write a report to S3, and email it. (EventBridge + Lambda/ECS + RDS + S3 + SNS/SES)
- Make an **EC2 app reach RDS and S3 with zero static credentials**. (Instance role + IMDSv2 + Gateway Endpoint + Secrets Manager)
- **Decouple** a synchronous EC2→EC2 call that keeps failing under load. (SQS buffer + autoscaling on queue depth)
- **Fan out** one event to email, a queue, and an HTTP endpoint. (SNS → SES + SQS + HTTPS subscription)
- Serve a **private internal dashboard** with no internet exposure. (VPC + Interface Endpoints + private ALB + IAM)
- **Zero-downtime deploy** of a containerized service with instant rollback. (ECR digests + ECS blue/green + ALB)
- Stop a **runaway Lambda** from melting the database. (RDS Proxy + reserved concurrency + SQS)
- Enforce **"prod data never leaves the prod account."** (SCP + VPC endpoints + bucket policy + KMS key policy)
- Give a vendor **temporary, scoped, audited access** to one bucket. (cross-account role + external ID + CloudTrail)
- A **container needs a DB password** at runtime but it can't be in the image. (task role + Secrets Manager + KMS)
- **Cost-optimize** a 24/7 batch fleet that's idle 70% of the time. (Spot + ASG schedules + S3 for state + queue-driven scaling)

---

## ✅ Answers

> Model answers for the scenarios above — each links back to its scenario. Try the scenario first; then check yourself here.

---

### Answer 1 — User uploads, processed and served globally

*In plain terms:* Let the browser upload straight to S3 (skip your servers), have a small function auto-make the thumbnails, and serve everything through a CDN so it's fast worldwide while the bucket stays private.

**Strong answer:**
- Upload **directly to S3 via a pre-signed PUT URL** minted by the backend — bytes never transit EC2, so the fleet stays small.
- S3 `ObjectCreated` event → **SNS or EventBridge → Lambda** to generate thumbnails; write derivatives back to a `processed/` prefix. Use a DLQ for failures.
- Serve through **CloudFront with Origin Access Control (OAC)**; bucket keeps **Block Public Access on**, policy allows only the distribution.
- **Lifecycle**: transition originals to Standard-IA after 30 days; expire incomplete multipart uploads and old versions.
- Encrypt with **SSE-S3** (or SSE-KMS if audit is required), enforce HTTPS via an `aws:SecureTransport` deny.

**Services in play:** EC2/ALB/ASG, S3, Lambda, SNS/EventBridge, CloudFront, IAM, KMS.

**Interviewer pushes:**
- Why pre-signed URLs over uploading through EC2? (bandwidth, scaling, cost)
- How do you stop users overwriting each other's keys? (key namespacing per user + IAM/condition)
- What if thumbnailing must not lose an event during a spike? (SNS→SQS→Lambda with DLQ)

[↩ Back to Scenario 1](#scenario-1--user-uploads-processed-and-served-globally)

---

### Answer 2 — EC2 fleet reads a large dataset from S3, privately

*In plain terms:* Add a free "S3 doorway" inside your network (a Gateway Endpoint) so S3 traffic stops routing through the paid NAT — cheaper, and it never leaves AWS's private network.

**Strong answer:**
- Add an **S3 Gateway VPC Endpoint** — S3 traffic leaves via the endpoint, **not the NAT Gateway**, eliminating NAT data charges and keeping traffic on the AWS backbone.
- Attach an **endpoint policy** + instance **IAM role** scoped to the specific bucket/prefix ARNs (no wildcards).
- For hot objects, add a read-through cache layer or request-coalesce; consider **S3 byte-range** fetches for partial reads.

**Services in play:** EC2, S3, VPC (Gateway Endpoint), NAT Gateway, IAM.

**Interviewer pushes:**
- Gateway vs Interface endpoint — which for S3 and why? (S3/DynamoDB use Gateway; it's free)
- How does the route table change? (prefix-list route to the endpoint)

[↩ Back to Scenario 2](#scenario-2--ec2-fleet-reads-a-large-dataset-from-s3-privately)

---

### Answer 3 — Cut compute cost 50% without losing HA

*In plain terms:* Make the servers disposable (no data stored on them), run a cheap mix of Spot + a small always-on base across three zones, and move static files to S3/CDN so you need fewer servers.

**Strong answer:**
- Externalize all state (sessions in ElastiCache/DynamoDB, assets in S3) so instances are disposable.
- Run a **mixed-instances ASG**: a small **On-Demand/Savings-Plan baseline** for guaranteed capacity + **Spot** for the rest, across ≥3 AZs.
- Serve static assets from **S3 + CloudFront** so the EC2 tier shrinks.
- Set Spot allocation to `price-capacity-optimized` and capacity-rebalance on.

**Services in play:** EC2 (Spot + ASG), ALB, S3, CloudFront, Savings Plans.

**Interviewer pushes:**
- What breaks if a Spot reclaim hits 40% of the fleet at once? (capacity rebalance, On-Demand floor, connection draining)
- Why not 100% Spot? (no guaranteed floor during a capacity crunch)

[↩ Back to Scenario 3](#scenario-3--cut-compute-cost-50-without-losing-ha)

---

### Answer 4 — Secure, highly-available 3-tier data layer

*In plain terms:* Keep the database in a private area reachable only from the app, keep a standby copy that auto-takes-over if it dies, store the password in a vault that rotates it, and keep encrypted backups in another region.

**Strong answer:**
- RDS in **private subnets only**, `PubliclyAccessible=false`; the DB **security group allows 5432 only from the app SG** (SG-to-SG reference, not CIDR).
- **Multi-AZ** for automatic failover; app connects to the **endpoint DNS**, never an IP.
- Credentials in **Secrets Manager with rotation**; app fetches at runtime via its instance role — no passwords on disk.
- **Encryption at rest (KMS)** set at creation; automated backups (PITR) + periodic manual snapshots **copied cross-region** for DR.

**Services in play:** EC2, RDS (Multi-AZ), VPC/Subnets/SG, Secrets Manager, KMS.

**Interviewer pushes:**
- Why SG-to-SG instead of the app's CIDR? (survives IP churn, least privilege)
- The DB is unencrypted today — how do you encrypt it? (snapshot → copy with encryption → restore)

[↩ Back to Scenario 4](#scenario-4--secure-highly-available-3-tier-data-layer)

---

### Answer 5 — Reporting load is crushing the primary

*In plain terms:* Send the heavy reporting queries to read-only copies of the database, and put a connection pooler in front so you don't run out of connections — a bigger box won't fix that.

**Strong answer:**
- Add **read replicas** and route reporting/read-only traffic to them; keep read-after-write-critical reads on the primary.
- Put **RDS Proxy** in front to pool/multiplex connections (fixes connection exhaustion that a bigger instance wouldn't).
- Profile with **Performance Insights** + `pg_stat_statements`; add indexes before scaling vertically.

**Services in play:** RDS (read replicas), RDS Proxy, CloudWatch/Performance Insights.

**Interviewer pushes:**
- What's the risk of reading from a replica? (replication lag → stale reads)
- CPU is fine but connections are maxed — bigger instance or Proxy? (Proxy/pooling)

[↩ Back to Scenario 5](#scenario-5--reporting-load-is-crushing-the-primary)

---

### Answer 6 — Migrate MySQL-on-EC2 to RDS, minimal downtime

*In plain terms:* Use AWS's migration service to copy the data live while the old database keeps serving, then flip over during a tiny window, and tighten security afterwards.

**Strong answer:**
- Use **AWS DMS** with full-load + CDC so the source keeps serving while data replicates; cut over during a short window once lag is ~0.
- Harden after: private subnets, KMS encryption, Secrets Manager, IAM auth optional, automated backups, Multi-AZ.
- Validate row counts/checksums before flipping the connection string (ideally via a DNS/config switch for fast rollback).

**Services in play:** EC2, RDS, DMS, VPC, Secrets Manager, KMS.

**Interviewer pushes:**
- How do you roll back mid-cutover? (keep source writable until validated; DNS/config flip)
- Encryption can't be added in place on RDS — when do you set it? (at create/restore time)

[↩ Back to Scenario 6](#scenario-6--migrate-mysql-on-ec2-to-rds-minimal-downtime)

---

### Answer 7 — Reliable S3-triggered processing at spiky scale

*In plain terms:* Fan the upload event out to two queues (one per consumer) so each works independently, let the queues absorb spikes, and make retries safe so nothing is processed twice or lost.

**Strong answer:**
- **S3 → SNS topic → two SQS queues (fan-out)**, each with its own **Lambda** consumer. SQS buffers spikes and decouples the consumers.
- Add a **DLQ** per queue + redrive; make consumers **idempotent** (dedupe on object key + version/ETag) so retries are safe.
- Use **reserved/maximum concurrency** on Lambda to protect downstream systems from the spike.

**Services in play:** S3, SNS, SQS, Lambda, DLQ, CloudWatch.

**Interviewer pushes:**
- Why SNS→SQS instead of S3→Lambda directly? (fan-out, buffering, retries, replay)
- "Exactly once" — honest answer? (at-least-once + idempotency; FIFO if true ordering/dedup needed)

[↩ Back to Scenario 7](#scenario-7--reliable-s3-triggered-processing-at-spiky-scale)

---

### Answer 8 — Lambda is exhausting RDS connections

*In plain terms:* Put a connection-sharing proxy (RDS Proxy) between the functions and the database, and buffer the spike with a queue — the problem is too many connections, not a slow database.

**Strong answer:**
- Front RDS with **RDS Proxy** so thousands of Lambda invocations share a small pool and Proxy holds connections across failover.
- Smooth the spike with **SQS in front of the Lambda** + a sensible batch size / reserved concurrency, turning a burst into a drained queue.
- Right-size pool + connection reuse; avoid opening a new connection per invocation.

**Services in play:** Lambda, RDS, RDS Proxy, SQS.

**Interviewer pushes:**
- Why does Lambda specifically cause this? (each concurrent execution = its own connection)
- Bigger DB instance vs Proxy? (Proxy — the bottleneck is connections, not compute)

[↩ Back to Scenario 8](#scenario-8--lambda-is-exhausting-rds-connections)

---

### Answer 9 — Order pipeline needs ordering and no duplicates

*In plain terms:* Use FIFO queues keyed by customer so orders stay in order with no duplicates, and still add your own "already-done" check so a retry can never double-charge.

**Strong answer:**
- Use **SQS FIFO** with `MessageGroupId = customerId` for per-customer ordering and built-in **content-based dedup**; **SNS FIFO → SQS FIFO** if you also fan out.
- Still enforce **idempotency keys** at the handler (a dedup window isn't infinite); persist processed IDs.
- DLQ + replay for poison messages; alarm on DLQ depth.

**Services in play:** SQS FIFO, SNS FIFO, Lambda, DynamoDB (idempotency store).

**Interviewer pushes:**
- FIFO throughput limits vs Standard — when is FIFO worth it? (ordering/dedup needs vs throughput)
- Why still need app-level idempotency with FIFO dedup? (5-minute dedup window, cross-system retries)

[↩ Back to Scenario 9](#scenario-9--order-pipeline-needs-ordering-and-no-duplicates)

---

### Answer 10 — Containerize and ship a monolith safely

*In plain terms:* Build a small, non-root image in stages, scan it for vulnerabilities, push it tagged by exact version, run it with a role that fetches secrets at runtime, and roll back by pointing at the previous image.

**Strong answer:**
- **Multi-stage Dockerfile** (build stage with toolchain → minimal runtime image), non-root `USER`, pinned base digest.
- CI builds, **scans the image** (Trivy/ECR scan), pushes to **ECR** tagged by immutable digest/commit SHA — never deploy `:latest`.
- Deploy on **ECS/EKS (or EC2)** with the app reading secrets from **Secrets Manager/SSM** via a **task role** (no baked creds).
- **Blue/green or rolling** deploy with health checks; rollback = repoint to the previous digest.

**Services in play:** Docker, ECR, ECS/EKS/EC2, Secrets Manager/SSM, IAM (task role), CodePipeline/GitHub Actions.

**Interviewer pushes:**
- Why digests over `:latest`? (reproducibility, safe rollback)
- A secret was needed at build time — how, without leaking it into a layer? (BuildKit secrets, multi-stage)

[↩ Back to Scenario 10](#scenario-10--containerize-and-ship-a-monolith-safely)

---

### Answer 11 — "Works in staging, breaks in prod"

*In plain terms:* ":latest" can quietly be two different images — pin the exact image fingerprint so both environments run the same thing, then compare config, secrets, memory limits and CPU type.

**Strong answer:**
- `:latest` is mutable — staging and prod likely pulled **different images**. Pin **digests** so both run byte-identical.
- Diff the **config/secrets** (env, SSM/Secrets values), **resource limits** (prod OOMKilled → exit 137), and **arch** (`amd64` vs `arm64`).
- Check egress/dependency reachability differences (SG, NAT, endpoints).

**Services in play:** Docker, ECR, ECS/EC2, Secrets Manager/SSM, CloudWatch.

**Interviewer pushes:**
- First command you run? (`docker logs` / task events + exit code)
- Exit 137 — what does it tell you? (SIGKILL, usually OOM)

[↩ Back to Scenario 11](#scenario-11--works-in-staging-breaks-in-prod)

---

### Answer 12 — Secure CI/CD with no long-lived AWS keys

*In plain terms:* Let the pipeline log in to AWS with short-lived tokens instead of stored keys, and give that deploy identity only the handful of permissions it actually needs.

**Strong answer:**
- Use **GitHub OIDC → an IAM role** via `sts:AssumeRoleWithWebIdentity`; the workflow gets short-lived credentials, scoped by a trust policy on the repo/branch.
- The deploy role is **least-privilege** (only ECR push + ECS update + the specific resources), with a **permission boundary**.
- Sign/scan images; record provenance; require approvals for prod.

**Services in play:** GitHub Actions (OIDC), IAM (role/trust/boundary), STS, ECR, ECS.

**Interviewer pushes:**
- How does the trust policy stop another repo assuming the role? (`sub` condition on repo/branch)
- Why a permission boundary on top of the policy? (caps blast radius even if the policy drifts)

[↩ Back to Scenario 12](#scenario-12--secure-cicd-with-no-long-lived-aws-keys)

---

### Answer 13 — Design the VPC for a 3-tier app

*In plain terms:* Put the load balancer and NAT in a public area, the app and database in private areas, set routes so private servers only reach out (never in), and use a free endpoint for S3.

**Strong answer:**
- **Public subnets**: ALB + NAT Gateway (one per AZ for HA). **Private app subnets**: EC2/containers. **Private DB subnets**: RDS, in a DB subnet group.
- Route tables: public → IGW; private → NAT for egress. **S3/DynamoDB via Gateway Endpoints**; other AWS APIs via **Interface Endpoints** to avoid NAT.
- SGs chained (ALB SG → app SG → DB SG); NACLs as a coarse subnet backstop.

**Services in play:** VPC, Subnets, IGW, NAT GW, Route Tables, SG/NACL, VPC Endpoints, ALB, EC2, RDS.

**Interviewer pushes:**
- Why NAT per AZ? (AZ-failure isolation; cross-AZ data charges)
- How does the app reach S3 without NAT? (Gateway Endpoint)

[↩ Back to Scenario 13](#scenario-13--design-the-vpc-for-a-3-tier-app)

---

### Answer 14 — Lambda must reach RDS, S3, and a third-party API

*In plain terms:* Place the function in the private network so it can reach the database, use a free endpoint for S3, and add a NAT so it can also call the outside API.

**Strong answer:**
- Put the Lambda in **private subnets** with an ENI; reach RDS over the private network (via **RDS Proxy** to manage connections).
- **S3 via a Gateway Endpoint** (no NAT). The **external API needs egress → route to a NAT Gateway** in a public subnet.
- Right-size subnet IP space for Lambda ENIs; least-privilege execution role.

**Services in play:** Lambda (VPC), RDS/RDS Proxy, S3 Gateway Endpoint, NAT GW, IGW, IAM.

**Interviewer pushes:**
- Why did the external API call hang before? (VPC Lambda has no route to internet without NAT)
- Why not also reach S3 via NAT? (cost + Gateway Endpoint is free and private)

[↩ Back to Scenario 14](#scenario-14--lambda-must-reach-rds-s3-and-a-third-party-api)

---

### Answer 15 — EC2 in a private subnet can't reach S3 / Secrets Manager

*In plain terms:* First decide if it's a network problem (a timeout usually means a missing route/endpoint) or a permission problem (a 403 means IAM/policy) — then fix the right one.

**Strong answer (triage):**
- Is there a route to egress at all? (NAT for Secrets Manager unless using an **Interface Endpoint**; S3 needs **Gateway Endpoint** or NAT.)
- Check **endpoint policy / bucket policy / IAM role** — a 403 vs timeout distinguishes auth from network.
- Verify **DNS resolution** (private DNS enabled on the interface endpoint), SG egress, and NACLs.

**Services in play:** EC2, VPC Endpoints (Gateway + Interface), NAT, IAM, Route 53 private DNS.

**Interviewer pushes:**
- Timeout vs 403 — what does each point to? (network vs permissions)
- Which endpoint type for Secrets Manager? (Interface; S3 is Gateway)

[↩ Back to Scenario 15](#scenario-15--ec2-in-a-private-subnet-cant-reach-s3--secrets-manager)

---

### Answer 16 — Least privilege across many teams and accounts

*In plain terms:* Split teams into separate AWS accounts with company-wide guardrails nobody can override, hand out access through single sign-on, and cap what each team's roles are allowed to do.

**Strong answer:**
- **Multi-account via AWS Organizations** (prod/dev/security accounts); **SCPs** as guardrails — e.g. deny disabling Block Public Access, deny leaving the org, restrict regions.
- Humans get access via **IAM Identity Center (SSO) permission sets**, not IAM users; apps use **roles**.
- **Permission boundaries** cap what team-created roles can do; **tag-based (ABAC)** access scopes resources by `team`/`env` tags.

**Services in play:** Organizations, SCPs, IAM Identity Center, IAM roles, permission boundaries, ABAC tags, S3 BPA.

**Interviewer pushes:**
- SCP vs IAM policy — which actually stops a public bucket org-wide? (SCP — a guardrail no account admin can override)
- How does ABAC reduce policy sprawl? (one policy keyed on tags instead of N per-resource policies)

[↩ Back to Scenario 16](#scenario-16--least-privilege-across-many-teams-and-accounts)

---

### Answer 17 — RBAC so a compromised EC2 can't pivot

*In plain terms:* Give the server an identity with access to only the exact things it needs (no "allow everything"), and lock down its metadata service, so a hacked box can't reach anything else.

**Strong answer:**
- The **instance role is least-privilege**: scoped to specific bucket/prefix, queue, and DB-auth ARNs — **no wildcards**.
- Enforce **IMDSv2** (`HttpTokens=required`) to blunt SSRF credential theft.
- Prefer **temporary creds via the role** (never static keys); separate roles per workload; condition policies on VPC/source where possible.
- Detective layer: CloudTrail + GuardDuty to catch anomalous use of the role.

**Services in play:** EC2 (instance role, IMDSv2), S3, RDS (IAM auth), SQS, CloudTrail, GuardDuty.

**Interviewer pushes:**
- Why IMDSv2 specifically? (defeats common SSRF → metadata cred theft)
- A wildcard `s3:*` on `*` — what's the real-world damage? (full-account data exfil/destruction)

[↩ Back to Scenario 17](#scenario-17--rbac-so-a-compromised-ec2-cant-pivot)

---

### Answer 18 — Developer RBAC: read prod, write dev, break-glass on-call

*In plain terms:* Use single sign-on roles for read-only prod and full dev, plus a special "break-glass" emergency role that needs MFA, expires quickly, and pings everyone when someone uses it.

**Strong answer:**
- **Identity Center permission sets**: `Dev-ReadProd` (read-only prod), `Dev-Dev` (full dev), assigned per group.
- A **break-glass role** with elevated prod access, **MFA-required**, time-boxed via session policies, and **alerting on assumption** (EventBridge → SNS/Slack).
- Everything logged in **CloudTrail**; periodic access reviews; least-privilege by default.

**Services in play:** IAM Identity Center, IAM roles/session policies, MFA, CloudTrail, EventBridge, SNS.

**Interviewer pushes:**
- How do you make break-glass safe, not a backdoor? (MFA, time-box, alert, audit, review)
- Read-only that still leaks data? (e.g. `s3:GetObject` everywhere — scope it)

[↩ Back to Scenario 18](#scenario-18--developer-rbac-read-prod-write-dev-break-glass-on-call)

---

### Answer 19 — Kill secret sprawl across EC2, Lambda, and containers

*In plain terms:* Move every password and key into one central vault that rotates them, and have apps fetch them at runtime with a scoped identity — never bake secrets into images or environment variables.

**Strong answer:**
- Move everything to **Secrets Manager** (or SSM Parameter Store for non-rotating config); enable **rotation** for DB creds.
- Workloads read at runtime via **role + resource policy + KMS grant** — never bake secrets into images/env.
- Scope each secret's resource policy to the specific consuming role; audit access via CloudTrail.

**Services in play:** Secrets Manager, SSM Parameter Store, KMS, IAM roles, Lambda/EC2/ECS, CloudTrail.

**Interviewer pushes:**
- Why are env-var secrets risky? (leak via `inspect`, logs, crash dumps, image layers)
- Secrets Manager vs Parameter Store — when each? (rotation/secret vs cheap config at scale)

[↩ Back to Scenario 19](#scenario-19--kill-secret-sprawl-across-ec2-lambda-and-containers)

---

### Answer 20 — "Checkout is down"

*In plain terms:* Walk the request path — load balancer, containers, database, queues, functions — read the error type to spot which layer broke, stop the bleeding fast (scale or roll back), then find the real cause.

**Strong answer (triage):**
- Localize the layer: ALB target health + 5xx, ECS task health/restarts, RDS connections/CPU, SQS depth + DLQ, Lambda errors/throttles.
- Read the **error shape**: 503 from ALB = no healthy targets; DB connection errors = RDS/Proxy; growing SQS + DLQ = consumer failing.
- Mitigate fast (scale out, roll back the last deploy, drain), then root-cause from CloudWatch/X-Ray traces.

**Services in play:** CloudFront, ALB, ECS, RDS, SQS, Lambda, CloudWatch, X-Ray.

**Interviewer pushes:**
- First three dashboards you open? (ALB 5xx/targets, RDS connections, SQS/DLQ)
- How do you tell app bug from infra? (deploy correlation, error class, blast radius)

[↩ Back to Scenario 20](#scenario-20--checkout-is-down)

---

### Answer 21 — S3 bill tripled and the app got slow

*In plain terms:* It's almost always requests or data transfer, not storage — hunt for a hot loop hammering S3, add caching, and clean up old file versions piling up.

**Strong answer:**
- It's usually **requests/transfer, not storage**: a hot **Lambda/EC2 loop hammering GET** with no cache, **KMS calls** per object, cross-region/egress transfer, or noncurrent-version pileup.
- Use **S3 Storage Lens + Cost Explorer + CloudWatch request metrics** to find the offending prefix/principal.
- Fix: add caching (CloudFront/in-app), **S3 bucket keys** to cut KMS calls, lifecycle to expire versions, keep compute in-region.

**Services in play:** S3, Lambda/EC2, KMS, CloudFront, CloudWatch, Cost Explorer, Storage Lens.

**Interviewer pushes:**
- Why would latency and cost rise together? (same hot read path; throttling/`503 Slow Down`)
- 503 Slow Down — fix? (spread keys across prefixes + backoff)

[↩ Back to Scenario 21](#scenario-21--s3-bill-tripled-and-the-app-got-slow)

---

### Answer 22 — Connection storm after an RDS failover

*In plain terms:* The app remembered the old database address and everything reconnected at once — connect by DNS name, retry with a randomised backoff, and put a proxy in front to absorb the rush.

**Strong answer:**
- The app cached the **old endpoint IP** and/or every instance reconnected at once (connection storm). Connect by **endpoint DNS** with short TTL; add **jittered retry/backoff**.
- Put **RDS Proxy** in front so it holds client connections through failover and throttles reconnects.
- Bound the connection pool; don't let every worker open simultaneously.

**Services in play:** RDS (Multi-AZ), RDS Proxy, app connection pool, Route 53 DNS TTL.

**Interviewer pushes:**
- Why did pinning the IP break it? (failover moves the primary; DNS is the contract)
- How does Proxy specifically smooth failover? (holds connections, multiplexes, throttles reconnect)

[↩ Back to Scenario 22](#scenario-22--connection-storm-after-an-rds-failover)

---

### Answer 23 — Security alert: public object + leaked key in an image

*In plain terms:* Rotate the leaked key immediately, make the object private again, rebuild the image without the secret baked in, then add guardrails so this class of leak can't happen again.

**Strong answer (contain → eradicate → prevent):**
- **Contain:** revoke/rotate the leaked key immediately; flip the object/bucket private (re-enable Block Public Access); check CloudTrail for misuse.
- **Eradicate:** rebuild the image without the secret (BuildKit secrets/runtime fetch), purge the bad image, scan history.
- **Prevent:** **SCP** to forbid public buckets org-wide, **ECR scan-on-push + secret scanning in CI**, move secrets to Secrets Manager, least-privilege keys (or none — use roles).

**Services in play:** S3 (BPA), IAM/STS, ECR (scan), Secrets Manager, CloudTrail, Organizations/SCP, GuardDuty.

**Interviewer pushes:**
- Order of operations under active compromise? (rotate first, then audit blast radius)
- How do you make this class of leak structurally impossible? (no static keys + CI secret scanning + SCP guardrails)

[↩ Back to Scenario 23](#scenario-23--security-alert-public-object--leaked-key-in-an-image)
---

<div align="center">

*📝 Scenario-based interview prep — combine services, reason out loud.*

</div>
