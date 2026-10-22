# Section 28: CloudFormation

## The idea

Imagine you bake a spectacular cake once by improvising — a pinch of this, a swirl of that. Now bake an identical one next week. You can't; you never wrote it down. Clicking around the AWS console is improvised baking. **CloudFormation is the written recipe**: you describe your entire infrastructure — VPCs, EC2, RDS, IAM roles, everything — in a **YAML or JSON template**, and AWS "cooks" it for you. A running instance of a recipe is called a **stack**.

This is **Infrastructure as Code (IaC)**: templates live in version control, deploy identically to dev/test/prod or to a new region, and delete cleanly in one command. If creation fails partway, CloudFormation **rolls back automatically by default** — no half-baked cakes left in the oven.

## Template anatomy (know the section names)

| Section | Job |
|---|---|
| **Parameters** | Inputs at deploy time (instance size, env name) |
| **Mappings** | Lookup tables, e.g. the right AMI ID per region |
| **Conditions** | "Only create this in prod" logic |
| **Resources** | **The only REQUIRED section** — what actually gets built |
| **Outputs** | Values to surface after creation; **Export** them and another stack reads them with **Fn::ImportValue** (cross-stack references) |

Intrinsic function one-liners: **!Ref** = reference a parameter/resource, **!GetAtt** = grab an attribute of a resource (like an instance's DNS name), **!Sub** = substitute variables into a string.

## Features → scenarios (this is the exam meat)

- **Change Sets** — a **preview** of exactly what an update would create/modify/delete *before* you apply it. "Team wants to see what a stack update will do first" → Change Set.
- **Stack Policies** — protect specific resources within a stack **from being modified by stack updates**.
- **Drift Detection** — answers "did someone hand-edit our resources in the console?" Compares live reality to the template. "Detect manual changes" → Drift Detection.
- **StackSets** — deploy **one template to MANY accounts and regions** in one operation. "Roll out a security baseline across the whole organization" → StackSets.
- **Nested Stacks** — stacks inside stacks; package common patterns (a standard VPC, say) as reusable components.
- **DeletionPolicy** — per-resource instruction for stack deletion: **Retain** (keep the resource) or **Snapshot** (snapshot it first — works for RDS, EBS, etc.). "Keep the database when the stack is deleted" → DeletionPolicy: Retain.

THE trap: manual console edits to stack-managed resources don't update the template — they create *drift*, and the next stack update can stomp them. The cure is Drift Detection (to find it) and discipline (to stop it).

**CDK in one line:** the Cloud Development Kit lets you write infrastructure in **Python/TypeScript/Java**, which *compiles down to CloudFormation templates*. "Developers want to define infrastructure in a real programming language with loops and logic" → **CDK**.

### Newer CloudFormation features (2026)

- **Pre-deployment validation (Jun 2026)** — validation now runs automatically on **Create Stack and Update Stack**, not only when you build a change set. It adds warning-level checks for service quotas, AWS Config recorder conflicts and ECR repository deletion readiness. The "catch problems *before* resources are touched" idea, now on every operation.
- **Express mode (Jun 2026)** — finishes a stack operation once resource configuration is applied, **without waiting for stabilization checks**, for deployments up to ~4x faster at no extra cost. **Rollback is disabled by default** in express mode, so it suits iterative dev loops, not cautious production updates.
- **Fn::GetStackOutput (May 2026)** — reads another stack's outputs **across accounts and Regions** (via an IAM role). The classic **Export / ImportValue** only works within the **same account and Region**.
- **Instance Refresh update policy (Jul 2026)** — when an update requires replacing instances, CloudFormation starts an EC2 Auto Scaling Instance Refresh automatically; rollback goes through normal stack rollback.

**Elastic Beanstalk contrast:** Beanstalk is web-app-shaped automation — hand it code, it builds the standard web stack (and uses CloudFormation underneath). CloudFormation itself can build **any** infrastructure in any shape. Beanstalk = specific meal deal; CloudFormation = whole grocery store.

## Question patterns

> *"Team must review what changes a template update will make before applying it"* → **Change Sets** (preview = change set, always)

> *"Deploy standardized security/compliance resources to all accounts in an organization, across regions"* → **CloudFormation StackSets** (one template → many accounts/regions)

> *"Ensure the RDS database is not deleted when the stack is torn down"* → **DeletionPolicy: Retain** (or Snapshot if they want a final copy)

> *"Determine whether anyone modified stack resources manually via the console"* → **Drift Detection** (template vs. reality diff)

> *"Developers prefer defining infrastructure in TypeScript/Python rather than YAML"* → **AWS CDK** (code that compiles to CloudFormation)

> *"Deploy identical environments repeatedly across regions with no manual steps"* → **CloudFormation templates** (the core IaC pitch)

> *"Prevent accidental updates to a critical resource during stack updates"* → **Stack Policy** (update-protection inside a stack)

> *"Reuse a common VPC pattern across many templates"* → **Nested Stacks** (componentize the recipe)

> *"A stack in one Region/account needs another stack's output"* → **Fn::GetStackOutput** (ImportValue is same account + Region only)

## Pocket card

| Keyword | Answer |
|---|---|
| Preview changes before update | Change Sets |
| Many accounts + regions, one template | StackSets |
| Keep data on stack delete | DeletionPolicy Retain / Snapshot |
| Detect manual console edits | Drift Detection |
| Infra in Python/TypeScript | CDK |
| Protect resource during updates | Stack Policy |
| Reusable template components | Nested Stacks |
| Share values between stacks | Outputs + Export / ImportValue |
| Cross-account / cross-Region stack outputs | Fn::GetStackOutput |
| Faster deploys, skip stabilization, no default rollback | Express mode |
| Only required template section | Resources |
| Failed creation | Automatic rollback (default) |

You now know how to stamp out infrastructure across many accounts — next comes the question of who governs all those accounts, which is Organizations and Control Tower territory.
