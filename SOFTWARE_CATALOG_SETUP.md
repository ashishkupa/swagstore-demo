# Datadog Software Catalog Setup for Swagstore

## Overview
This guide walks through pushing your changes to GitHub and setting up Datadog Software Catalog with ownership integration.

---

## Part 1: Push Changes to GitHub

### Option A: Using GitHub Token (Recommended)

1. **Generate GitHub Personal Access Token (PAT)**
   - Go to GitHub → Settings → Developer settings → Personal access tokens
   - Click "Generate new token (classic)"
   - Give it a name: "swagstore-demo"
   - Select scopes:
     - ✅ `repo` (full control of private repositories)
     - ✅ `workflow` (if using GitHub Actions)
   - Generate and copy the token

2. **Configure Git to Use Token**
   ```bash
   cd /home/ashishk/Documents/mylab/swagstore-demo
   
   # Replace YOUR_TOKEN with your actual token
   git remote set-url origin https://YOUR_TOKEN@github.com/ashishkupa/swagstore-demo.git
   
   # Verify
   git remote -v
   ```

3. **Push Your Commits**
   ```bash
   git push origin main
   ```

### Option B: Using SSH

If you prefer SSH (more secure long-term):

1. **Generate SSH Key** (if you don't have one)
   ```bash
   ssh-keygen -t ed25519 -C "your-email@company.com"
   ```

2. **Add to GitHub**
   - Go to GitHub → Settings → SSH and GPG keys
   - Click "New SSH key"
   - Paste your public key from `~/.ssh/id_ed25519.pub`

3. **Configure Git**
   ```bash
   git remote set-url origin git@github.com:ashishkupa/swagstore-demo.git
   git push origin main
   ```

---

## Part 2: Verify Files on GitHub

After pushing, verify these files are in your repo:

```
✅ CODEOWNERS                              (root level)
✅ src/adservice/service.datadog.yaml
✅ src/cartservice/service.datadog.yaml
✅ src/checkoutservice/service.datadog.yaml
✅ src/currencyservice/service.datadog.yaml
✅ src/emailservice/service.datadog.yaml
✅ src/frontend/service.datadog.yaml
✅ src/paymentservice/service.datadog.yaml
✅ src/productcatalogservice/service.datadog.yaml
✅ src/recommendationservice/service.datadog.yaml
✅ src/shippingservice/service.datadog.yaml
✅ src/shoppingassistantservice/service.datadog.yaml
```

Visit: https://github.com/ashishkupa/swagstore-demo/blob/main/CODEOWNERS

---

## Part 3: Setup Datadog Software Catalog

### Step 1: Connect GitHub Integration to Datadog

1. **Go to Datadog Dashboard**
   - Navigate to: **Integrations → Catalog → GitHub**

2. **Install GitHub Integration**
   - Click "Install Integration" or "Connect New Org"
   - You'll be redirected to GitHub
   - Authorize Datadog to access your repositories
   - Select repositories or allow all

3. **GitHub Permissions Requested**
   - Read access to repositories
   - Read access to CODEOWNERS file
   - Read access to metadata

4. **Confirm Connection in Datadog**
   - Go back to Datadog
   - Should show: "GitHub integration installed successfully"

### Step 2: Register Services in Software Catalog

#### Manual Method (If auto-discovery doesn't work)

1. **Go to Software Catalog**
   - Datadog → Service Catalog

2. **Click "Add Service"**

3. **Fill in Service Details**
   ```
   Service Name: cartservice
   
   Source: GitHub
   
   Repository: ashishkupa/swagstore-demo
   
   Service Definition File: src/cartservice/service.datadog.yaml
   
   Framework: gRPC
   
   Language: dotnet
   ```

4. **Repeat for all 11 services**

#### Auto-Discovery Method (Recommended)

If GitHub integration is set up correctly, Datadog will automatically:

1. Scan your repos for `service.datadog.yaml` files
2. Discover ownership from `CODEOWNERS`
3. Register services automatically
4. May take 5-15 minutes

**To trigger manual sync:**
1. Go to Integrations → GitHub
2. Look for "Sync" button
3. Click to force rescan

### Step 3: Verify Services in Catalog

1. **Go to Service Catalog**
   - Datadog → Service Catalog

2. **You should see all 12 services**
   ```
   ✅ adservice           (Owner: ads-team)
   ✅ cartservice         (Owner: dotnet-team)
   ✅ checkoutservice     (Owner: backend-team)
   ✅ currencyservice     (Owner: backend-team)
   ✅ emailservice        (Owner: backend-team)
   ✅ frontend            (Owner: frontend-team)
   ✅ paymentservice      (Owner: fintech-team)
   ✅ productcatalogservice (Owner: backend-team)
   ✅ recommendationservice (Owner: data-science-team)
   ✅ shippingservice     (Owner: backend-team)
   ✅ shoppingassistantservice (Owner: data-science-team)
   ```

3. **Click on Any Service**
   - Should show:
     - ✅ Service name
     - ✅ Description
     - ✅ Team/Owner
     - ✅ GitHub link
     - ✅ Links (Architecture, Runbook, etc.)
     - ✅ Service tier
     - ✅ Type (backend, frontend)
     - ✅ Language

### Step 4: Test Incident Routing

1. **Create a Test Monitor**
   ```
   Metric: system.cpu.user
   Alert: Always
   Notification: @cartservice-owner
   ```

2. **Trigger Alert**
   - When alert fires, check if:
     - ✅ Routed to dotnet-team (from CODEOWNERS)
     - ✅ Includes service metadata
     - ✅ Links to GitHub/runbook

### Step 5: View Service Dependencies

1. **Go to Service Map**
   - Datadog → Service Catalog → Service Map (or Infrastructure → Service Map)

2. **You should see:**
   - All 12 services as nodes
   - Dependencies drawn between them
   - Latency on each connection
   - Error rates
   - Owner labels on each service

---

## Part 4: Configure Service Catalog for Interviews

### Create Custom Views

#### View 1: By Team

1. **Go to Service Catalog**
2. **Add Filter: Team**
3. **Select: backend-team**
   - Shows: cartservice, checkoutservice, currencyservice, etc.

#### View 2: By Service Tier

1. **Add Filter: Service Tier**
2. **Select: critical**
   - Shows: cartservice, checkoutservice, frontend, paymentservice, productcatalogservice

#### View 3: By Language

1. **Add Filter: Language**
2. **Select: Go**
   - Shows: frontend, checkoutservice, shippingservice, productcatalogservice

---

## Part 5: Service Catalog Features for Your Interview

### Feature 1: Ownership Impact
**Show this scenario:**

1. Alert fires on paymentservice
2. Datadog automatically looks up CODEOWNERS
3. Routes to fintech-team
4. Shows:
   - Team members
   - On-call engineer
   - Recent deployments
   - Git commits

**Message**: "One source of truth for ownership - GitHub, Datadog, Slack all in sync"

### Feature 2: Service Dependencies
**Show this scenario:**

1. Frontend is slow
2. Hover over frontend → cartservice connection
3. See: "100ms latency"
4. Click → Shows trace
5. See: cartservice talking to Redis
6. Trace shows N+1 query issue

**Message**: "Ownership + observability = faster incident resolution"

### Feature 3: Compliance & Audit
**Show this scenario:**

1. CISO asks: "Who owns paymentservice?"
2. Go to Service Catalog
3. Click paymentservice
4. Shows:
   - Team: fintech-team
   - PCI-compliant: true
   - Last deployment: 2 days ago by alice@company.com
   - Links to compliance documentation

**Message**: "Service Catalog is the source of truth for compliance"

---

## Troubleshooting

### Problem: Services not appearing in catalog

**Solution 1: Check file location**
- Ensure `service.datadog.yaml` is at repo root or in service directory
- File must be valid YAML

**Solution 2: Re-sync GitHub**
- Go to Integrations → GitHub
- Click "Sync" button manually
- Wait 5-10 minutes

**Solution 3: Check GitHub permissions**
- Ensure Datadog GitHub app has read access
- Visit GitHub → Settings → Applications → Authorized OAuth Apps
- Check Datadog has access

### Problem: CODEOWNERS not recognized

**Solution 1: Verify format**
```bash
# Make sure file is named exactly: CODEOWNERS (case-sensitive)
# Must be in root directory
cd /home/ashishk/Documents/mylab/swagstore-demo
ls -la CODEOWNERS
```

**Solution 2: Check syntax**
- CODEOWNERS must follow GitHub syntax
- Each line: `path @owner`
- Test: Create a PR and see if code owners are requested

### Problem: Services show but ownership is wrong

**Solution**: Clear cache and resync
```
1. Go to Integrations → GitHub
2. Disconnect and reconnect
3. Wait 10 minutes for full rescan
```

---

## What You'll Have After Setup

| Component | Status |
|-----------|--------|
| CODEOWNERS file | ✅ Created & committed |
| service.datadog.yaml (all 11 services) | ✅ Created & committed |
| GitHub integration in Datadog | ✅ To be configured |
| Services in Software Catalog | ✅ Will auto-populate |
| Team ownership mapped | ✅ Will show automatically |
| Incident routing configured | ✅ Optional but powerful |

---

## Next Steps After Setup

1. **Create Demo Alerts**
   - One for each team
   - Show automatic routing to code owners

2. **Setup Slack Integration**
   - Alerts mention @team-name
   - Links include service catalog

3. **Create Team Dashboards**
   - Developer dashboard (by team)
   - SRE dashboard (by service tier)
   - CISO dashboard (by compliance tags)

---

## Quick Reference: Git Push Commands

```bash
# If using token
git remote set-url origin https://YOUR_TOKEN@github.com/ashishkupa/swagstore-demo.git
git push origin main

# If using SSH
git remote set-url origin git@github.com:ashishkupa/swagstore-demo.git
git push origin main

# Verify
git log --oneline -3
git remote -v
```

---

## Interview Demo: Software Catalog Flow

```
Interviewer: "How do you handle on-call and ownership?"

1. Show Datadog Service Catalog
2. Click on cartservice
3. Show: "Owner: dotnet-team, Members: 5 engineers"
4. Show: Recent deployments, SLO assignments
5. Trigger synthetic alert
6. Watch it auto-route to dotnet-team via Slack
7. Show GitHub link: "Here's the code, here's the runbook"

Result: "Unified ownership model → faster incident response"
```

