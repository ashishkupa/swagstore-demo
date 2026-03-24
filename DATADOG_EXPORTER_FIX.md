# ROOT CAUSE FOUND: Datadog Agent Export Configuration Issue

## The Problem

The Datadog agent logs show:
```
"debugexporter" | pushTraces
```

This means traces are being exported to a **debug exporter** instead of **Datadog's backend**. That's why services don't appear in Service Catalog - the traces never reach Datadog!

---

## Current Configuration

Your DatadogAgent CRD has:

✅ **APM Enabled:** true  
✅ **OTLP Receiver:** Listening on port 4317 (gRPC) and 4318 (HTTP)  
✅ **Traces Collected:** Locally in agent  
❌ **Traces Exported:** Going to debug exporter (not Datadog backend)  

---

## Solution: Configure Proper OTLP Exporter

You need to add an OTLP **exporter** section to send traces to Datadog.

### Option A: Update DatadogAgent CRD (Recommended)

Add this to the datadogagent spec:

```yaml
spec:
  features:
    otlp:
      receiver:
        protocols:
          grpc:
            enabled: true
            endpoint: 0.0.0.0:4317
          http:
            enabled: true
            endpoint: 0.0.0.0:4318
      exporter:                    # ADD THIS SECTION
        protocols:
          otlp:
            metrics:
              temporality: delta
              summaries: false
```

### Option B: Set Environment Variable

Add to DatadogAgent:

```yaml
spec:
  agent:
    env:
    - name: DD_OTLP_CONFIG_EXPORTER_OTLP_METRICS_TEMPORALITY
      value: delta
```

---

## How to Apply the Fix

### Step 1: Edit the DatadogAgent

```bash
kubectl edit datadogagent datadog -n observe
```

### Step 2: Add exporter configuration

Find the `otlp:` section and add:

```yaml
otlp:
  receiver:
    protocols:
      grpc:
        enabled: true
        endpoint: 0.0.0.0:4317
      http:
        enabled: true
        endpoint: 0.0.0.0:4318
  exporter:              # ← ADD THIS
    protocols:
      otlp:
        metrics:
          temporality: delta
          summaries: false
```

### Step 3: Save and wait

The agent will automatically restart with new config.

### Step 4: Verify

```bash
# Check agent logs for "otlpexporter" instead of "debugexporter"
kubectl logs -n observe daemonset/datadog-agent | grep exporter
```

Should show:
```
otlpexporter | pushTraces
```

Instead of:
```
debugexporter | pushTraces
```

---

## Why This Fixes It

**Before:**
- Traces collected locally in agent
- Exported to debug exporter (goes nowhere)
- Datadog doesn't receive traces
- Services don't appear in Service Catalog

**After:**
- Traces collected locally in agent
- Exported to Datadog OTLP endpoint
- Datadog receives traces
- Services auto-discover in Service Catalog
- Ownership automatically linked

---

## Timeline After Fix

```
1. Apply updated DatadogAgent config
   ↓
2. Agent restarts (< 1 minute)
   ↓
3. Traces start exporting to Datadog
   ↓
4. Services appear in APM → Services (5 minutes)
   ↓
5. Services appear in Service Catalog (2 minutes)
   ↓
6. Ownership tags auto-added from CODEOWNERS (1 minute)
   ↓
Total time: ~10 minutes
```

---

## Alternative: Use Datadog Exporter

If OTLP exporter doesn't work, try Datadog exporter format:

```yaml
otlp:
  exporter:
    protocols:
      http:
        endpoint: "https://otlp.datadoghq.com:443"
```

But OTLP should work since your agent is already connected (it has DD_API_KEY).

---

## Quick Summary

- ✅ Kubernetes cluster: Running  
- ✅ Services: Running (12/12)  
- ✅ Datadog agent: Running  
- ✅ APM configured: Yes  
- ✅ Traces collected: Yes  
- ❌ **Traces exported:** Using debug exporter (FIX NEEDED)  

**This is the last piece - fix the exporter and services will appear!**

