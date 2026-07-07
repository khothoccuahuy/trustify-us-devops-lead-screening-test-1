# DevOps Lead — Take-home Screening Test

Welcome! This test is designed to reflect real day-to-day work for this role: provisioning
serverless infrastructure with Terraform and designing a CI/CD pipeline around it.

**Time budget:** 3–5 hours. There is no need to over-engineer — we care more about clear
reasoning and correct fundamentals than a "perfect" solution.

**You do NOT need to run `terraform apply` against a real AWS account.** We only need to
review your code and design reasoning — no need to incur any AWS cost.

AI tools (Copilot, ChatGPT, Claude, etc.) are allowed. Just be transparent about where you
used them in your PR description — we'll go through the code together in the next round
regardless.

---

## Context

The team is building a simple serverless API. You're asked to provision the infrastructure
using Terraform and design a CI/CD pipeline to deploy it automatically.

---

## Part 1 — Terraform (90–150 min)

Write Terraform to provision:

1. **Lambda function** (Python runtime). The function body can be trivial —
   e.g. `return {"statusCode": 200, "body": "ok"}` — we're not evaluating application logic.
2. **API Gateway** (REST or HTTP API) that triggers the Lambda function.
3. **S3 bucket** for static frontend hosting — versioning enabled, public access blocked
   according to best practice.
4. **IAM role** for the Lambda function following least-privilege (no wildcard
   `AdministratorAccess` or overly broad `*` policies).
5. **Remote state** setup — S3 backend + DynamoDB lock table configuration (code only,
   no need to actually apply it).
6. **VPC + Aurora PostgreSQL Serverless v2**:
   - VPC with at least two private subnets (across separate AZs) for the Aurora cluster,
     plus whatever public/NAT setup you judge necessary for Lambda's outbound access.
   - Aurora Serverless v2 cluster (PostgreSQL-compatible) placed in the private subnets,
     with a DB subnet group and reasonable min/max ACU scaling config.
   - Security groups: Lambda's SG may reach the Aurora SG on port 5432 only — no
     `0.0.0.0/0` ingress on the DB.
   - Attach the Lambda function to the VPC so it can reach the cluster. Actual DB
     connection logic in the Lambda body is **not required** — a trivial handler is fine,
     same as item 1.
   - Credentials: use Secrets Manager or the RDS Data API — do not hardcode DB
     credentials in Terraform or in the Lambda environment variables in plaintext.

**Requirements:**
- Use a **module structure** — do not put everything into a single `main.tf`.
- Separate `variables.tf` and `outputs.tf`.
- Output the API Gateway endpoint URL and the Aurora cluster endpoint.

---

## Part 2 — CI/CD Pipeline Design (45–60 min)

Write a `.github/workflows/deploy.yml` (GitHub Actions) — or `.gitlab-ci.yml` if you prefer
GitLab CI — describing a pipeline with:

1. **Plan stage** — run `terraform plan`, save the plan as an artifact.
2. **Manual approval gate** before applying to production.
3. **Apply stage** — run `terraform apply` using the approved plan.
4. **Rollback strategy** — describe (in a comment or in your README) how you'd roll back
   if the apply step fails.

The pipeline does not need to actually run successfully — it just needs to be logically
sound and well-structured.

---

## Part 3 — Design Document (30–45 min)

Write a short `DESIGN.md` answering:

1. Why did you organize the Terraform modules the way you did?
2. If this needed to scale to **multi-environment (dev/staging/prod) across 3 separate
   AWS accounts**, what would you change in this setup?
3. You already provisioned the VPC + Aurora Serverless v2 setup in Part 1 — walk through
   your networking design decisions (subnet layout, NAT vs VPC endpoints, security group
   rules) and the trade-offs you considered (cost vs. availability, public vs. private
   access, RDS Data API vs. direct DB connections from Lambda).
4. What do you think is the biggest risk in the pipeline you designed, and how would you
   mitigate it?

---

## How to Submit

1. Clone this repository.
2. Create a branch: `git checkout -b submission/<your-name>`
3. Complete the assignment (Terraform + CI/CD + DESIGN.md).
4. Commit in logical, meaningful chunks — no need to squash into a single commit.
   (e.g. "Add Lambda + API Gateway module", "Add VPC + Aurora Serverless v2 module",
   "Add remote state config", "Add GitHub Actions pipeline", "Add design doc")
5. Push your branch and open a Pull Request against `master`.
6. Fill in the PR description using the template below.
7. **Deadline:** 3 days from when you received this test.

### PR Description Template

```markdown
### Approach
(Briefly explain how you structured the Terraform modules and pipeline)

### Assumptions
(Any assumptions you made where the requirements were ambiguous)

### What I'd do differently with more time
(Anything you'd want to improve or expand given more time)

### AI tool usage
(Did you use Copilot/ChatGPT/Claude for any part? No issue either way —
just be transparent)
```

---

Good luck! If anything in this test is unclear, feel free to note your assumption in the
PR description rather than blocking on it.
