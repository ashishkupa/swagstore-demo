# ✅ Datadog Software Catalog Setup - Verification & Next Steps

## Status: GitHub Push ✅ + GitHub Integration ✅

Your commits are now on GitHub and Datadog GitHub integration is connected. Now we need to verify the Software Catalog is discovering your services.

---

## 🔍 Step 1: Verify Services Auto-Discovery

### In Datadog:

1. Go to **Service Catalog**
   - URL: https://app.datadoghq.com/service-catalog

2. You should see services appearing with their team ownership

3. **Expected to see:**
   ```
   ✅ adservice (squad-payments)
   ✅ cartservice (squad-checkout)
   ✅ checkoutservice (squad-checkout)
   ✅ currencyservice (squad-checkout)
   ✅ emailservice (squad-engagement)
   ✅ frontend (squad-storefront)
   ✅ paymentservice (squad-payments)
   ✅ productcatalogservice (squad-storefront)
   ✅ recommendationservice (squad-engagement)
   ✅ shippingservice (squad-checkout)
   ✅ shoppingassistantservice (squad-engagement)
   ```

### If services are NOT appearing:

**Try manual sync:**
1. Go to Datadog → **Integrations** → **GitHub**
2. Look for your org in the list
3. Click **Sync** or **Resync**
4. Wait 5-10 minutes

**If still missing, check:**
1. Verify files are on GitHub:
   ```
   https://github.com/ashishkupa/swagstore-demo/blob/main/CODEOWNERS
   https://github.com/ashishkupa/swagstore-demo/blob/main/src/frontend/service.datadog.yaml
   ```
   (Check a few services)

2. Verify file format is correct:
   - CODEOWNERS must be in root directory
   - service.datadog.yaml must be in each service folder
   - Both files must be valid YAML/text

3. Force re-sync in Datadog:
   - Disconnect and reconnect GitHub integration
   - Wait 10 minutes

---

## 📋 Step 2: Verify Service Metadata

Click on any service (e.g., cartservice) and verify it shows:

```
✅ Service Name:        cartservice
✅ Team:                squad-checkout
✅ Description:         Manages shopping cart operations and inventory
✅ Type:                backend
✅ Language:            dotnet
✅ Framework:           gRPC
✅ Service Tier:        critical
✅ Contacts:            squad-checkout@company.com, #squad-checkout
✅ Links:               GitHub, Architecture, Runbook, etc.
✅ Tags:                team, component, service-tier, etc.
```

If metadata is NOT showing:
- Datadog is discovering the service but not reading service.datadog.yaml
- Verify file format: should be valid YAML
- Try re-sync

---

## 🎯 Step 3: Test Alert Routing

### Create a Test Monitor:

1. Go to **Monitors** → **New Monitor**
2. Choose **Metric Alert**
3. Configure:
   ```
   Metric: system.cpu.user
   Alert when: always
   ```
4. Set Notification:
   ```
   Notify: @squad-checkout
   ```
5. Save

### What should happen:

- Alert fires immediately (since we're alerting on always)
- Should mention `@squad-checkout` in message
- Verify Slack notification goes to the right channel

---

## 📊 Step 4: Create Squad Dashboards

Optional but powerful for interviews:

### Dashboard 1: Squad View

Create dashboard filtering by team:

1. New Dashboard → "Squad: Checkout"
2. Add widgets:
   - Services: cartservice, currencyservice, shippingservice, checkoutservice
   - Metrics: latency, error rate, throughput
   - Filter by team: squad-checkout

### Dashboard 2: Service Dependency

Create a dependency map:

1. New Dashboard → "Service Map"
2. Add Service Map widget
3. Shows all services + their dependencies
4. Click on edge to see latency between services

### Dashboard 3: Business Metrics

Create dashboard by business responsibility:

1. New Dashboard → "Conversion Metrics"
2. Filter services: squad-checkout, squad-storefront
3. Track: checkout success rate, cart abandonment, page load time

---

## 🔐 Step 5: Setup Team Access Control (Optional)

If you want to restrict what each squad can see:

### In Datadog:

1. Go to **Organization Settings** → **Roles**
2. Create role: "Squad: Checkout"
3. Add permissions:
   - View: cartservice, currencyservice, shippingservice, checkoutservice
   - Edit: same services
   - Cannot view: paymentservice, emailservice, etc.
4. Assign users to role

---

## 🎯 Step 6: Verify CODEOWNERS on GitHub

Test that GitHub is using CODEOWNERS for PR reviews:

1. Create a test branch:
   ```bash
   git checkout -b test/codeowners-verification
   echo "test" > src/cartservice/test.txt
   git add src/cartservice/test.txt
   git commit -m "Test CODEOWNERS"
   git push origin test/codeowners-verification
   ```

2. Create a Pull Request to main
   - GitHub should auto-request review from @squad-checkout
   - If it works, you know CODEOWNERS is active

3. Close the PR (don't merge)

---

## 📱 Step 7: Setup Slack Integration (Optional)

To route alerts to Slack channels:

1. In Datadog → **Integrations** → **Slack**
2. Connect your Slack workspace
3. In monitors/alerts, use:
   ```
   Notify: @squad-checkout
   ```
4. Datadog will automatically post to `#squad-checkout` channel (if it exists)

**Note:** Make sure Slack channels exist:
- #squad-storefront
- #squad-checkout
- #squad-payments
- #squad-engagement
- #platform-sre

---

## ✅ Interview Demo Checklist

Before your interview, verify:

- [ ] All 12 services appear in Service Catalog
- [ ] Each service shows correct team ownership
- [ ] Clicking service shows full metadata
- [ ] Service Map displays all services
- [ ] Dependencies are visible between services
- [ ] Alert routing works (test monitor created)
- [ ] Slack integration working (if using)
- [ ] GitHub CODEOWNERS verification PR created
- [ ] Demo scenario scripts ready (see INTERVIEW_DEMO_PLAN.md)

---

## 🎬 Interview Demo Flow

### Opening (2 minutes)
```
"Let me show you our service ownership model in Datadog"
→ Open Service Catalog
→ Point out: "We have 12 services across 5 squads"
```

### For Developer (5 minutes)
```
1. Show: Frontend service in Service Catalog
   "See? Squad Storefront owns this. Here's the code, here's the runbook."
   
2. Click: GitHub link
   "Direct link to repo with CODEOWNERS config"
   
3. Show: Architecture link
   "Team documentation"
```

### For SRE (5 minutes)
```
1. Show: Service Map
   "All 12 services, all dependencies visible"
   
2. Point to: cartservice
   "If this goes down, checkout and frontend are affected"
   
3. Show: cartservice latency trend
   "We track p99 latency, error rate, throughput per service"
   
4. Test: Create alert
   "Alert auto-routes to squad-checkout via Slack"
```

### For CISO (5 minutes)
```
1. Show: Service Catalog
   "Single source of truth for who owns what"
   
2. Filter: by service-tier: critical
   "Here are our critical services"
   
3. Filter: by tag: pci-compliant
   "These services are PCI compliant"
   
4. Show: paymentservice details
   "Team: squad-payments, Compliance: PCI, Contacts: email + Slack"
```

### Closing (2 minutes)
```
"One platform for all personas:
- Developers see where code lives
- SRE sees service health & dependencies
- CISO sees ownership & compliance

GitHub + Datadog = unified ownership model"
```

---

## 🆘 Troubleshooting

### Services not appearing in Service Catalog

**Problem:** GitHub integration is connected but no services showing

**Solution:**
1. Verify files on GitHub
2. Click Resync in GitHub integration
3. Wait 10 minutes
4. Check if service.datadog.yaml format is valid YAML

### Service shows but no team assignment

**Problem:** Service appears but "Team" field is empty

**Solution:**
1. Check CODEOWNERS file is in root directory
2. Verify syntax: `src/service-name/ @team-name`
3. Re-sync GitHub integration

### Alert doesn't route to team

**Problem:** Created alert with `@squad-checkout` but didn't go anywhere

**Solution:**
1. Verify Slack integration is connected
2. Verify Slack channel exists: #squad-checkout
3. Check alert notification format: should include `@squad-checkout`

### Service Map is empty

**Problem:** No services showing in Service Map

**Solution:**
1. Need APM data (traces) to show services
2. Since you have Datadog instrumentation, traces should be flowing
3. Wait 5 minutes, refresh
4. Check if loadgenerator is still running (sending traffic)

---

## 📚 Next Resources

- **TEAM_STRUCTURE.md** - Full team organization details
- **INTERVIEW_DEMO_PLAN.md** - Interview scenario details
- **QUICK_START.md** - Troubleshooting guide

---

## 🎉 You're Ready!

After verifying the checklist above, you're ready for:

✅ Interview demos  
✅ Team onboarding  
✅ Incident response automation  
✅ Compliance tracking  
✅ SLO management  

Your Swagstore demo is production-ready with professional service ownership and Datadog observability! 🚀

