<div align="center">

<img src="https://cdn.simpleicons.org/amazonrds/527FFF" width="110" alt="Amazon RDS logo" />

# Amazon RDS — Complete Notes

**Relational Database Service** · Managed SQL databases on AWS

![AWS](https://img.shields.io/badge/AWS-RDS-527FFF?style=flat-square&logo=amazonrds&logoColor=white)
![Type](https://img.shields.io/badge/Service-Database-527FFF?style=flat-square)
![Model](https://img.shields.io/badge/Model-Managed-blue?style=flat-square)
![Engines](https://img.shields.io/badge/Engines-6-lightgrey?style=flat-square)

*A reader-friendly, all-in-one reference for Amazon RDS.*

<!--PDF-BUTTON-START-->

[<img src="https://img.shields.io/badge/⬇%20Download%20as%20PDF-527FFF?style=for-the-badge&logo=adobeacrobatreader&logoColor=white" alt="Download as PDF" height="40">](https://github.com/RamanKharchee/cloud-notes/raw/main/amazon-rds-notes.pdf)

<sub>Opens the rendered PDF — use your browser's download button to save it.</sub>

<!--PDF-BUTTON-END-->

</div>

---

## 📑 Table of Contents

1. [What is RDS](#1--what-is-rds)
2. [Supported Engines](#2--supported-engines)
3. [Core Concepts](#3--core-concepts)
4. [High Availability (Multi-AZ)](#4--high-availability-multi-az)
5. [Read Replicas (Scaling Reads)](#5--read-replicas-scaling-reads)
6. [Storage Types](#6--storage-types)
7. [Backups, Snapshots & PITR](#7--backups-snapshots--pitr)
8. [Security](#8--security)
9. [Amazon Aurora](#9--amazon-aurora)
10. [Maintenance & Scaling](#10--maintenance--scaling)
11. [Monitoring](#11--monitoring)
12. [Pricing](#12--pricing)
13. [Common CLI Commands](#13--common-cli-commands)
14. [RDS vs Other AWS Databases](#14--rds-vs-other-aws-databases)
15. [Quick Mental Model](#15--quick-mental-model)
16. [Common Interview Questions](#16--common-interview-questions)
17. [Detailed Interview Q&A and When to Use](#17--detailed-interview-qa-and-when-to-use)

---

## 1. 🗃️ What is RDS

Amazon RDS is a **managed relational database** service. AWS handles the heavy operational work — provisioning, OS and DB patching, backups, replication, failover, and scaling — so you focus on schema and queries. You still control the database engine, parameters, and data.

> 💡 **Mental model:** RDS is "**managed SQL**." AWS owns the host, OS, and DB engine maintenance; **you** own the data, schema, queries, and tuning. (You do **not** get OS/SSH access.)

**Common uses:** transactional apps (OLTP), web/mobile backends, e-commerce, SaaS, ERP, anything needing a managed SQL database with HA and backups.

---

## 2. ⚙️ Supported Engines

| Engine | Notes |
|---|---|
| **Amazon Aurora** | AWS-built, MySQL- & PostgreSQL-compatible; cloud-native performance & scaling. |
| **MySQL** | Popular open-source. |
| **PostgreSQL** | Feature-rich open-source. |
| **MariaDB** | MySQL fork. |
| **Oracle** | Commercial; BYOL or license-included. |
| **SQL Server** | Microsoft; multiple editions. |

> Aurora is technically part of the RDS family but is a distinct, re-architected engine — see [section 9](#9--amazon-aurora).

---

## 3. 🧩 Core Concepts

| Concept | Description |
|---|---|
| **DB Instance** | The running database environment (compute + memory + storage). |
| **DB Instance Class** | The hardware size (e.g. `db.t3.medium`, `db.r6g.large`). |
| **DB Engine** | The database software (MySQL, Postgres…). |
| **Endpoint** | DNS name + port apps connect to. |
| **Parameter Group** | Engine configuration settings. |
| **Option Group** | Extra engine features (e.g. Oracle TDE). |
| **Subnet Group** | The VPC subnets (across AZs) RDS can use. |
| **Multi-AZ** | Synchronous standby for failover (HA). |
| **Read Replica** | Async copy for read scaling. |

---

## 4. 🛟 High Availability (Multi-AZ)

- **Multi-AZ deployment** maintains a **synchronous standby** replica in a **different AZ**.
- On failure (AZ outage, instance crash, maintenance), RDS **automatically fails over** to the standby — the **endpoint stays the same** (DNS just repoints), typically in 60–120 s.
- The standby is **not** readable (it's for HA, not scaling) — *(except Multi-AZ DB cluster mode, which adds 2 readable standbys)*.
- **Multi-AZ ≠ backup and ≠ read scaling.** It's purely about availability/durability.

---

## 5. 📖 Read Replicas (Scaling Reads)

- **Asynchronous** copies you can **read from** — offload reporting/read traffic from the primary.
- Up to **15** read replicas (engine-dependent); can be **cross-AZ and cross-region**.
- Each replica has its **own endpoint**; apps must be pointed at it deliberately.
- A replica can be **promoted** to a standalone primary (DR or migration).
- **Replica lag** is possible (async) — don't use for read-after-write consistency.

> **Multi-AZ vs Read Replica:** Multi-AZ = synchronous, automatic failover, not readable (HA). Read Replica = asynchronous, manual use, readable (scaling). You often use **both**.

---

## 6. 💾 Storage Types

| Type | Class | Use |
|---|---|---|
| **General Purpose SSD (gp3/gp2)** | SSD | Most workloads; gp3 lets you scale IOPS/throughput independently. |
| **Provisioned IOPS (io1/io2)** | SSD | I/O-intensive, latency-sensitive production DBs. |
| **Magnetic** | HDD (legacy) | Backwards compatibility only — avoid. |

- **Storage autoscaling** can grow storage automatically as you fill it.
- Storage is **EBS-backed**; you can scale storage up (not down) without downtime.

---

## 7. 💿 Backups, Snapshots & PITR

- **Automated backups** — daily full + transaction logs; retention **0–35 days**. Enable **Point-in-Time Recovery (PITR)** to restore to any second in the window.
- **Manual snapshots** — user-triggered; kept until you delete them (survive instance deletion).
- Backups/snapshots are stored in **S3** (managed by AWS) and can be **copied cross-region** and **shared** across accounts.
- Restoring a snapshot **creates a new instance** (you then repoint the endpoint).

> ⚠️ Automated backups are deleted when the instance is deleted (unless you take a final snapshot). **Manual snapshots persist** — use them before risky changes.

---

## 8. 🔐 Security

- **Network:** launch in a **VPC**, control access with **Security Groups**; keep DBs in **private subnets** (no public access).
- **Auth:** native DB users/passwords, or **IAM database authentication** (token-based), or **Kerberos/AD**.
- **Encryption at rest:** **KMS** (enable at creation — can't easily add later; encrypts storage, backups, snapshots, replicas).
- **Encryption in transit:** SSL/TLS to the endpoint.
- **Secrets:** store/rotate credentials in **Secrets Manager** (native RDS rotation).
- **Audit:** CloudTrail (API) + engine audit logs to CloudWatch.

---

## 9. 🚀 Amazon Aurora

AWS's cloud-native engine, MySQL- and PostgreSQL-compatible:

- **Storage auto-grows** up to 128 TiB; data is **6-way replicated across 3 AZs** automatically.
- Up to **15 low-latency read replicas**; failover in seconds via a **cluster (writer/reader) endpoint**.
- **Aurora Serverless v2** — autoscales capacity (ACUs) up/down with load; great for variable/spiky workloads.
- **Global Database** — sub-second cross-region replication for global apps & DR.
- Typically **3–5× MySQL** / **2–3× PostgreSQL** throughput; pay for storage actually used.

> Choose **Aurora** for performance, automatic storage scaling, and fast failover; choose **standard RDS engines** for exact version parity, lower entry cost, or specific feature needs.

---

## 10. 🔧 Maintenance & Scaling

- **Maintenance window** — weekly window for patching; some changes need a brief restart.
- **Vertical scaling** — change the **instance class** (brief downtime, or use Multi-AZ to minimize).
- **Storage scaling** — increase storage / IOPS online; **autoscaling** optional.
- **Read scaling** — add **read replicas**; **Aurora Serverless** scales compute automatically.
- **Blue/Green Deployments** — create a synchronized staging copy, test, then switch over with minimal downtime.

---

## 11. 📊 Monitoring

- **CloudWatch metrics** — CPU, connections, freeable memory, IOPS, storage, replica lag.
- **Enhanced Monitoring** — OS-level metrics at up to 1-second granularity.
- **Performance Insights** — visual DB load + top SQL/wait analysis for tuning.
- **Event subscriptions** (SNS) — failovers, low storage, backups, etc.
- **CloudTrail** — management API audit.

---

## 12. 💰 Pricing

You pay for:

1. **DB instance hours** — by class & engine; On-Demand or **Reserved Instances** (1–3 yr, big savings).
2. **Storage** — per GB-month provisioned (+ provisioned IOPS for io1/io2).
3. **Backup storage** — free up to DB size; extra beyond that.
4. **Data transfer OUT** and **cross-region** replication.
5. **Multi-AZ** roughly doubles instance/storage cost (standby).
6. **Extras** — Performance Insights (long retention), snapshots export, etc.

> **💸 Cost tips:** use **Reserved Instances** for steady prod · right-size the class · **Graviton (`g`)** instances for price/perf · stop non-prod (up to 7 days at a time) · trim old snapshots · use **Aurora Serverless** for spiky/dev workloads.

---

## 13. ⌨️ Common CLI Commands

```bash
aws rds create-db-instance --db-instance-identifier mydb \
  --db-instance-class db.t3.micro --engine postgres \
  --master-username admin --master-user-password '****' \
  --allocated-storage 20 --vpc-security-group-ids sg-123 --multi-az

aws rds describe-db-instances
aws rds create-db-snapshot --db-instance-identifier mydb --db-snapshot-identifier snap1
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier mydb-restored --db-snapshot-identifier snap1
aws rds create-db-instance-read-replica \
  --db-instance-identifier mydb-replica --source-db-instance-identifier mydb
aws rds modify-db-instance --db-instance-identifier mydb \
  --db-instance-class db.r6g.large --apply-immediately
aws rds reboot-db-instance --db-instance-identifier mydb --force-failover
aws rds delete-db-instance --db-instance-identifier mydb --final-db-snapshot-identifier final-snap
```

---

## 14. 🆚 RDS vs Other AWS Databases

| Service | Type | When |
|---|---|---|
| **RDS** | Managed relational (SQL) | Traditional OLTP apps needing a specific engine. |
| **Aurora** | Cloud-native relational | High performance/scale relational with auto-storage & fast failover. |
| **DynamoDB** | Managed NoSQL (key-value/document) | Massive scale, single-digit-ms, serverless, flexible schema. |
| **ElastiCache** | In-memory (Redis/Memcached) | Caching, sessions, leaderboards, sub-ms reads. |
| **Redshift** | Data warehouse (OLAP) | Analytics over huge datasets. |
| **DocumentDB / Neptune / Keyspaces / Timestream** | Purpose-built | Documents / graphs / Cassandra / time-series. |

---

## 15. 🧠 Quick Mental Model

- **RDS = managed SQL.** AWS runs the host/OS/engine maintenance; you own data + queries (no SSH).
- Pick an **engine** (MySQL, Postgres, MariaDB, Oracle, SQL Server, or **Aurora**).
- **Multi-AZ** = synchronous standby for **automatic failover** (HA), same endpoint — *not* readable, *not* a backup.
- **Read Replicas** = async, readable copies for **scaling reads** (own endpoint, possible lag).
- **Backups** (automated + PITR) and **manual snapshots** (persist) protect data — snapshot before risky changes.
- Keep it **private in a VPC**, encrypt with **KMS**, manage creds in **Secrets Manager**.
- **Aurora** for cloud-native scale/perf; **Reserved Instances + right-sizing** for cost.

---

## 16. ❓ Common Interview Questions

Rapid-fire questions interviewers ask about RDS:

- **Q: Multi-AZ vs Read Replica?** — Multi-AZ = synchronous standby for **automatic failover** (HA), same endpoint, not readable. Read Replica = asynchronous, **readable** copy for scaling reads (own endpoint, possible lag).
- **Q: Does Multi-AZ improve read performance?** — No. It's for availability/durability only; use read replicas to scale reads.
- **Q: Automated backup vs snapshot?** — Automated = daily + transaction logs, enables PITR, deleted with the instance. Manual snapshot = user-triggered, **persists** until deleted.
- **Q: What is Amazon Aurora?** — AWS cloud-native MySQL/Postgres-compatible engine; storage auto-grows, 6-way replicated across 3 AZs, fast failover, up to 15 replicas.
- **Q: How do you secure RDS?** — Private subnets, security groups, KMS encryption at rest (set at creation), TLS in transit, Secrets Manager for credentials, IAM auth optional.
- **Q: Can you SSH into an RDS instance?** — No — it's managed; you get a DB endpoint, not OS access.
- **Q: How to scale an RDS database?** — Vertically (change instance class), storage autoscaling, read replicas for reads, or Aurora Serverless for variable load.
- **Q: How to migrate with minimal downtime?** — Blue/Green Deployments or AWS DMS.

---

## 17. 🎯 Detailed Interview Q&A and When to Use

> Try each question first, then open its answer below. Every answer opens with a plain-English *In plain terms* line, then the technical detail and **when to use** guidance — aimed at readers newer to AWS/DevOps.

### Questions

**Basics**

- [Q1. What does RDS manage for you?](#q1-what-does-rds-manage-for-you)
- [Q2. Engines; RDS MySQL/Postgres vs Aurora?](#q2-engines-rds-mysqlpostgres-vs-aurora)
- [Q3. Multi-AZ vs Read Replica — be precise.](#q3-multi-az-vs-read-replica--be-precise)
- [Q4. RDS snapshot; automated backups vs manual snapshots?](#q4-rds-snapshot-automated-backups-vs-manual-snapshots)
- [Q5. Parameter group vs option group?](#q5-parameter-group-vs-option-group)

**Intermediate**

- [Q6. Multi-AZ — does the standby serve reads? Then what's it for?](#q6-multi-az--does-the-standby-serve-reads-then-whats-it-for)
- [Q7. Multi-AZ failover — what happens, how does the app find the new primary, how long?](#q7-multi-az-failover--what-happens-how-does-the-app-find-the-new-primary-how-long)
- [Q8. Why connect to the endpoint, never the instance IP?](#q8-why-connect-to-the-endpoint-never-the-instance-ip)
- [Q9. Read replica lag — detect, and what breaks?](#q9-read-replica-lag--detect-and-what-breaks)
- [Q10. Promote a read replica? Use case?](#q10-promote-a-read-replica-use-case)
- [Q11. Encrypt RDS; gotcha on an existing unencrypted DB?](#q11-encrypt-rds-gotcha-on-an-existing-unencrypted-db)
- [Q12. Connect to RDS with no stored password?](#q12-connect-to-rds-with-no-stored-password)
- [Q13. Storage autoscaling vs provisioning a big disk up front?](#q13-storage-autoscaling-vs-provisioning-a-big-disk-up-front)

**Advanced & Scenario**

- [Q14. Scenario — Postgres high CPU + slow queries at peak.](#q14-scenario--postgres-high-cpu--slow-queries-at-peak)
- [Q15. Scenario — major engine upgrade with near-zero downtime.](#q15-scenario--major-engine-upgrade-with-near-zero-downtime)
- [Q16. Scenario — "too many connections," DB CPU is fine.](#q16-scenario--too-many-connections-db-cpu-is-fine)
- [Q17. Why RDS Proxy, especially for serverless?](#q17-why-rds-proxy-especially-for-serverless)
- [Q18. Aurora storage architecture — why faster failover and replica creation?](#q18-aurora-storage-architecture--why-faster-failover-and-replica-creation)
- [Q19. Scenario — accidental `DELETE FROM users` 20 minutes ago. Recovery?](#q19-scenario--accidental-delete-from-users-20-minutes-ago-recovery)
- [Q20. Scenario — survive a full region outage, RTO in minutes.](#q20-scenario--survive-a-full-region-outage-rto-in-minutes)

---

### Answers

#### Q1. What does RDS manage for you?

*In plain terms:* A managed database — AWS handles backups, patching, failover and replication so you don't babysit the DB server; you lose OS/root access in return.

Provisioning, patching, backups, automated failover (Multi-AZ), replication, monitoring, and point-in-time recovery. You stop hand-rolling backups, replication, and OS/DB patching — but you give up OS/superuser access.
**When to use:** RDS → want managed backups/patching/HA on a standard engine, focus on the app. Self-managed DB on EC2 → need an unsupported engine/version, OS/superuser access, custom extensions, or extreme cost control with an in-house DBA.

[↩ Back to questions](#questions)

#### Q2. Engines; RDS MySQL/Postgres vs Aurora?

*In plain terms:* RDS runs standard engines (MySQL/Postgres/etc.); Aurora is AWS's souped-up, cloud-native version — faster failover and replicas, but pricier.

RDS supports MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, plus Aurora. Aurora is AWS's cloud-native MySQL/Postgres-compatible engine with a distributed storage layer — faster failover, faster replicas, autoscaling storage — at higher cost.
**When to use:** RDS (vanilla) → standard needs, lower cost, specific engine/version. Aurora → need fast failover, many fast read replicas, autoscaling storage, higher throughput, and budget allows.

[↩ Back to questions](#questions)

#### Q3. Multi-AZ vs Read Replica — be precise.

*In plain terms:* Multi-AZ = a standby that auto-takes-over if the DB dies (for uptime); read replica = extra copies to spread read load — different jobs.

Multi-AZ = synchronous standby in another AZ for HA/failover, not readable. Read Replica = asynchronous copy for scaling reads, readable, can lag. One is for availability, the other for read throughput.
**When to use:** Multi-AZ → high availability / failover (not scaling). Read Replica → scale read traffic / offload reporting (not failover). Both → HA + read scaling.

[↩ Back to questions](#questions)

#### Q4. RDS snapshot; automated backups vs manual snapshots?

*In plain terms:* Automated backups run daily and let you rewind to any minute, but die with the DB; manual snapshots you take and keep forever.

Automated backups run on a schedule + transaction logs enable PITR, retained up to 35 days, deleted when you delete the instance. Manual snapshots are user-triggered and persist until you explicitly delete them.
**When to use:** Automated backups → daily ops + PITR within retention. Manual snapshot → long-term retention, before risky changes, keeping data after deleting the instance, cross-account/region copy.

[↩ Back to questions](#questions)

#### Q5. Parameter group vs option group?

*In plain terms:* Parameter group tweaks DB settings (like max connections); option group turns on extra features (like Oracle encryption) — settings vs add-ons.

Parameter group = engine config knobs (`max_connections`, timeouts, logging). Option group = additional engine features/plugins (Oracle TDE, SQL Server Audit).
**When to use:** Parameter group → tune engine config. Option group → enable engine features.

[↩ Back to questions](#questions)

#### Q6. Multi-AZ — does the standby serve reads? Then what's it for?

*In plain terms:* No, the standby just sits there ready to take over — it's for staying online, not for extra read capacity.

No, the standby serves no traffic. It exists purely for automatic failover — a synchronous replica that takes over (same endpoint) if the primary fails, an AZ goes down, or during patching/maintenance.

[↩ Back to questions](#questions)

#### Q7. Multi-AZ failover — what happens, how does the app find the new primary, how long?

*In plain terms:* When the primary fails, AWS repoints the DNS name to the standby and the app reconnects — usually about a minute.

RDS flips the DNS CNAME endpoint to the promoted standby; the app keeps using the same endpoint and reconnects. Typically ~60–120 seconds. The app must handle reconnection and short DNS TTL caching — failover is transparent at the endpoint, not the IP.

[↩ Back to questions](#questions)

#### Q8. Why connect to the endpoint, never the instance IP?

*In plain terms:* Always connect using the DNS name, not the IP — the IP changes on failover, but the name always points to the current primary.

The underlying instance IP changes on failover, replacement, or maintenance. The DNS endpoint always resolves to the current primary, so connecting by endpoint survives failover; pinning an IP breaks the moment RDS moves the primary.

[↩ Back to questions](#questions)

#### Q9. Read replica lag — detect, and what breaks?

*In plain terms:* Read copies trail the primary slightly, so a user might not see their own just-made change — watch the lag metric and route critical reads to the primary.

Async replication means replicas trail the primary. Monitor `ReplicaLag` in CloudWatch. High lag = stale reads (a user writes then reads a replica and doesn't see their change) and stale reporting. Mitigate by routing critical reads to the primary.

[↩ Back to questions](#questions)

#### Q10. Promote a read replica? Use case?

*In plain terms:* You can turn a read copy into its own independent database — handy for disaster recovery, migrations or splitting off a workload.

Yes — promotion makes it a standalone independent primary (breaks replication).
**When to use:** DR failover, regional migration, sharding off a workload, or a low-downtime major-version upgrade path.

[↩ Back to questions](#questions)

#### Q11. Encrypt RDS; gotcha on an existing unencrypted DB?

*In plain terms:* You must turn on encryption when the DB is created; to encrypt an existing one you snapshot it, copy the snapshot with encryption, and restore.

Encryption (KMS) is set at creation and covers storage, snapshots, replicas, backups. You can't encrypt an existing unencrypted instance in place — snapshot it, copy the snapshot with encryption enabled, and restore from the encrypted copy.
**When to use:** Enable encryption at creation for any sensitive/regulated data (make it a default policy).

[↩ Back to questions](#questions)

#### Q12. Connect to RDS with no stored password?

*In plain terms:* Skip stored passwords — either fetch a short-lived login token via IAM, or pull the password from a vault that rotates it automatically.

IAM database authentication — the app fetches a short-lived token, no static password. Or Secrets Manager with automatic rotation — credentials fetched at runtime and rotated on a schedule.
**When to use:** IAM DB auth → short-lived tokens, apps/Lambda already on IAM. Secrets Manager → standard username/password with automatic rotation, broad engine support, shared credentials.

[↩ Back to questions](#questions)

#### Q13. Storage autoscaling vs provisioning a big disk up front?

*In plain terms:* Let the disk grow itself when it fills up so you never hit 'disk full' — versus guessing big up front and paying for empty space.

Autoscaling grows the volume when free space drops below a threshold (up to a cap), so you pay for what you use and avoid "disk full" outages.
**When to use:** Autoscaling → unpredictable growth, avoid disk-full outages. Big fixed disk → predictable size where you want a hard cap on cost.

[↩ Back to questions](#questions)

#### Q14. Scenario — Postgres high CPU + slow queries at peak.

*In plain terms:* Drill from the CPU graph down to the actual slow query using Performance Insights, then add an index — fix the query before buying a bigger box.

Top-down: CloudWatch (CPU, connections, IOPS) → Performance Insights for top SQL by load and wait events → `pg_stat_statements` for worst queries → `EXPLAIN ANALYZE` for plans → check missing indexes, sequential scans, lock contention, autovacuum bloat. Fix the query/index before scaling the instance.

[↩ Back to questions](#questions)

#### Q15. Scenario — major engine upgrade with near-zero downtime.

*In plain terms:* Use Blue/Green: build an upgraded copy kept in sync, test it, then switch over in about a minute with easy rollback.

RDS Blue/Green Deployments: spins up a green copy at the new version kept in sync via replication, you test it, then switch over in ~a minute. Alternative: create a read replica, upgrade it, promote.
**When to use:** Blue/Green → major version/engine upgrades or schema changes on prod with near-zero downtime and a tested rollback.

[↩ Back to questions](#questions)

#### Q16. Scenario — "too many connections," DB CPU is fine.

*In plain terms:* The DB ran out of connection slots, not CPU — add a connection pooler like RDS Proxy instead of a bigger instance.

You've hit `max_connections` — too many open/idle connections, not compute load. Usually no connection pooling (each app instance/Lambda opening its own). Fix with a pooler, not a bigger instance.

[↩ Back to questions](#questions)

#### Q17. Why RDS Proxy, especially for serverless?

*In plain terms:* Lambda can open thousands of connections at once and drown the DB — RDS Proxy shares a small pool and holds connections through failover.

Lambda scales to thousands of concurrent executions, each opening a DB connection → instant exhaustion. RDS Proxy pools and reuses connections, smooths failover (holds client connections during it), and integrates with IAM/Secrets Manager.
**When to use:** Serverless/Lambda or many app instances exhausting connections; want faster, transparent failover and IAM/Secrets integration.

[↩ Back to questions](#questions)

#### Q18. Aurora storage architecture — why faster failover and replica creation?

*In plain terms:* Aurora splits compute from a shared storage layer copied 6 ways across 3 zones, so adding replicas and failing over is near-instant — no data copying.

Aurora separates compute from a shared distributed storage layer that keeps 6 copies across 3 AZs. Replicas read the same storage instead of copying data, so adding a replica is fast and failover just promotes a reader pointing at the same volume — no data movement, sub-30s typical.

[↩ Back to questions](#questions)

#### Q19. Scenario — accidental `DELETE FROM users` 20 minutes ago. Recovery?

*In plain terms:* Use point-in-time recovery to restore a copy to the moment just before the bad DELETE, then repoint or copy the rows back.

Point-in-time recovery (PITR) restores to a timestamp just before the delete into a new instance (automated backups + transaction logs make this possible). Restore, validate, then repoint the app or copy the rows back. Minimizes data loss to the moment before the bad statement.

[↩ Back to questions](#questions)

#### Q20. Scenario — survive a full region outage, RTO in minutes.

*In plain terms:* For a whole-region failure: Aurora Global Database fails over in about a minute; cross-region replicas are cheaper but slower; snapshot copies are cheapest but take hours.

Aurora Global Database = primary region + secondary region with storage-level replication (typical lag < 1s), promote secondary in ~1 min. Cross-region read replica = cheaper but slower/lossier promotion. Snapshot copy = cheapest but RTO in hours. Lower RTO/RPO costs more (a warm standby region running 24/7).
**When to use:** Aurora Global Database → RTO/RPO seconds-to-minutes, can pay for a warm secondary. Cross-region read replica → cheaper, higher RPO, slower promote. Snapshot copy → cheapest, RTO in hours, non-critical only.

[↩ Back to questions](#questions)

---

<div align="center">

*📝 Notes compiled as a quick reference for Amazon RDS.*

</div>
