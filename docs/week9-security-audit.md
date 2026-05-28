# Week 9 Security Audit — cis410-deploy-sa

**Project:** cis410-seth
**Date:**  05/27/2026
**Auditor:** Seth Richardson

---

## 1. IAM Audit Results

### Before — Week 8 Configuration (over-permissioned)

| Role | Scope | Problem |
|---|---|---|
| roles/run.admin | Project | Overly broad — grants ability to delete services and modify IAM, not just deploy |
| roles/storage.admin | Project | Overly broad — grants access to ALL GCS buckets in the project |
| roles/artifactregistry.writer | Project | Acceptable — scoped to push images only |
| roles/viewer | Project | Acceptable — read-only project metadata |
| roles/iam.serviceAccountUser | Compute SA | Required — needed to act as Compute Engine default SA |

### After — Week 9 Least-Privilege Fix

| Role | Scope | Why Sufficient |
|---|---|---|
| roles/run.developer | Project | Deploy only — cannot delete services or modify IAM |
| roles/storage.admin | tfstate bucket only | Scoped to one bucket — not all storage |
| roles/artifactregistry.writer | Project | Unchanged — push images only |
| roles/viewer | Project | Unchanged — read project metadata |
| roles/iam.serviceAccountUser | Compute SA | Unchanged — required for Cloud Run deployment |

---

## 2. Secret Manager Migration

- **Secret created:** `flask-app-secret`
- **Replication:** automatic
- **Access granted to:** `cis410-deploy-sa` — roles/secretmanager.secretAccessor on this secret only
- **Access granted to:** `PROJECT_NUMBER-compute@developer.gserviceaccount.com` — roles/secretmanager.secretAccessor on this secret only (required for Cloud Run runtime access)
- **Cloud Run update:** APP_SECRET environment variable mounted from Secret Manager at runtime

---

## 3. Monitoring Configuration

- **Log-based alert:** `cis410-flask-app-alert` — fires on severity>=WARNING for cis410-flask-app
- **Notification channel:** sethrichardson@students.highline.edu
- **Billing budget:** `cis410-monthly-budget` — $20 limit, alerts at 50% / 90% / 100%

---

## 4. Reflection

**Q1: Why is roles/run.admin inappropriate for a CI/CD pipeline service account?**

Using roles/run.admin would violate the principle of least privilege. For a CI/CD pipeline service account, you would only need to build and deploy. Granting admin permissions could expose your infrastructure, which would create a significant security risk.

---

**Q2: What is the security difference between storing a secret in GitHub Secrets vs. Google Secret Manager?**

One of the biggest differences between GitHub Secrets and Google Secret Manager is in auditing access. With GitHub secrets, you have basic audit logs that show when a secret is created and updated, but Google Secret Manager shows a more robust log output that details who accessed the secret and when.  This gives a much clearer account of when secrets are used. Another difference is that with Google Secret Manager, you can assign precise permissions to specific Cloud Run services. This makes it so that a specific service can read the secret, but a human developer wouldn’t have access to it. In GitHub Secrets, if a developer has run and modify access to a workflow, they could potentially exfiltrate a GitHub secret. 

---

**Q3: A coworker says "I will clean up IAM permissions after the project launches. For now I need everything to work fast." What is the risk of this approach?**


Fixing permissions later, after the code goes into production, has a few pitfalls. First, you may say I’ll go back and fix it later but that doesn't always happen. Other tasks can get in the way, and fixing permissions gets pushed down the priority list or never ends up happening. Also, stripping down permissions when code is already in production makes the task that much harder. You could take away permissions for small microservices that could lead to production outages. 