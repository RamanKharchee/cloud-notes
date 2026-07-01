<div align="center">

<img src="https://cdn.simpleicons.org/awslambda/FF9900" width="110" alt="AWS Lambda logo" />

# AWS Lambda — Complete Notes

**Serverless functions** · Run code without managing servers

![AWS](https://img.shields.io/badge/AWS-Lambda-FF9900?style=flat-square&logo=awslambda&logoColor=white)
![Type](https://img.shields.io/badge/Model-Serverless-FF9900?style=flat-square)
![Billing](https://img.shields.io/badge/Billing-Per%20ms-blue?style=flat-square)
![Scaling](https://img.shields.io/badge/Scaling-Automatic-lightgrey?style=flat-square)

*A reader-friendly, all-in-one reference for AWS Lambda.*

<!--PDF-BUTTON-START-->

[<img src="https://img.shields.io/badge/⬇%20Download%20as%20PDF-FF9900?style=for-the-badge&logo=adobeacrobatreader&logoColor=white" alt="Download as PDF" height="40">](https://github.com/RamanKharchee/cloud-notes/raw/main/aws-lambda-notes.pdf)

<sub>Opens the rendered PDF — use your browser's download button to save it.</sub>

<!--PDF-BUTTON-END-->

</div>

---

## 📑 Table of Contents

1. [What is Lambda](#1--what-is-lambda)
2. [Core Concepts](#2--core-concepts)
3. [The Execution Model](#3--the-execution-model)
4. [Triggers & Event Sources](#4--triggers--event-sources)
5. [Configuration (Memory, Timeout, etc.)](#5--configuration-memory-timeout-etc)
6. [Cold Starts & Concurrency](#6--cold-starts--concurrency)
7. [Deployment Packages & Layers](#7--deployment-packages--layers)
8. [Permissions (Execution Role vs Resource Policy)](#8--permissions-execution-role-vs-resource-policy)
9. [Networking (VPC Access)](#9--networking-vpc-access)
10. [Versions & Aliases](#10--versions--aliases)
11. [Error Handling & Retries](#11--error-handling--retries)
12. [Monitoring & Logging](#12--monitoring--logging)
13. [Pricing](#13--pricing)
14. [Common CLI Commands](#14--common-cli-commands)
15. [Limits & Best Practices](#15--limits--best-practices)
16. [Quick Mental Model](#16--quick-mental-model)
17. [Common Interview Questions](#17--common-interview-questions)
18. [Detailed Interview Q&A and When to Use](#18--detailed-interview-qa-and-when-to-use)

---

## 1. ⚡ What is Lambda

AWS Lambda runs your code **in response to events** without you provisioning or managing servers. You upload a **function**, Lambda runs it on demand, scales automatically, and you pay only for the **compute time consumed** (per millisecond). No idle servers, no patching the OS.

> 💡 **Mental model:** Lambda is **Function as a Service (FaaS)**. You own the function code; AWS owns everything below it — servers, scaling, availability, OS.

**Common uses:** API backends (with API Gateway), event processing (S3/DynamoDB/Kafka), stream processing, scheduled jobs (cron), glue between services, real-time file/data transforms, automation.

---

## 2. 🧩 Core Concepts

| Concept | Description |
|---|---|
| **Function** | Your code + config (runtime, memory, role, handler). |
| **Handler** | The entry-point method Lambda invokes (`file.method`). |
| **Event** | The JSON input passed to the handler. |
| **Context** | Runtime info (request id, remaining time, log group). |
| **Runtime** | Language environment: Node.js, Python, Java, Go, .NET, Ruby, or custom. |
| **Trigger / Event Source** | What invokes the function (S3, API GW, EventBridge…). |
| **Execution Role** | IAM role giving the function permissions to call AWS. |
| **Concurrency** | Number of simultaneous executions. |

---

## 3. 🔄 The Execution Model

1. An **event** arrives from a trigger.
2. Lambda provisions an **execution environment** (micro-VM / Firecracker) if none is warm.
3. It runs **init** code (outside the handler) once per environment.
4. It invokes the **handler** with the event + context.
5. The environment is **frozen** and **reused** for later invocations (warm), then eventually torn down.

**Invocation types:**

| Type | How | Example |
|---|---|---|
| **Synchronous** | Caller waits for the response | API Gateway, ALB, direct invoke |
| **Asynchronous** | Event queued; Lambda retries on failure | S3, SNS, EventBridge |
| **Stream / poll-based** | Lambda polls and batches records | Kinesis, DynamoDB Streams, SQS, Kafka |

> 💡 Put expensive setup (DB connections, SDK clients) in the **init** phase (module scope) so warm invocations reuse it.

---

## 4. 🔔 Triggers & Event Sources

Lambda integrates with 200+ services. Common triggers:

- **API Gateway / Function URL / ALB** — HTTP APIs and webhooks.
- **S3** — object created/deleted (process uploads).
- **EventBridge / CloudWatch Events** — schedules (cron) and event rules.
- **SNS / SQS** — pub-sub and queue processing.
- **DynamoDB Streams / Kinesis** — stream processing.
- **MSK / self-managed Kafka** — message consumption.
- **Cognito, Step Functions, IoT, CloudFront (Lambda@Edge)** — and many more.

---

## 5. ⚙️ Configuration (Memory, Timeout, etc.)

| Setting | Range / note |
|---|---|
| **Memory** | 128 MB – 10,240 MB. **CPU scales with memory** (more memory = more vCPU). |
| **Timeout** | 1 s – **900 s (15 min)** max. |
| **Ephemeral storage** (`/tmp`) | 512 MB – 10,240 MB. |
| **Environment variables** | Up to 4 KB total; can be KMS-encrypted. |
| **Architecture** | x86_64 or **arm64 (Graviton)** — arm is cheaper/faster for many workloads. |

> 💡 Tuning **memory** is the main performance lever — more memory also raises CPU, often making functions both faster *and* cheaper. Use **Lambda Power Tuning** to find the sweet spot.

---

## 6. ❄️ Cold Starts & Concurrency

- **Cold start** = the latency to set up a new execution environment (download code, start runtime, run init) before the first invocation. Warm reuse avoids it.
- Bigger packages, VPC attachment, and heavy init increase cold starts. Smaller deps + lazy loading reduce them.
- **Concurrency** = number of simultaneous executions.
  - **Reserved concurrency** — caps (and guarantees) a function's max concurrency.
  - **Provisioned concurrency** — pre-warms N environments → **no cold starts** (extra cost).
- **Account concurrency limit** defaults to **1,000** per region (adjustable). Burst scaling adds environments quickly, then linearly.

---

## 7. 📦 Deployment Packages & Layers

- **Package types:** a **.zip archive** (code + deps) or a **container image** (up to 10 GB, OCI).
- **Zip limits:** 50 MB zipped (direct upload), 250 MB unzipped; larger via S3.
- **Layers** — share common libraries/dependencies across functions (up to 5 layers per function). Keeps deployment packages small.
- **Runtimes:** managed (Node, Python, Java, Go, .NET, Ruby) or **custom runtime** via the Runtime API / `provided.al2`.

---

## 8. 🔐 Permissions (Execution Role vs Resource Policy)

Two distinct permission directions — don't confuse them:

| | **Execution Role** | **Resource-based Policy** |
|---|---|---|
| Question | What can the **function call**? | Who can **invoke the function**? |
| Type | IAM role assumed by Lambda | Policy attached to the function |
| Example | Read S3, write DynamoDB, put logs | Allow S3/EventBridge/API GW to invoke |

> The function always needs `logs:CreateLogGroup/Stream/PutLogEvents` (usually via the `AWSLambdaBasicExecutionRole` managed policy) to write to CloudWatch.

---

## 9. 🌐 Networking (VPC Access)

- By default a Lambda runs in an AWS-managed network with internet access.
- Attach it to a **VPC** (subnets + security group) to reach private resources (RDS, ElastiCache, internal services).
- A VPC-attached Lambda has **no internet by default** — it needs a **NAT Gateway** (in a private subnet) for outbound internet, or **VPC Endpoints** for AWS services.
- Modern Lambda uses **Hyperplane ENIs** (shared), so VPC cold-start penalty is now minimal.

---

## 10. 🏷️ Versions & Aliases

- **$LATEST** — the mutable working version.
- **Published version** — an immutable snapshot (`:1`, `:2`…).
- **Alias** — a named, movable pointer to a version (`prod`, `staging`).
- Aliases enable **weighted/canary deploys** (e.g. 10% traffic to v2, 90% to v1) and clean rollbacks.

---

## 11. 🔁 Error Handling & Retries

- **Synchronous** invokes: the error returns to the caller; **no automatic retry** (caller decides).
- **Asynchronous** invokes: Lambda **retries twice** (3 total attempts) over time; failures go to a **Dead-Letter Queue (SQS/SNS)** or an **on-failure destination**.
- **Stream/poll** sources (Kinesis/DynamoDB): retry by **batch** until success or record expiry; can block the shard — use bisect-on-error, max retry, and a failure destination.
- **Destinations** (async) route success/failure events to SQS, SNS, Lambda, or EventBridge.

---

## 12. 📊 Monitoring & Logging

- **CloudWatch Logs** — automatic; `console.log`/`print` lands in a log group per function.
- **CloudWatch Metrics** — Invocations, Errors, Duration, Throttles, ConcurrentExecutions.
- **AWS X-Ray** — distributed tracing of the function and its downstream calls.
- **Lambda Insights** — enhanced per-invocation system metrics.
- **CloudTrail** — API/management audit.

---

## 13. 💰 Pricing

You pay for:

1. **Requests** — per invocation (first 1M/month free, then per million).
2. **Duration** — GB-seconds = (memory ÷ 1024) × execution time, billed per **1 ms**.
3. **Extras** — Provisioned Concurrency (per GB-s reserved), ephemeral storage above 512 MB, data transfer.

> **💸 Cost tips:** right-size **memory** (it sets CPU too) · use **arm64/Graviton** · trim dependencies and init time · avoid idle Provisioned Concurrency · batch records from streams/queues · keep functions short and single-purpose.

---

## 14. ⌨️ Common CLI Commands

```bash
aws lambda create-function --function-name myfn --runtime python3.12 \
  --handler app.handler --role arn:aws:iam::123:role/lambda-role \
  --zip-file fileb://function.zip

aws lambda invoke --function-name myfn --payload '{"key":"val"}' out.json
aws lambda update-function-code --function-name myfn --zip-file fileb://function.zip
aws lambda update-function-configuration --function-name myfn --memory-size 512 --timeout 30
aws lambda publish-version --function-name myfn
aws lambda create-alias --function-name myfn --name prod --function-version 1
aws lambda list-functions
aws lambda get-function --function-name myfn
```

---

## 15. 📌 Limits & Best Practices

| Item | Value |
|---|---|
| Max timeout | **15 min (900 s)** |
| Memory | 128 MB – 10,240 MB |
| Ephemeral `/tmp` | 512 MB – 10,240 MB |
| Default concurrency | 1,000 / region (adjustable) |
| Deployment zip | 50 MB (zipped) / 250 MB (unzipped) |
| Container image | up to 10 GB |
| Invocation payload | 6 MB (sync), 256 KB (async) |
| Layers per function | 5 |

**Best practices:** keep functions **small & single-purpose** · do heavy setup in **init** and reuse connections · don't store state in the environment (it's ephemeral) · use **environment variables** (encrypted) for config · least-privilege **execution role** · set sensible **timeouts + DLQ** · use **aliases** for safe deploys · prefer **Provisioned Concurrency** only where cold starts hurt.

---

## 16. 🧠 Quick Mental Model

- **Lambda = run code on events, pay per millisecond, zero servers to manage.**
- A **trigger** sends an **event** → Lambda runs your **handler** with an **execution role's** permissions.
- Environments are **reused** (warm) — put setup in **init**; first call may be a **cold start**.
- Tune **memory** (it also sets CPU); cap blast radius with **reserved**, kill cold starts with **provisioned** concurrency.
- **Sync** = caller waits, no retry; **async** = queued + retried + DLQ; **streams** = batched polling.
- Attach to a **VPC** for private resources (needs NAT/endpoints for outbound).
- Ship safely with **versions + aliases**; watch it with **CloudWatch + X-Ray**.

---

## 17. ❓ Common Interview Questions

Rapid-fire questions interviewers ask about Lambda:

- **Q: What is a cold start?** — The latency to set up a new execution environment (download code, start runtime, run init) before the first invocation. Provisioned Concurrency eliminates it.
- **Q: Sync vs async vs stream invocation?** — Sync = caller waits, no auto-retry (API GW). Async = queued, retried twice, DLQ on failure (S3/SNS). Stream/poll = batched records (Kinesis/DynamoDB/SQS).
- **Q: Execution role vs resource-based policy?** — Execution role = what the function **can call**. Resource policy = **who can invoke** the function.
- **Q: How do you tune Lambda performance?** — Increase **memory** (CPU scales with it), trim dependencies, reuse connections via the init phase, use arm64/Graviton.
- **Q: Max timeout and memory?** — 15 minutes; 128 MB–10,240 MB.
- **Q: How to reach a private RDS from Lambda?** — Attach the function to the VPC (subnets + SG); add NAT/endpoints if it also needs outbound internet.
- **Q: How do versions and aliases help?** — Immutable versions + movable aliases enable canary/weighted deploys and clean rollbacks.
- **Q: When NOT to use Lambda?** — Long-running (>15 min) jobs, very high steady throughput where always-on compute is cheaper, or workloads needing heavy local state.

---

## 18. 🎯 Detailed Interview Q&A and When to Use

> Try each question first, then open its answer below. Every answer opens with a plain-English *In plain terms* line, then the technical detail and **when to use** guidance — aimed at readers newer to AWS/DevOps.

### Questions

**Basics**

- [Q1. What is Lambda, and when serverless over servers?](#q1-what-is-lambda-and-when-serverless-over-servers)
- [Q2. How is a Lambda invoked — sync, async, or poll-based?](#q2-how-is-a-lambda-invoked--sync-async-or-poll-based)
- [Q3. Cold starts — cause, and how do you reduce them?](#q3-cold-starts--cause-and-how-do-you-reduce-them)
- [Q4. Memory and CPU coupling?](#q4-memory-and-cpu-coupling)
- [Q5. Reserved vs provisioned concurrency?](#q5-reserved-vs-provisioned-concurrency)
- [Q6. The 15-minute timeout — what if you need longer?](#q6-the-15-minute-timeout--what-if-you-need-longer)

**Intermediate**

- [Q7. Putting a Lambda in a VPC — why, and the cost?](#q7-putting-a-lambda-in-a-vpc--why-and-the-cost)
- [Q8. Error handling and retries by invocation type?](#q8-error-handling-and-retries-by-invocation-type)
- [Q9. Zip vs container-image packaging?](#q9-zip-vs-container-image-packaging)
- [Q10. Secrets and environment variables?](#q10-secrets-and-environment-variables)
- [Q11. Lambda layers?](#q11-lambda-layers)
- [Q12. Versions and aliases?](#q12-versions-and-aliases)

**Advanced & Scenario**

- [Q13. Scenario — Lambda is exhausting RDS connections under load.](#q13-scenario--lambda-is-exhausting-rds-connections-under-load)
- [Q14. Scenario — function times out or is slow intermittently.](#q14-scenario--function-times-out-or-is-slow-intermittently)
- [Q15. Scenario — async events are silently disappearing.](#q15-scenario--async-events-are-silently-disappearing)
- [Q16. Concurrency limits and throttling?](#q16-concurrency-limits-and-throttling)
- [Q17. Lambda vs Fargate vs EC2 — decide.](#q17-lambda-vs-fargate-vs-ec2--decide)
- [Q18. EventBridge vs SQS vs SNS as a trigger?](#q18-eventbridge-vs-sqs-vs-sns-as-a-trigger)

---

### Answers

#### Q1. What is Lambda, and when serverless over servers?

*In plain terms:* Run a function on demand with no servers to manage — billed per request and it scales to zero when idle; great for glue and spiky work.

Run code in response to events with no servers to manage; you pay per request + GB-second and it scales to zero. AWS handles provisioning, patching, and scaling.
**When to use:** Lambda → event-driven, spiky, short tasks (glue, APIs, stream/file processing). Fargate → long-running containers, steady load. EC2 → full OS control, stateful, or cheaper at constant high throughput.

[↩ Back to questions](#questions)

#### Q2. How is a Lambda invoked — sync, async, or poll-based?

*In plain terms:* Three ways it's triggered — someone waits for the reply (sync), it's queued fire-and-forget (async), or it polls a queue/stream in batches.

Sync (API Gateway, ALB) = caller waits, no automatic retry. Async (S3, SNS, EventBridge) = AWS queues it, retries twice, then DLQ/destination. Poll-based event-source mapping (SQS, Kinesis, DynamoDB Streams) = Lambda polls and invokes in batches.
**When to use:** Sync → request/response APIs. Async → fire-and-forget side effects. Poll-based → stream/queue processing with batching.

[↩ Back to questions](#questions)

#### Q3. Cold starts — cause, and how do you reduce them?

*In plain terms:* The first call after idle is slow because AWS spins up a fresh environment — shrink the package, lazy-load, or keep some pre-warmed (provisioned concurrency).

A cold start is the extra latency to create a new execution environment (download code, init runtime, run init code). Worse with large packages, heavy init, VPC ENI attach, and some runtimes.
**When to use provisioned concurrency:** latency-sensitive paths (user-facing APIs) that can't tolerate cold-start spikes; otherwise trim the package, lazy-load, and reuse connections in the init phase.

[↩ Back to questions](#questions)

#### Q4. Memory and CPU coupling?

*In plain terms:* There's no CPU dial — you get more CPU by giving the function more memory, so bumping memory can make it finish faster (and sometimes cheaper).

CPU (and network/disk) scales **with the memory** you allocate — there's no separate CPU knob. Bumping memory often makes a CPU-bound function finish faster and sometimes cheaper.
**When to use more memory:** CPU-bound work, or to cut duration on latency-sensitive functions (tune with Lambda Power Tuning).

[↩ Back to questions](#questions)

#### Q5. Reserved vs provisioned concurrency?

*In plain terms:* Reserved concurrency guarantees/caps how many run at once; provisioned keeps some warm to kill cold-start lag (and costs even when idle).

Reserved = caps/guarantees a slice of the account concurrency pool for a function. Provisioned = keeps N environments **pre-warmed** (no cold start), and costs even when idle.
**When to use:** Reserved → guarantee capacity and/or protect a downstream (DB) from a function's burst. Provisioned → eliminate cold-start latency on hot, user-facing functions.

[↩ Back to questions](#questions)

#### Q6. The 15-minute timeout — what if you need longer?

*In plain terms:* Lambda tops out at 15 minutes — for longer or multi-step jobs, chain them with Step Functions or move to Fargate/Batch.

Lambda caps at 15 min. For longer or multi-step work, orchestrate with **Step Functions**, or move the job to **Fargate/ECS/Batch**.
**When to use Step Functions:** long-running, multi-step, or workflows needing retries/branching/human-approval across many Lambdas.

[↩ Back to questions](#questions)

#### Q7. Putting a Lambda in a VPC — why, and the cost?

*In plain terms:* Put it in your private network only when it must reach private things like a database — then it loses default internet, so add a NAT/endpoints.

Attach it to private subnets + an SG when it must reach private resources (RDS, internal services). It uses Hyperplane ENIs (shared, fast attach). In a VPC it **loses default internet access** — outbound needs a NAT Gateway; AWS APIs are better reached via VPC endpoints.
**When to use VPC attachment:** only when you need private network access (e.g. RDS); keep functions out of the VPC otherwise to avoid the extra networking.

[↩ Back to questions](#questions)

#### Q8. Error handling and retries by invocation type?

*In plain terms:* Sync = the caller retries; async = AWS retries twice then sends failures to a dead-letter queue; queue/stream = retried until it succeeds or expires.

Sync = the caller must retry. Async = AWS retries twice then sends to a **DLQ** or an **on-failure destination**. Poll-based = retried until success or record expiry; SQS failures go back to the queue/DLQ; streams can bisect-on-error.
**When to use DLQ vs destinations:** Destinations (richer — success *and* failure routing to SQS/SNS/EventBridge/Lambda) for new work; DLQ for a simple failed-event capture.

[↩ Back to questions](#questions)

#### Q9. Zip vs container-image packaging?

*In plain terms:* Ship as a small zip (fastest) or as a container image (up to 10 GB, your own base) — image for big dependencies or existing Docker workflows.

Zip = simplest, fastest cold start, 250 MB unzipped limit. Container image = up to 10 GB, use your own base/tooling, fits existing Docker workflows.
**When to use a container image:** large dependencies/models, custom runtimes, or to unify with an existing container build pipeline.

[↩ Back to questions](#questions)

#### Q10. Secrets and environment variables?

*In plain terms:* Non-secret config in env vars is fine, but real secrets should be pulled from a vault at runtime, not left in plain environment variables.

Env vars are fine for non-sensitive config (and are encrypted at rest with KMS), but plaintext secrets leak via the console/`get-function`. Pull secrets from **Secrets Manager/SSM** at runtime (the Lambda extension caches them).
**When to use Secrets Manager vs Parameter Store:** Secrets Manager for rotating credentials; Parameter Store for cheap config at scale.

[↩ Back to questions](#questions)

#### Q11. Lambda layers?

*In plain terms:* A layer is shared libraries mounted into many functions, so common dependencies aren't bundled into every one — not for fast-changing app code.

A layer is a shared zip of libraries/runtime content mounted into functions, so common dependencies aren't bundled into every function.
**When to use layers:** shared dependencies across many functions, a custom runtime, or to shrink the deployment package — not for fast-changing app code.

[↩ Back to questions](#questions)

#### Q12. Versions and aliases?

*In plain terms:* Publish immutable versions and point a movable alias (like 'prod') at them — lets you shift a little traffic to a new version and roll back instantly.

A published **version** is immutable; an **alias** is a movable pointer (e.g. `prod`) that can split traffic between two versions for canary/weighted deploys and instant rollback.
**When to use:** safe production deploys — shift 10% to a new version behind an alias, watch metrics, then complete or roll back.

[↩ Back to questions](#questions)

#### Q13. Scenario — Lambda is exhausting RDS connections under load.

*In plain terms:* Each running copy opens its own DB connection, so a spike drowns the database — put RDS Proxy in front and buffer the burst with a queue.

Each concurrent execution opens its own DB connection, so a burst blows past `max_connections`. Front RDS with **RDS Proxy** to pool/reuse connections; buffer the spike with **SQS + reserved concurrency** so a burst becomes a drained queue, not a connection storm.

[↩ Back to questions](#questions)

#### Q14. Scenario — function times out or is slow intermittently.

*In plain terms:* Intermittent slowness is usually cold starts (especially in a VPC), a slow downstream call, or too little memory — trace it and raise memory.

Check for **cold starts** (especially VPC ENI attach), **downstream latency** (DB/API), and **under-provisioned memory** (CPU-starved). Read Performance Insights/X-Ray, raise memory, reuse connections in init, and consider provisioned concurrency for the hot path.

[↩ Back to questions](#questions)

#### Q15. Scenario — async events are silently disappearing.

*In plain terms:* Async events vanish when you're throttled or have no dead-letter queue — add a DLQ and alarm on throttles so failures aren't silent.

Likely **throttling** (429 → retries exhausted) or **no DLQ/destination** configured, so failures vanish. Add a DLQ/on-failure destination, check the concurrency limit and reserved concurrency, and alarm on `Throttles`/`DeadLetterErrors`.

[↩ Back to questions](#questions)

#### Q16. Concurrency limits and throttling?

*In plain terms:* There's an account-wide cap (~1000 at once); go over and you get throttled — use reserved concurrency to guarantee capacity or to cap a function so it can't swamp a database.

The account has a default ~1,000 concurrent-execution limit; reserved concurrency **carves a function's share out of that pool**. Exceeding it returns `429 TooManyRequestsException`.
**When to set reserved concurrency:** to guarantee a critical function always has capacity, or to **cap** a function so it can't overwhelm a downstream (or starve other functions).

[↩ Back to questions](#questions)

#### Q17. Lambda vs Fargate vs EC2 — decide.

*In plain terms:* Lambda for short bursty tasks, Fargate for steady containers without managing servers, EC2 when you need full control or it's cheaper always-on.

Lambda = event-driven, scale-to-zero, ≤15 min, no infra. Fargate = containers without managing nodes, long-running, predictable. EC2 = full control, special hardware, or cheapest at steady high load.
**When to use:** Lambda → bursty/short/glue. Fargate → steady containerized services. EC2 → control/stateful/GPU/cost-at-scale.

[↩ Back to questions](#questions)

#### Q18. EventBridge vs SQS vs SNS as a trigger?

*In plain terms:* EventBridge routes/filters events (and cron); SQS buffers and decouples; SNS fans one event out to many — often combined SNS to SQS to Lambda.

EventBridge = routing/filtering events by content + schedules + SaaS sources. SQS = durable buffer + decoupling + backpressure. SNS = pub/sub fan-out to many subscribers.
**When to use:** EventBridge → event routing/filtering/cron. SQS → throttle/buffer/decouple. SNS → fan one event out to many consumers (often SNS→SQS→Lambda).

[↩ Back to questions](#questions)

---

<div align="center">

*📝 Notes compiled as a quick reference for AWS Lambda.*

</div>
