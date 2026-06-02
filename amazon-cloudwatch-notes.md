<div align="center">

<img src="cloudwatch-logo.svg" width="110" alt="Amazon CloudWatch logo" />

# Amazon CloudWatch — Complete Notes

**Observability** · Metrics, logs, alarms & events for AWS

![AWS](https://img.shields.io/badge/AWS-CloudWatch-E7157B?style=flat-square&logoColor=white)
![Type](https://img.shields.io/badge/Service-Monitoring-E7157B?style=flat-square)
![Pillars](https://img.shields.io/badge/Metrics-Logs-Alarms-blue?style=flat-square)
![Scope](https://img.shields.io/badge/Scope-Regional-lightgrey?style=flat-square)

*A reader-friendly, all-in-one reference for Amazon CloudWatch.*

<!--PDF-BUTTON-START-->

[<img src="https://img.shields.io/badge/⬇%20Download%20as%20PDF-E7157B?style=for-the-badge&logo=adobeacrobatreader&logoColor=white" alt="Download as PDF" height="40">](https://github.com/RamanKharchee/cloud-notes/raw/main/amazon-cloudwatch-notes.pdf)

<sub>Opens the rendered PDF — use your browser's download button to save it.</sub>

<!--PDF-BUTTON-END-->

</div>

---

## 📑 Table of Contents

1. [What is CloudWatch](#1--what-is-cloudwatch)
2. [Core Concepts](#2--core-concepts)
3. [Metrics](#3--metrics)
4. [Alarms](#4--alarms)
5. [Logs & Logs Insights](#5--logs--logs-insights)
6. [Dashboards](#6--dashboards)
7. [Events & EventBridge](#7--events--eventbridge)
8. [CloudWatch Agent](#8--cloudwatch-agent)
9. [Insights (Container, Lambda, App)](#9--insights-container-lambda-app)
10. [CloudWatch vs CloudTrail](#10--cloudwatch-vs-cloudtrail)
11. [Pricing](#11--pricing)
12. [Common CLI Commands](#12--common-cli-commands)
13. [Best Practices](#13--best-practices)
14. [Quick Mental Model](#14--quick-mental-model)
15. [Common Interview Questions](#15--common-interview-questions)

---

## 1. 📈 What is CloudWatch

Amazon CloudWatch is AWS's **observability service** — it collects **metrics**, **logs**, and **events** from AWS services and your apps, lets you visualize them on **dashboards**, and triggers **alarms/actions** when something crosses a threshold. It's how you answer *"is my system healthy, and what do I do when it isn't?"*

> 💡 **Mental model:** CloudWatch is the **monitoring nerve center** of AWS — metrics tell you *what's happening*, logs tell you *why*, alarms *react*, and EventBridge *routes* operational events to automation.

**Common uses:** infrastructure/app monitoring, alerting, auto-recovery, autoscaling triggers, log aggregation & search, dashboards, scheduled jobs.

---

## 2. 🧩 Core Concepts

| Concept | Description |
|---|---|
| **Metric** | A time-ordered set of data points (e.g. `CPUUtilization`). |
| **Namespace** | A container grouping metrics (e.g. `AWS/EC2`). |
| **Dimension** | A name/value pair identifying a metric (e.g. `InstanceId=i-123`). |
| **Statistic** | Aggregation over a period (Avg, Sum, Min, Max, p99). |
| **Alarm** | Watches a metric and changes state / takes action. |
| **Log Group / Stream** | Container / sequence of log events. |
| **Dashboard** | Customizable view of metrics & logs. |
| **Event / Rule** | An operational change routed via EventBridge. |

---

## 3. 📊 Metrics

- AWS services publish metrics automatically (EC2 CPU, ELB request count, DynamoDB throttles, Lambda duration…).
- **Resolution:** standard = **1-minute**; **high-resolution custom metrics** down to **1 second**. EC2 **detailed monitoring** = 1-min (extra cost) vs default 5-min.
- **Custom metrics** — push your own with `PutMetricData` (app KPIs, business metrics).
- **Metric Math** combines metrics into computed series; **anomaly detection** builds a band of expected values.
- ⚠️ EC2 does **not** report **memory** or **disk usage** by default — you need the **CloudWatch agent** for those.

---

## 4. 🚨 Alarms

- An alarm watches one metric (or a Metric Math expression) over N periods and enters a **state**: **OK**, **ALARM**, or **INSUFFICIENT_DATA**.
- **Actions** on state change: notify via **SNS**, trigger **Auto Scaling**, **EC2 actions** (stop/terminate/reboot/recover), or a **Systems Manager** action.
- **Composite alarms** combine several alarms with AND/OR to cut noise.
- Common pattern: `CPU > 70% for 3 minutes` → SNS email + Auto Scaling scale-out.

```
Metric ──▶ Alarm (threshold, periods) ──▶ State change ──▶ Action (SNS / ASG / EC2)
```

---

## 5. 📜 Logs & Logs Insights

- **CloudWatch Logs** centralizes logs into **log groups** (per app/service) and **log streams** (per source). Lambda, ECS, API GW, VPC Flow Logs, and the agent all ship here.
- **Retention** is configurable per log group (1 day → forever; default never-expire = cost trap).
- **Logs Insights** — a fast query language to search/aggregate logs:

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() by bin(5m)
| sort @timestamp desc
```

- **Metric filters** turn log patterns into metrics (e.g. count "ERROR" → alarm). **Subscription filters** stream logs to Lambda/Kinesis/OpenSearch.

---

## 6. 📐 Dashboards

- Customizable, **cross-region/cross-account** views combining metric graphs, logs widgets, alarms, and text.
- Use them as a **single pane of glass** for a service or environment; can be shared.

---

## 7. 🔔 Events & EventBridge

- **CloudWatch Events** (now **Amazon EventBridge**) reacts to **operational events** and **schedules**.
- **Rules** match event patterns (e.g. "EC2 instance entered `stopped`") or run on a **cron/rate schedule**, then route to **targets** (Lambda, SNS, SQS, Step Functions…).
- Classic uses: scheduled Lambda (cron jobs), auto-remediation, event-driven automation.

> EventBridge is the evolution of CloudWatch Events with richer routing, a schema registry, and SaaS/partner event buses.

---

## 8. 🛠️ CloudWatch Agent

- A unified agent installed on EC2/on-prem servers to collect what AWS can't see from outside: **memory, disk, swap, processes**, plus custom **application logs**.
- Configured via a JSON config (or SSM Parameter Store); pushes custom metrics + logs to CloudWatch.

---

## 9. 🔬 Insights (Container, Lambda, App)

| Feature | Gives |
|---|---|
| **Container Insights** | Metrics & logs for ECS / EKS / Kubernetes workloads. |
| **Lambda Insights** | Per-invocation system-level metrics for functions. |
| **Application Insights** | Automated monitoring/setup for common app stacks. |
| **ServiceLens / X-Ray** | Traces + service maps tying metrics, logs, and traces together. |
| **Synthetics** | Canaries that probe endpoints on a schedule. |
| **RUM** | Real-User Monitoring for front-end performance. |

---

## 10. ⚖️ CloudWatch vs CloudTrail

A very common point of confusion:

| | **CloudWatch** | **CloudTrail** |
|---|---|---|
| Answers | "How is it **performing**?" | "**Who did what**, and when?" |
| Data | Metrics, logs, alarms (performance/ops) | API call audit log (governance/security) |
| Use | Monitoring, alerting, dashboards | Auditing, compliance, forensics |

> Memory hook: **CloudWatch = performance/health; CloudTrail = audit/who-did-what.** They're complementary, not interchangeable.

---

## 11. 💰 Pricing

You pay for:

1. **Metrics** — custom metrics per month; API requests (`PutMetricData`, `GetMetricData`).
2. **Logs** — ingestion per GB + storage per GB-month + Logs Insights queries scanned.
3. **Alarms** — per alarm per month (high-resolution costs more).
4. **Dashboards** — per dashboard per month (a free allotment first).
5. **Extras** — Synthetics canary runs, Contributor Insights, RUM events.

> **💸 Cost tips:** set **log retention** (don't keep forever) · avoid excessive high-resolution metrics/alarms · use metric filters instead of streaming everything · sample noisy logs · drop unused custom metrics.

---

## 12. ⌨️ Common CLI Commands

```bash
aws cloudwatch put-metric-data --namespace MyApp \
  --metric-name Latency --value 123 --unit Milliseconds

aws cloudwatch put-metric-alarm --alarm-name high-cpu \
  --namespace AWS/EC2 --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-123 \
  --statistic Average --period 300 --threshold 70 \
  --comparison-operator GreaterThanThreshold --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:...:alerts

aws cloudwatch describe-alarms
aws logs tail /aws/lambda/myfn --follow          # tail a log group
aws logs start-query --log-group-name /aws/lambda/myfn \
  --start-time ... --end-time ... --query-string 'fields @message | filter @message like /ERROR/'
```

---

## 13. ✅ Best Practices

- **Set log retention** on every log group (forever-by-default wastes money).
- Alarm on **symptoms users feel** (latency, error rate, queue depth), not just CPU.
- Use **SNS** for alarm fan-out (email, Slack, PagerDuty); **composite alarms** to reduce noise.
- Install the **agent** for memory/disk + app logs (EC2 doesn't report them by default).
- Build **dashboards** per service; use **Logs Insights** for ad-hoc investigation.
- Wire alarms to **Auto Scaling** and **EC2 auto-recovery** for self-healing.
- Pair with **CloudTrail** for the full picture (performance + audit).

---

## 14. 🧠 Quick Mental Model

- **CloudWatch = AWS observability:** metrics (what), logs (why), alarms (react), events (route).
- Metrics live in **namespaces** with **dimensions**; standard 1-min, high-res 1-sec, custom via `PutMetricData`.
- **Alarms** go OK / ALARM / INSUFFICIENT_DATA and fire **SNS / Auto Scaling / EC2** actions.
- **Logs** → log groups/streams; query with **Logs Insights**; turn patterns into metrics with metric filters.
- **EventBridge** (CloudWatch Events) routes operational events + cron schedules to targets.
- EC2 needs the **agent** for memory/disk; use **Container/Lambda Insights** for those workloads.
- **CloudWatch = performance/health, CloudTrail = audit.**

---

## 15. ❓ Common Interview Questions

Rapid-fire questions interviewers ask about CloudWatch:

- **Q: CloudWatch vs CloudTrail?** — CloudWatch monitors performance/health (metrics, logs, alarms). CloudTrail audits API activity (who did what, when).
- **Q: Does EC2 report memory usage by default?** — No. CPU/network/disk-I/O yes; **memory and disk space** need the **CloudWatch agent**.
- **Q: What are the alarm states?** — OK, ALARM, INSUFFICIENT_DATA. Actions fire on state changes (SNS, Auto Scaling, EC2 recover).
- **Q: How do you alert the team?** — Alarm → **SNS** topic → email/SMS/Slack/PagerDuty subscribers.
- **Q: How do you run a scheduled task?** — An **EventBridge (CloudWatch Events)** cron/rate rule targeting a Lambda.
- **Q: How do you search logs?** — **CloudWatch Logs Insights** query language; or create **metric filters** to alarm on log patterns.
- **Q: Standard vs detailed monitoring (EC2)?** — Standard = 5-minute metrics (free); detailed = 1-minute (extra cost).
- **Q: How do you make a system self-healing?** — Alarms trigger **Auto Scaling** replacement and **EC2 auto-recovery** on failed status checks.
- **Q: How do you control log cost?** — Set **retention** per log group and sample/limit high-cardinality custom metrics.

---

<div align="center">

*📝 Notes compiled as a quick reference for Amazon CloudWatch.*

</div>
