# Section 35: Disaster Recovery

## The idea

Disaster recovery (DR) answers one question: *when a whole region catches fire, how fast are you back, and how much data did you lose?* Two numbers define everything:

- **RPO — Recovery Point Objective**: maximum acceptable **data loss**. How far back does the tape rewind? If you back up nightly, your RPO is up to 24 hours of lost work.
- **RTO — Recovery Time Objective**: maximum acceptable **downtime**. How long until you're serving customers again?

```
 last backup          DISASTER              back online
──────●───────────────────✖───────────────────────●──────▶ time
      ◀───── RPO ─────────▶◀───────── RTO ────────▶
       (data you lost)        (time you were down)
```

Think of DR strategies as **kinds of spare tires**. Backup & Restore is *no spare — call a tow truck*. Pilot Light is a *space-saver donut in the trunk*. Warm Standby is a *full spare, already inflated*. Active-Active is *a second identical car driving alongside you*. Better spares cost more to carry around; the exam always asks for the **cheapest one that meets the stated RPO/RTO** — the same eliminate-then-cheapest logic you used for S3 storage classes.

## The four strategies

| Strategy | RTO | RPO | Cost | What runs in the DR region |
|---|---|---|---|---|
| **Backup & Restore** | Hours | Hours | $ cheapest | Nothing — backups in S3, rebuild on disaster |
| **Pilot Light** | Tens of minutes | Minutes–seconds | $$ | Core only: **DB continuously replicated, servers off/minimal** |
| **Warm Standby** | Minutes | Seconds | $$$ | **Scaled-DOWN full copy always running** → scale UP on disaster |
| **Multi-Site Active-Active** | ~Zero | ~Zero | $$$$ | Full capacity in both regions, serving live traffic |

Signal phrases:

- "**Minimal DR footprint**, database continuously replicated, compute launched only on disaster" → **Pilot Light** (the pilot flame is lit; the furnace is off).
- "A smaller version of the full environment is **always running**" → **Warm Standby**.
- "**Zero downtime**, both regions active" → **Multi-Site Active-Active**.
- "Lowest cost, RTO of hours is fine" → **Backup & Restore**.

**THE trap: don't buy a better spare tire than the question asks for.** If RTO is "a few hours," Active-Active is wrong even though it's "better" — it fails the cost qualifier.

## The supporting cast

| Service | Role in DR |
|---|---|
| **Aurora Global Database** | Cross-region replication, **RPO ~1 second, RTO < 1 minute** |
| **DynamoDB Global Tables** | Multi-region, multi-active NoSQL — writes anywhere. Default is **eventually consistent**; **multi-Region strong consistency (MRSC, GA Jun 2025)** gives **zero RPO** (needs 3 Regions: 3 replicas, or 2 replicas + a witness) |
| **Aurora DSQL** | Serverless distributed SQL, **active-active multi-Region** with strong consistency (two peered Regions + a log-only witness Region), **99.999%** multi-Region availability |
| **S3 Cross-Region Replication (CRR)** | Objects copied to a bucket in another region |
| **AWS Backup** | **Centralized, cross-service backup plans** (EBS, RDS, DynamoDB, EFS…) + **Vault Lock** for immutable backups. **Logically air-gapped vaults** (shareable via RAM) plus **multi-party approval** let a separate recovery account restore even if the original account is compromised or locked out — the ransomware answer |
| **Application Recovery Controller (ARC) Region switch** | Orchestrates a **full multi-Region failover plan** (EC2/ASG, Aurora Global, DNS via routing controls) manually or triggered by CloudWatch alarms. The older ARC *readiness checks* are closed to new customers |
| **Elastic Disaster Recovery (DRS)** | **Continuous block-level replication of servers** (on-prem or EC2) into AWS — cheap DR for lifted servers, RPO seconds / RTO minutes |
| **Route 53 failover routing** | Health check fails → DNS flips traffic to the DR region |
| **DMS** | Database Migration Service — also does ongoing replication between databases |

## How to answer any DR question

1. Find the stated **RPO/RTO numbers** (or words like "zero downtime").
2. **Eliminate** strategies that can't meet them.
3. Pick the **cheapest survivor**.

## Question patterns

> *"RTO of 10 minutes, RPO of seconds, keep costs reasonable"* → **Warm Standby** (Active-Active also works but costs more; Pilot Light is too slow)

> *"DR needed at the LOWEST cost; hours of downtime acceptable"* → **Backup & Restore** (cheapest strategy that meets a relaxed RTO)

> *"Application must survive a region failure with zero downtime and zero data loss"* → **Multi-Site Active-Active** (only ~zero RPO/RTO option)

> *"Database continuously replicated to DR region; EC2 launched only during a disaster"* → **Pilot Light** (core data live, compute off)

> *"Centrally manage and automate backups of EBS, RDS, and DynamoDB across the organization"* → **AWS Backup** (one service, backup plans + vault lock)

> *"Cost-effective DR for 200 on-premises servers into AWS"* → **Elastic Disaster Recovery (DRS)** (continuous block replication, pay tiny until failover)

> *"Global relational database with cross-region RPO of ~1 second"* → **Aurora Global Database** (RPO 1s, RTO under a minute)

> *"Automatically route users to the standby region when the primary fails"* → **Route 53 failover routing with health checks**

> *"Multi-Region NoSQL with zero data loss; reads must always return the latest write"* → **DynamoDB global tables with multi-Region strong consistency** (default global tables are eventually consistent)

> *"Backups must survive a compromised or locked-out account (ransomware)"* → **AWS Backup logically air-gapped vault + multi-party approval**

> *"Orchestrate and automate failover of a whole application between Regions"* → **ARC Region switch**

## Pocket card

| Keyword | Answer |
|---|---|
| Max data loss | RPO |
| Max downtime | RTO |
| Cheapest, RTO hours | Backup & Restore |
| DB replicated, servers off | Pilot Light |
| Scaled-down copy running | Warm Standby |
| Zero RPO/RTO, both live | Multi-Site Active-Active |
| RPO 1s / RTO <1 min relational | Aurora Global Database |
| Multi-region multi-active NoSQL | DynamoDB Global Tables |
| Multi-Region NoSQL, zero RPO | Global tables with MRSC |
| Active-active multi-Region SQL | Aurora DSQL |
| Ransomware-proof / separate-account restore | Backup air-gapped vault + multi-party approval |
| Automated Region failover plan | ARC Region switch |
| Centralized cross-service backups | AWS Backup (+ Vault Lock) |
| Replicate on-prem servers for DR | Elastic Disaster Recovery (DRS) |
| DNS failover to DR region | Route 53 failover routing |

You now hold the full toolbox — time to sharpen it against the exam's favorite tricks: the classic traps and must-know combos.
