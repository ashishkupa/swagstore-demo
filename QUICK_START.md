# 🚀 Quick Reference: GitHub Push & Software Catalog Setup

## ⚡ 3-Minute Setup

### Step 1: Get GitHub Token
```bash
# Go to: https://github.com/settings/tokens
# Create Personal Access Token (classic)
# Scopes: repo, workflow
# Copy your token (you'll need it below)
```

### Step 2: Push to GitHub (Choose ONE)

#### Option A: Using Token (Easiest)
```bash
cd /home/ashishk/Documents/mylab/swagstore-demo

# Replace YOUR_TOKEN with your actual token
git remote set-url origin https://YOUR_TOKEN@github.com/ashishkupa/swagstore-demo.git

# Push
git push origin main

# Verify
git log --oneline -1
```

#### Option B: Using SSH
```bash
# Generate key (if you don't have one)
ssh-keygen -t ed25519 -C "your-email@company.com"

# Add public key to GitHub: https://github.com/settings/ssh/new

# Configure git
git remote set-url origin git@github.com:ashishkupa/swagstore-demo.git

# Push
git push origin main
```

### Step 3: Verify on GitHub
Open: https://github.com/ashishkupa/swagstore-demo

Check for:
- ✅ CODEOWNERS file
- ✅ service.datadog.yaml in each service folder

---

## ✨ Datadog Software Catalog Setup (5 minutes)

### Step 1: Connect GitHub
1. Go to Datadog → **Integrations**
2. Search for **GitHub**
3. Click **Install Integration**
4. Authorize with GitHub
5. Select your repo (or allow all)
6. Confirm

### Step 2: Wait for Sync
- Datadog will automatically discover:
  - CODEOWNERS file
  - service.datadog.yaml files
  - Services get registered
  - Takes 5-15 minutes

### Step 3: Verify in Datadog
1. Go to **Service Catalog**
2. You should see 12 services with owners:
   ```
   ✅ adservice (ads-team)
   ✅ cartservice (dotnet-team)
   ✅ checkoutservice (backend-team)
   ✅ ... (all 11 more)
   ```

### Step 4: Click Any Service
See:
- ✅ Description
- ✅ Team ownership
- ✅ Contact info
- ✅ GitHub links
- ✅ Service tier
- ✅ Tags

---

## 📊 What You'll Have

| Artifact | Status | Value |
|----------|--------|-------|
| CODEOWNERS | ✅ Created | GitHub auto-requests code owners on PRs |
| service.datadog.yaml | ✅ Created (11 files) | Datadog Software Catalog source of truth |
| GitHub Integration | ⏳ To setup | Syncs ownership to Datadog |
| Service Catalog | ⏳ To verify | Single pane of glass for all services |
| Team Routing | ⏳ To test | Alerts automatically go to right team |

---

## 🎯 Interview Demo (After Setup)

### Show This Flow

```
"How do you manage ownership and incident response?"

1. Alert fires on cartservice
   → Datadog looks up CODEOWNERS
   → Routes to @dotnet-team

2. Team gets Slack notification with:
   → Service health dashboard
   → GitHub links to recent commits
   → Datadog traces showing issue
   → Runbook link

3. Show Service Catalog:
   → 12 services, all with owners
   → Dependencies drawn between them
   → SLOs assigned to each
   → Deployment history

"One source of truth for ownership - GitHub, Datadog, Slack all in sync"
```

---

## 📱 Slack Commands (After Setup)

```bash
# After alerts are routed to team
@datadogbot alert on cartservice
# Datadog sends alert to @dotnet-team

# After Slack integration
/datadog service cartservice
# Shows service info, owner, status
```

---

## ✅ Checklist

- [ ] Generate GitHub Personal Access Token
- [ ] Push commits: `git push origin main`
- [ ] Verify files on GitHub
- [ ] Connect GitHub to Datadog (Integrations)
- [ ] Wait 5-15 minutes for sync
- [ ] Check Service Catalog has 12 services
- [ ] Click service → verify owner shows
- [ ] Test alert routing (optional)
- [ ] Review for interview

---

## 🆘 Troubleshooting

### "Permission denied" on git push
**Fix**: Use token instead of SSH, or generate SSH key properly

### Services not appearing in Datadog
**Fix**: 
1. Ensure files pushed to GitHub
2. Go to Integrations → GitHub → click Sync
3. Wait 5 more minutes

### Owner shows as blank in Service Catalog
**Fix**:
1. Check CODEOWNERS syntax
2. Ensure service.datadog.yaml has team name
3. Re-sync GitHub integration

---

## 📞 Quick Links

- GitHub Token: https://github.com/settings/tokens
- GitHub SSH Keys: https://github.com/settings/ssh/new
- Datadog Integrations: https://app.datadoghq.com/integrations
- Datadog Service Catalog: https://app.datadoghq.com/service-catalog

---

## 💡 Pro Tips

1. **Test with One Service First**
   - Push CODEOWNERS
   - Let Datadog discover cartservice
   - Verify owner appears
   - Then all others will follow

2. **Use Preview Mode**
   - In Datadog, you can preview service before publishing
   - Check all links work before going live

3. **Team Naming**
   - Use GitHub team handles (e.g., @dotnet-team)
   - Easier for Slack integration
   - Easier for permissions

4. **Documentation Links**
   - Include Runbook links in service.datadog.yaml
   - Include Architecture links
   - Include deployment links
   - Make incident response faster

