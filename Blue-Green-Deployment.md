# Blue-Green Deployment on Google Cloud Platform

A hands-on demonstration of the blue-green deployment strategy on GCP using two Compute Engine VMs behind a Global External Application Load Balancer, with zero-downtime traffic cutover between application versions. Built as a companion piece to the [AWS version](./README.md) of this project for a direct cloud-provider comparison.

## Overview

- **Blue instance** runs "Version 1" of the app
- **Green instance** runs "Version 2" of the app
- A **Global External Application Load Balancer** routes traffic through backend services
- Switching the backend service behind the load balancer's routing rule cuts traffic from blue to green (or back) with no VM restarts

## Architecture

```
                         ┌─────────────┐
                         │   Client    │
                         └──────┬──────┘
                                │
                    ┌───────────▼───────────┐
                    │  Global External ALB  │
                    │   (Routing Rules)     │
                    └──────┬─────────┬──────┘
                            │         │
                  ┌─────────▼──┐  ┌───▼──────────┐
                  │blue-backend│  │green-backend │
                  └─────────┬──┘  └───┬──────────┘
                            │         │
                     ┌──────▼──┐  ┌───▼─────┐
                     │ blue-ig │  │green-ig │
                     └──────┬──┘  └───┬─────┘
                            │         │
                     ┌──────▼──┐  ┌───▼─────┐
                     │  blue   │  │  green  │
                     │instance │  │instance │
                     │  (v1)   │  │  (v2)   │
                     └─────────┘  └─────────┘
```

## Prerequisites

- GCP account/project with billing linked
- Compute Engine API enabled

## Tech Stack

- **GCP Compute Engine** — Ubuntu 22.04 LTS, e2-micro
- **GCP Instance Groups** (unmanaged)
- **GCP Backend Services + Health Checks**
- **GCP Global External Application Load Balancer**
- **Apache2** — web server to serve each version

---

## Steps

### Part 1: Project Setup

1. Create a new GCP project, link a billing account
2. Enable the **Compute Engine API** (this provisions the default service account — do this *before* creating any VM to avoid a `serviceAccount not found` error)
3. Wait 1-2 minutes after enabling before creating VMs

### Part 2: Create the Blue VM

4. **Compute Engine** → **VM instances** → **Create Instance**
5. Name: `blue-instance`
6. Region/Zone: e.g. `asia-south1`
7. Machine type: `e2-micro` (free tier eligible)
8. Boot disk: **Ubuntu 22.04 LTS**
9. Firewall: check **Allow HTTP traffic**
10. Create, wait for the green checkmark, note the **External IP**

### Set Up the App on Blue (Version 1)

11. Click **SSH** next to the instance to open a browser-based terminal
12. Install Apache and a text editor:
    ```bash
    sudo apt update -y
    sudo apt install -y nano apache2
    sudo systemctl start apache2
    sudo systemctl enable apache2
    sudo systemctl status apache2
    ```
13. Edit the page:
    ```bash
    sudo nano /var/www/html/index.html
    ```
    ```html
    <html>
    <head><title>Blue-Green Demo - GCP</title></head>
    <body style="background-color:#cce5ff; text-align:center; padding-top:100px;">
      <h1>Version 1 - BLUE</h1>
      <p>Serving from blue-instance (GCP)</p>
    </body>
    </html>
    ```
14. Test at `http://<blue-external-ip>` — confirm the blue page loads.

### Part 3: Create the Green VM

15. Repeat steps 4-10, name it `green-instance`, same region/zone/machine type/OS.
16. SSH in, install Apache + nano (same as step 12).
17. Edit the page:
    ```html
    <html>
    <head><title>Blue-Green Demo - GCP</title></head>
    <body style="background-color:#d4edda; text-align:center; padding-top:100px;">
      <h1>Version 2 - GREEN</h1>
      <p>Serving from green-instance (GCP)</p>
    </body>
    </html>
    ```
18. Test at `http://<green-external-ip>` — confirm the green page loads.

### Part 4: Create Instance Groups

19. **Compute Engine** → **Instance groups** → **Create instance group**
20. Choose **New unmanaged instance group**
21. Name: `blue-ig`, same region/zone as `blue-instance`, add `blue-instance` → **Create**
22. Repeat for `green-ig`, adding `green-instance`

### Part 5: Create Health Check and Backend Services

> **Note:** Health checks are created under **Compute Engine → Health checks**, not under Network Services → Load balancing. You can also create one inline while setting up a backend service, which is the path used below.

23. **Compute Engine** → **Backend services** → **Create backend service**
24. Name: `blue-backend`, Backend type: **Instance group**
25. Add `blue-ig` as a backend, port 80
26. Health check: click **Create a health check**
    - Name: `http-health-check`, Protocol: HTTP, Port: 80, Path: `/`
    - Scope: **Global** (matches the global load balancer being built)
    - **Create**
27. **Create** the backend service
28. Repeat steps 23-27 for `green-backend`, using `green-ig` and reusing the same `http-health-check`

### Part 6: Create the Load Balancer

29. **Network Services** → **Load balancing** → **Create load balancer**
30. Type: **Application Load Balancer**
31. Public facing: **Public facing (external)**
32. Deployment: **Global workloads**
33. Generation: **Global external Application Load Balancer**
34. **Configure**
35. Name: `bluegreen-lb`
36. **Frontend configuration**: HTTP, Port 80, name `bluegreen-frontend`, Ephemeral IP → **Done**
37. **Backend configuration**: select existing backend service `blue-backend`
38. **Routing rules**: leave default (routes `/*` to `blue-backend`)
39. Review → **Create**
40. Wait 3-5 minutes for full provisioning, then note the **frontend IP**

### Part 7: Test Baseline and Perform the Cutover

41. Open the frontend IP in a browser — should show **Version 1 - BLUE**

   <img width="724" height="287" alt="image" src="https://github.com/user-attachments/assets/1912fb63-c24c-4bca-a393-d98a5c8872e8" />


42. Go to the load balancer → **Edit** → **Backend configuration** → change the default backend from `blue-backend` to `green-backend` → **Update**
43. Wait ~30-60 seconds for the change to propagate
44. Refresh the frontend IP — should now show **Version 2 - GREEN**

    <img width="603" height="224" alt="image" src="https://github.com/user-attachments/assets/2ccb6331-3065-4af3-9438-7f596126159f" />


45. To confirm rollback, switch back to `blue-backend`, refresh, confirm blue returns

    <img width="320" height="160" alt="image" src="https://github.com/user-attachments/assets/08a34ebe-d054-44c8-90f2-3296748a6fcf" />


46. Delete the Load Balancer (removes the forwarding rule + proxy)
47. Delete both backend services
48. Delete the health check
49. Delete both instance groups
50. Delete both VM instances

**Cost note**: GCP's free tier covers 1 `e2-micro` instance/month in select US regions — `asia-south1` may not qualify, so check billing. The Load Balancer is not free (billed per forwarding rule + data processed) — clean up promptly.

---

## Troubleshooting Encountered

Real issues hit while building this, kept here for reference (and because working through cloud provider quirks is itself worth documenting):

| Issue | Cause | Fix |
|---|---|---|
| `serviceAccount not found` on VM creation | Default Compute Engine service account missing/not yet provisioned | Recreated the project; enabled Compute Engine API *before* creating any VM |
| `sudo: I'm afraid I can't do that` | Account had `Compute OS Login` but not `Compute OS Admin Login` IAM role | Added `roles/compute.osAdminLogin` in IAM, reconnected SSH session |
| `nano`/`vim: command not found` | Ubuntu image doesn't ship an editor by default | `sudo apt install -y nano` |
| Health check not visible in Load Balancing section | Created inline during backend service setup, not as a standalone resource under that menu | Health checks live under **Compute Engine → Health checks**; recreate there or inline while creating the backend service |

## Key Takeaway

The blue-green pattern is provider-agnostic — the concept (route via an LB, flip the backend, keep the old version warm for rollback) is identical on AWS and GCP. What differs is the object model: AWS uses Target Groups directly attached to a Load Balancer listener, while GCP requires an extra layer (Instance Group → Backend Service → URL Map) before the Load Balancer is even involved.

## Notes

- Ubuntu VMs on GCP use your Google account's OS Login username, not a fixed `ubuntu` user like AWS
- `ufw` is inactive by default — GCP's firewall rules control access at the network level instead
