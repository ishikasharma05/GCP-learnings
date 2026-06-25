# GCP Learnings

Documenting my hands-on exploration of Google Cloud Platform (GCP) — covering IAM, budget management, and static website hosting using Cloud Storage.

This repo is a continuation of my cloud learning journey, following hands-on work with AWS (IAM, EC2, S3, MFA, VPC) in my [DevOps_Learnings](https://github.com/ishikasharma05/DevOps_Learnings) repo. After getting comfortable with AWS, I wanted to see how the same core cloud concepts — identity, storage, cost control — are implemented on a different platform.

I also attempted to set up a free-tier account on Microsoft Azure to compare all three major providers, but ran into payment verification issues during signup. GCP signup, by contrast, went smoothly.

## What's Inside

📄 [`gcp-iam-budgets-bucket-hosting.md`](./gcp-iam-budgets-bucket-hosting.md) — covers three things in one writeup:

1. **IAM** — understanding GCP's principal → role → resource model, and the difference between Basic, Predefined, and Custom roles.
2. **Budgets & Alerts** — setting a project budget with percentage-based threshold alerts (50% / 90% / 100%) to track spend proactively.
3. **Static Website Hosting via Cloud Storage** — deploying my portfolio to a Cloud Storage bucket, GCP's equivalent of AWS S3, including how public access is granted through IAM permissions rather than a separate bucket policy.

The doc also includes a direct **AWS vs GCP comparison table** for the overlapping concepts, and a section on the one real problem I hit — figuring out how to fully delete a GCP *project* (not just the resources inside it) via **IAM & Admin → Manage Resources**.

## Why This Repo Exists

I'm building toward a DevOps/cloud career, and I think understanding *why* a concept exists matters more than memorizing one platform's specific steps. Comparing AWS and GCP side by side — same problems, different implementations — has made both platforms easier to reason about, not harder.

This is a learning log, not a finished product. Some sections note where my understanding might be incomplete, and I'll keep updating this as I go deeper into GCP or revisit it.

## Related

- [DevOps_Learnings](https://github.com/ishikasharma05/DevOps_Learnings) — AWS hands-on work (IAM, EC2, S3 hosting, MFA, VPC)
- [LinkedIn](https://linkedin.com/in/ishika-sharma-connect) — posts on what I'm learning, in public

---

**Status:** 🚧 Actively learning — this repo will grow as I explore more of GCP.
