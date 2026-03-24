# Service Catalog Not Discovering Services - Deep Troubleshooting

## Problem: Services Not Showing in Service Catalog

When services don't appear in Service Catalog, it usually means one of these:

1. **GitHub Integration connection issue**
2. **Service definition files not discoverable**
3. **APM data not flowing** (services need to be running and traced)
4. **Datadog agent not collecting traces**

---

## 🔍 Step 1: Verify GitHub Integration Status

### In Datadog:

1. Go to: **Integrations** → **GitHub**
2. Look for: Your GitHub organization

**Check for:**
```
Status: Connected ✅ OR Installed ✅
Organization: ashishkupa
Permissions: Read repositories ✅
```

**If it says "Not Connected":**
1. Click "Install Integration"
2. Authorize with GitHub
3. Select your repository
4. Wait 5 minutes

---

## 🔍 Step 2: Verify Service Discovery

### Option A: Check if Datadog Can See Your Repo

In Datadog:
1. Go to **Integrations** → **GitHub**
2. Under your org, check if it lists: `swagstore-demo`
3. If it doesn't, click the org name to refresh

### Option B: Manual Discovery Trigger

Some integrations need a manual trigger:
1. In GitHub integration page
2. Look for button: "Sync", "Rescan", or "Refresh"
3. Click it
4. Wait **20-30 minutes** for discovery

---

## 🔍 Step 3: Verify APM is Running

Service Catalog auto-discovers services from TWO sources:

### Source 1: APM Traces
- Services running with Datadog instrumentation
- Must send traces to Datadog
- This is your primary discovery method

**Check if traces are flowing:**
1. Go to **APM** → **Services**
2. Do you see your services listed? (cartservice, frontend, etc.)
3. If yes: APM is working, Services should appear in Catalog
4. If no: APM instrumentation isn't working

### Source 2: Service Definition Files (Optional)
- CODEOWNERS
- service.datadog.yaml
- These enhance discovery but don't create services on their own

---

## 🔍 Step 4: Check Kubernetes Cluster

**Your services might not be discovered if:**

1. **Pods aren't running:**
   ```bash
   kubectl get pods
   ```
   Should show 12/12 pods Running

2. **Datadog agent isn't collecting traces:**
   ```bash
   kubectl get pods -n datadog
   # or
   kubectl get daemonset -A | grep datadog
   ```

3. **Traces aren't flowing:**
   - Services running? ✅
   - Datadog agent collecting? ✅
   - Network connectivity? ✅

---

## 🚀 Likely Solution: Start from APM

The core issue is probably that **services need to send traces to Datadog first**.

### Here's what needs to happen:

1. **Services must be running:**
   ```bash
   kubectl get pods
   # Should show 12 pods in Running state
   ```

2. **Services must have APM enabled:**
   - Already done (Datadog tracer installed in images)

3. **Datadog Agent must be collecting traces:**
   - Check: Is there a `datadog-agent` pod running?
   - Or DaemonSet collecting APM?

4. **Traces must reach Datadog:**
   - Check in Datadog UI: **APM** → **Services**
   - Do you see: cartservice, frontend, etc.?

5. **If you see services in APM:**
   - They WILL appear in Service Catalog automatically
   - Usually within 5-10 minutes

---

## 📋 Complete Checklist

### Do This Now:

- [ ] **Step 1:** Check Kubernetes pods are running
  ```bash
  kubectl get pods
  # Count: how many are "Running"?
  ```

- [ ] **Step 2:** Check if Datadog agent is deployed
  ```bash
  kubectl get pods -n datadog
  # OR
  kubectl get daemonset -A | grep -i datadog
  ```

- [ ] **Step 3:** Check APM in Datadog
  - Go to: Datadog → **APM** → **Services**
  - Do you see any services? List them:

- [ ] **Step 4:** If no services in APM, check Datadog agent logs
  ```bash
  kubectl logs -n datadog daemonset/datadog-agent --tail=50
  ```

- [ ] **Step 5:** Check GitHub integration
  - Datadog → **Integrations** → **GitHub**
  - Status: Connected?

---

## 🎯 Most Likely Issue: Missing APM Data

**Why services won't appear in Service Catalog:**
- ❌ Services not running
- ❌ Datadog agent not collecting traces
- ❌ APM disabled or misconfigured
- ❌ Network blocking traces

**If you fix any of these, services will appear automatically in Service Catalog**

---

## 🆘 Emergency Troubleshooting

### If services aren't in APM (Datadog → APM → Services):

1. **Verify pods are running:**
   ```bash
   kubectl get pods -l app=cartservice
   kubectl logs cartservice-xxx
   ```

2. **Verify Datadog agent exists:**
   ```bash
   kubectl get daemonset -A
   ```
   Should show `datadog-agent` or similar

3. **Check if traces can reach Datadog:**
   - Look for environment variables in pod:
   ```bash
   kubectl exec cartservice-xxx -- env | grep DD_
   ```

4. **Check for network issues:**
   - Can pod reach Datadog endpoint?
   ```bash
   kubectl exec cartservice-xxx -- curl -v https://trace.agentless.datadoghq.com/v0.4/traces
   ```

---

## 📞 Answer These Questions

Tell me:

1. **Are your Kubernetes pods running?**
   ```
   Run: kubectl get pods
   How many show "Running"? __/12
   ```

2. **Do you see services in Datadog APM?**
   ```
   Go to: Datadog → APM → Services
   List any services you see:
   ```

3. **Is Datadog agent deployed?**
   ```
   Run: kubectl get daemonset -A | grep datadog
   What do you see?
   ```

4. **When did you set up Datadog GitHub integration?**
   ```
   Minutes ago? Hours ago?
   ```

This will help me pinpoint the exact issue!

---

## 🔗 Key Insight

**Service Catalog discoveries services from APM traces first.**

The sequence is:
```
1. Service running + sending traces
   ↓
2. Datadog agent collects traces
   ↓
3. Traces reach Datadog backend
   ↓
4. Service appears in APM → Services
   ↓
5. Service appears in Service Catalog
   ↓
6. CODEOWNERS/service.datadog.yaml adds metadata
```

If services don't show up, trace collection is blocked somewhere in this chain.

