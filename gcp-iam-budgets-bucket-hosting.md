# Exploring Google Cloud Platform (GCP) — IAM, Budgets & Static Hosting

After working with AWS (IAM, EC2, S3, MFA, VPC), I wanted to see how another major cloud platform handles the same core ideas — so I created a free-tier account on **Google Cloud Platform (GCP)** and worked through IAM, budget management, and static website hosting using Cloud Storage.

I also attempted to create a free-tier account on **Microsoft Azure** to compare all three major providers, but ran into payment verification issues during signup and wasn't able to complete it. GCP signup, by contrast, was straightforward.

> **Note:** The portfolio is not currently live on GCP — the deployment was done as a hands-on learning exercise during a couple of sessions, and was intentionally not left running to avoid exposing personal credentials/account details tied to a live resource.

---

## 1. IAM (Identity and Access Management)

Same core concept as AWS IAM — controlling **who** can access GCP resources and **what** they're allowed to do — but with GCP's own terminology and structure.

What I worked through:
- Explored GCP's **IAM & Admin** console.
- Understood the **principal → role → resource** model: a principal (a user, service account, or group) is granted a **role** (a bundle of permissions) on a specific **resource** (project, bucket, etc.).
- Practiced adding users and assigning them predefined roles (e.g. Viewer, Editor) to understand the permission boundaries between them, rather than granting broad access by default.

📸 ***Screenshot: IAM & Admin console showing a user with an assigned role***

**Key difference from AWS IAM (as I understood it):** GCP's roles are organized in three tiers — **Basic roles** (Owner/Editor/Viewer, broad and easy but not least-privilege), **Predefined roles** (scoped to a specific service, like "Storage Object Viewer"), and **Custom roles** (build your own permission set). AWS's policy-document approach (JSON statements) felt more granular by default; GCP's predefined roles felt faster to apply but less precise until you go custom.

---

## 2. Setting a Budget

Before doing any hands-on resource creation, I set up a **budget alert** in **Billing → Budgets & alerts** — partly out of caution on a free account, and partly because the feature itself turned out to be genuinely useful to learn.

Steps followed:
1. Went to **Billing → Budgets & alerts → Create Budget**.
2. Selected the scope (which project/billing account the budget applies to).
3. Set a target budget amount.
4. Configured **threshold alerts as percentages** of that budget (e.g. get notified at 50%, 90%, and 100% of the set amount) instead of only a single fixed-amount trigger.
5. Confirmed and created the budget.

📸 ***Screenshot: Budget creation screen showing percentage-based thresholds***

**What I liked about this:** the percentage-based threshold method makes the budget flexible — instead of one rigid dollar trigger, you get a clear escalating signal (50% → "keep an eye on it," 90% → "something's about to go wrong") as actual usage approaches the limit. For someone new to cloud billing, this is a much safer way to avoid surprise charges than checking the billing dashboard manually.

---

## 3. Hosting a Static Website Using Cloud Storage (GCP's equivalent of S3)

GCP's equivalent of AWS S3 is **Cloud Storage**, and within it, files are organized into **buckets** — the same underlying concept as S3, different platform.

Steps followed:
1. Went to **Cloud Storage → Buckets → Create**.
2. Named the bucket and selected a region/storage class.
3. Uploaded the portfolio files (HTML/CSS/JS) into the bucket.
4. Configured public access so the files could be served to visitors — conceptually the same idea as an S3 bucket policy, but handled through GCP's **IAM permissions on the bucket** (granting the `Storage Object Viewer` role to `allUsers`) rather than a separate bucket-policy JSON document like AWS uses.
5. Verified the uploaded files were reachable via the public object URLs.

📸 ***Screenshot: Cloud Storage bucket showing uploaded portfolio files***
📸 ***Screenshot: Public access / permissions configuration on the bucket***

**What this clarified:** S3 and Cloud Storage solve the identical problem (cheap, serverless static file hosting) but expose the "make it public" step differently — AWS separates "block public access" + a policy document, while GCP folds public access directly into the IAM permissions model already used for everything else on the platform. Functionally similar outcome, different mental model to learn.

---

## 4. The One Real Problem I Hit: Deleting the Project

After finishing the hands-on portion, I deleted the uploaded files from the bucket quickly — but then couldn't immediately figure out how to delete the **project** itself (GCP organizes all resources under a "Project," similar in spirit to an AWS account/region scope, but it's a distinct concept of its own).

**What I learned:** deleting individual resources (files, buckets) doesn't remove the project they live in. To fully tear down a GCP project, you have to go through:

**IAM & Admin → Manage Resources** → select the project → **Delete**

This shuts down billing and resource creation for that project entirely (with a recovery grace period before permanent deletion), rather than relying on deleting things piece by piece.

📸 ***Screenshot: Manage Resources page showing project deletion option***

This was a good reminder that "deleting your stuff" and "deleting the project" are two separate actions on GCP — worth knowing before doing any cost-sensitive experimentation.

---

## Summary: GCP vs AWS, First Impressions

| Concept | AWS | GCP |
|---|---|---|
| Identity & access | IAM (JSON policy documents) | IAM (role tiers: Basic/Predefined/Custom) |
| Object storage | S3 | Cloud Storage |
| Making storage public | Bucket Policy (separate JSON) | IAM permission on bucket (`allUsers` role) |
| Cost control | Budgets (AWS Budgets) | Budgets & alerts (percentage-based thresholds) |
| Full teardown | Delete resources individually | Must explicitly delete the **Project** via Manage Resources |

---

## Stack / Learning Context

This was an exploratory session to compare cloud platforms after building hands-on familiarity with AWS (IAM, EC2, S3, MFA, VPC). The goal wasn't to master GCP, but to confirm which concepts are universal across cloud providers (IAM, storage, budgeting) versus which are platform-specific implementation details — useful context before committing deeper to any one ecosystem.
