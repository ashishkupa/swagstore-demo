# Datadog Service Catalog Ownership - Troubleshooting Guide

## 🔍 Common Issues & Solutions

### Issue 1: Services Not Showing in Service Catalog

**Symptoms:**
- Go to Service Catalog but see 0 services or only some services
- Team field is empty even though service exists

**Solutions:**

#### Step 1: Verify GitHub Integration Connection
1. Go to Datadog → **Integrations** → **GitHub**
2. Look for your organization
3. Verify it says "Connected" or "Installed"
4. If not connected:
   - Click "Install Integration"
   - Authorize with GitHub
   - Select your repository

#### Step 2: Force Re-sync from GitHub
1. In the GitHub integration page
2. Look for a **"Resync"** or **"Refresh"** button
3. Click it
4. Wait **10-15 minutes** for sync to complete

#### Step 3: Check File Format

**CODEOWNERS must be:**
- In the root directory of your repo
- Named exactly: `CODEOWNERS` (case-sensitive)
- Proper format: `path/to/code @team-name`

**Example:** Check https://github.com/ashishkupa/swagstore-demo/blob/main/CODEOWNERS

**service.datadog.yaml must be:**
- Valid YAML format (check for syntax errors)
- In each service directory
- Readable by Datadog parser

Test YAML syntax:
```bash
python -m yaml src/cartservice/service.datadog.yaml
```

---

### Issue 2: Team Shows as Empty in Service Catalog

**Symptoms:**
- Service appears in catalog
- But "Team" field is blank or shows nothing

**Root Causes & Fixes:**

#### Cause A: CODEOWNERS Not Synced Yet
- Solution: Wait 15 minutes and refresh
- Or: Manually force resync in GitHub integration

#### Cause B: CODEOWNERS Has Wrong Format
```
# ❌ WRONG
src/cartservice squad-checkout

# ✅ CORRECT  
src/cartservice/ @squad-checkout
```

Check syntax:
- Path should end with `/`
- Team name should start with `@`

#### Cause C: service.datadog.yaml Missing Team Field
Check the file has:
```yaml
team: squad-checkout
```

If missing, add it to every service.datadog.yaml

---

### Issue 3: Service Catalog Shows Old Team After Update

**Symptoms:**
- You updated CODEOWNERS
- Datadog still shows old team assignment

**Solution:**
1. Go to GitHub integration
2. Click **Disconnect** or **Remove**
3. Wait 2 minutes
4. Click **Install** or **Connect** again
5. Wait 15 minutes for full sync

This forces a complete rescan.

---

## ✅ Complete Verification Checklist

### Step 1: Verify Files on GitHub

Run these commands or check in GitHub UI:

```bash
# 1. Check CODEOWNERS exists
curl -s https://raw.githubusercontent.com/ashishkupa/swagstore-demo/main/CODEOWNERS | head -5

# 2. Check service.datadog.yaml exists
curl -s https://raw.githubusercontent.com/ashishkupa/swagstore-demo/main/src/cartservice/service.datadog.yaml | head -10
```

Expected output:
```
# CODEOWNERS output:
# Swagstore Demo - Code Ownership
src/frontend/ @squad-storefront

# service.datadog.yaml output:
service:
  name: cartservice
  team: squad-checkout
```

### Step 2: Verify in Datadog

1. Go to **Service Catalog**
   - URL: https://app.datadoghq.com/service-catalog

2. Search for: `cartservice`

3. Click on it and verify:
   ```
   Name: cartservice
   Team: squad-checkout ← Should be filled
   Type: backend
   Language: dotnet
   ```

4. If Team is blank:
   - Go back to GitHub integration
   - Click Resync
   - Wait 10 minutes
   - Refresh Service Catalog

### Step 3: Test Ownership in GitHub

Create a test PR to verify CODEOWNERS works:

```bash
git checkout -b test/codeowners-test
echo "test" > src/cartservice/test.txt
git add src/cartservice/test.txt
git commit -m "Test CODEOWNERS"
git push origin test/codeowners-test
```

Then create a PR to main:
- GitHub should automatically request review from `@squad-checkout`
- If it does, CODEOWNERS is working
- Close the PR (don't merge)

---

## 🔧 Manual Verification Script

Run this to check everything:

```bash
#!/bin/bash
cd /home/ashishk/Documents/mylab/swagstore-demo

echo "1. Check CODEOWNERS format:"
grep "squad-" CODEOWNERS | head -3

echo -e "\n2. Check service.datadog.yaml has team:"
grep "team:" src/*/service.datadog.yaml | head -3

echo -e "\n3. Check CODEOWNERS is in root:"
ls -la CODEOWNERS

echo -e "\n4. Verify service files exist:"
ls -1 src/*/service.datadog.yaml | wc -l
```

---

## 📊 What Should Happen Step-by-Step

### Timeline:

```
You: Update CODEOWNERS on GitHub
   ↓ (GitHub stores it)

GitHub Integration: Detects change (webhook)
   ↓ (Datadog receives webhook)

Datadog: Queues rescan
   ↓ (Usually happens automatically)

After 5-15 minutes: Datadog syncs
   ↓ (Discovers services and ownership)

Service Catalog: Shows updated team assignments
   ↓ (You see it in UI)
```

**If it's been more than 20 minutes:** Force manual resync

---

## 🚀 Nuclear Option: Complete Reset

If nothing works, try this:

1. **In Datadog:**
   - Go to Integrations → GitHub
   - Click Disconnect/Uninstall
   - Wait 5 minutes

2. **In GitHub:**
   - Go to Settings → Applications → Authorized Apps
   - Find Datadog
   - Click Revoke

3. **Reconnect:**
   - Back to Datadog Integrations → GitHub
   - Click Install
   - Re-authorize
   - Wait 20 minutes

4. **Verify:**
   - Go to Service Catalog
   - All services should re-appear with ownership

---

## 📞 Quick Debugging

### Q: How do I know if GitHub integration is working?
**A:** 
1. Go to GitHub integration page
2. Should say "Connected" or "Installed"
3. Should show your organization name
4. Should have a resync option

### Q: How do I know if CODEOWNERS is correct?
**A:** Create a test PR (see Step 3 above). If GitHub requests review from the team, it's working.

### Q: How do I know if service.datadog.yaml is correct?
**A:** 
- Check if YAML is valid: `python -m yaml src/cartservice/service.datadog.yaml`
- Check if `team:` field exists in the file
- Wait 20 minutes and check Datadog

### Q: How long does sync take?
**A:** Usually 5-10 minutes. Can take up to 20 minutes. If more than 30 minutes, try manual resync.

---

## 📋 Pre-Interview Checklist

Before your interview, verify:

- [ ] Go to Service Catalog
- [ ] All 12 services are showing
- [ ] Click on cartservice
- [ ] Team field shows: squad-checkout
- [ ] Click on frontend
- [ ] Team field shows: squad-storefront
- [ ] Click on paymentservice
- [ ] Team field shows: squad-payments

If any team field is blank:
1. Force resync in GitHub integration
2. Wait 20 minutes
3. Refresh page

---

## What Are You Seeing?

Tell me:
1. **Is Service Catalog showing any services?** (Yes/No)
2. **How many services?** (0, 1-5, 6-11, all 12)
3. **When you click a service, is Team field:**
   - Empty?
   - Showing wrong team?
   - Showing correct team?
4. **When did you update CODEOWNERS?** (minutes ago)

This will help me give more specific advice!

