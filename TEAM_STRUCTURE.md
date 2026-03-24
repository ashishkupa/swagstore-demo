# Swagstore Team Structure & Ownership (Updated)

## 📋 Team Organization

### **Squad: Storefront** 🛍️
**Owned Services:** Frontend, Product Catalog  
**Primary Tech Stack:** Go  
**Business Responsibility:** User experience, SEO, and product browsing

| Service | Language | Type | Tier |
|---------|----------|------|------|
| frontend | Go | Frontend/HTTP | Critical |
| productcatalogservice | Go | Backend/gRPC | Critical |

**Contact:** `@squad-storefront` → `#squad-storefront` Slack  
**Key Metrics:** Page load time, SEO ranking, product search performance

---

### **Squad: Checkout** 🛒
**Owned Services:** Cart Service, Currency Service, Shipping, Checkout  
**Primary Tech Stack:** C# (.NET), Node.js, Go  
**Business Responsibility:** Conversion rate, tax calculations, logistics integration

| Service | Language | Type | Tier |
|---------|----------|------|------|
| cartservice | C# (.NET) | Backend/gRPC | Critical |
| currencyservice | Node.js | Backend/gRPC | Standard |
| shippingservice | Go | Backend/gRPC | Standard |
| checkoutservice | Go | Backend/gRPC | Critical |

**Contact:** `@squad-checkout` → `#squad-checkout` Slack  
**Key Metrics:** Conversion rate, cart abandonment, shipping accuracy, tax compliance

---

### **Squad: Payments** 💳
**Owned Services:** Payment Service, Ad Service  
**Primary Tech Stack:** Java  
**Business Responsibility:** Transaction security, PCI compliance, revenue tracking

| Service | Language | Type | Tier |
|---------|----------|------|------|
| paymentservice | Node.js | Backend/gRPC | Critical |
| adservice | Java | Backend/gRPC | Critical |

**Contact:** `@squad-payments` → `#squad-payments` Slack  
**Key Metrics:** Transaction success rate, PCI audit compliance, revenue, fraud detection

---

### **Squad: Engagement** 💌
**Owned Services:** Recommendation, Email Service, Shopping Assistant  
**Primary Tech Stack:** Python  
**Business Responsibility:** Customer retention, AI-driven upsells, notifications

| Service | Language | Type | Tier |
|---------|----------|------|------|
| recommendationservice | Python | Backend/gRPC | Standard |
| emailservice | Python | Backend/gRPC | Standard |
| shoppingassistantservice | Python | Backend/HTTP | Standard |

**Contact:** `@squad-engagement` → `#squad-engagement` Slack  
**Key Metrics:** Customer retention, recommendation accuracy, email engagement rate, LLM latency

---

### **Platform / SRE** 🏗️
**Owned Infrastructure:** MariaDB, Redis, Istio Mesh, Kubernetes, Docker  
**Business Responsibility:** Database reliability, network latency, cluster health

| Component | Type | Responsibility |
|-----------|------|-----------------|
| Kubernetes manifests | IaC | Cluster orchestration |
| Docker/Dockerfile | Containers | Image building & optimization |
| Redis | Cache | Session & cart caching |
| MariaDB | Database | Primary data store |
| Istio Mesh | Service Mesh | Networking & security |
| CI/CD Pipeline | Automation | Deployment automation |

**Contact:** `@platform-sre` → `#platform-sre` Slack  
**Key Metrics:** Cluster uptime, latency p99, database response time, deployment frequency

---

## 🔄 CODEOWNERS Routing

When PRs are created, GitHub automatically requests reviews from:

```
# Squad: Storefront
src/frontend/ → @squad-storefront
src/productcatalogservice/ → @squad-storefront

# Squad: Checkout
src/cartservice/ → @squad-checkout
src/currencyservice/ → @squad-checkout
src/shippingservice/ → @squad-checkout
src/checkoutservice/ → @squad-checkout

# Squad: Payments
src/paymentservice/ → @squad-payments
src/adservice/ → @squad-payments

# Squad: Engagement
src/recommendationservice/ → @squad-engagement
src/emailservice/ → @squad-engagement
src/shoppingassistantservice/ → @squad-engagement

# Platform/SRE
kubernetes-manifests/ → @platform-sre
Dockerfile → @platform-sre
.github/workflows/ → @platform-sre
```

---

## 📊 Service Catalog Mapping

In Datadog Service Catalog, each service shows:

```yaml
# Example: cartservice
team: squad-checkout
tech-stack: dotnet
service-tier: critical
business-responsibility: "conversion-rate, tax-calculations, logistics-integration"

# Example: recommendationservice
team: squad-engagement
tech-stack: python
service-tier: standard
business-responsibility: "customer-retention, ai-driven-upsells, notifications"
```

---

## 🎯 Incident Response Flow

**When an alert fires:**

1. **Alert on frontend** → Routed to `@squad-storefront`
2. **Alert on cartservice** → Routed to `@squad-checkout`
3. **Alert on paymentservice** → Routed to `@squad-payments`
4. **Alert on recommendationservice** → Routed to `@squad-engagement`
5. **Alert on database/cluster** → Routed to `@platform-sre`

**Each team receives:**
- Slack notification with service name
- Link to Service Catalog (owner, contacts, runbook)
- Link to traces & logs
- Business context (what they own)

---

## 👥 Cross-Squad Dependencies

```
Storefront
  ↓ (calls)
Product Catalog

Frontend
  ↓ (calls)
Checkout Service
  ↓ (calls)
Cart Service
  ↓ (calls)
Product Catalog

Checkout Service
  ↓ (calls)
Currency Service (exchange rates)
  ↓ (calls)
Shipping Service (logistics)
  ↓ (calls)
Payment Service

Payment Service
  ↓ (calls)
Ad Service (revenue tracking)

Recommendation Service
  ↓ (calls)
Email Service (send recommendations)
```

**For incident response:** If `cartservice` is down, it affects `checkout` and `frontend`. Datadog shows this dependency chain automatically.

---

## 📈 SLO Assignment by Team

### **Squad: Storefront**
- Frontend Availability: 99.99% uptime
- Product Search: p99 latency < 200ms
- SEO Indexing: All products indexed within 24h

### **Squad: Checkout**
- Checkout Completion: 99.5% success rate
- Cart Operations: p99 latency < 500ms
- Shipping Calculation: p99 latency < 1s

### **Squad: Payments**
- Payment Success Rate: 99.9%
- PCI Compliance: 100% (zero violations)
- Revenue Reporting: 0% data loss

### **Squad: Engagement**
- Recommendation Accuracy: 85% match rate
- Email Delivery: 99.5% in-inbox rate
- Assistant Latency: p99 < 2s

### **Platform/SRE**
- Database Uptime: 99.99%
- Cluster Health: 100% pod availability
- Deployment Success: 95% first-time success rate

---

## 🔐 Access Control (Optional)

Using Datadog roles, you can give each squad access to only their services:

```
Squad: Storefront
- Can view: frontend, productcatalogservice traces, logs, dashboards
- Can view: cartservice performance (read-only, for dependencies)
- Cannot view: paymentservice, emailservice details

Squad: Checkout
- Can view: cartservice, checkoutservice, currencyservice, shippingservice
- Can view: frontend performance (read-only)
- Cannot modify: paymentservice

Squad: Payments
- Can view: paymentservice, adservice
- Can view: checkout flow (dependency visibility)
- Must not: modify payment processing code without audit trail

Squad: Engagement
- Can view: recommendationservice, emailservice, shoppingassistantservice
- Can experiment: with recommendation algorithms
- Can deploy: changes to non-production first

Platform/SRE
- Full access to all services (for troubleshooting)
- Can modify infrastructure, deployments, alerts
```

---

## 📝 Git Commits

**Latest:** `a291f26`
```
Update team ownership structure for Swagstore

Reorganize services by squad/business responsibility:
- Squad: Storefront (frontend, product catalog)
- Squad: Checkout (cart, currency, shipping, checkout)
- Squad: Payments (payment, ads)
- Squad: Engagement (recommendations, email, assistant)
- Platform/SRE (infrastructure)

Each service includes team, tech stack, tier, and business tags.
```

---

## ✅ Next Steps

1. **Push to GitHub**
   ```bash
   git push origin main
   ```

2. **Connect to Datadog**
   - Go to Integrations → GitHub
   - Install integration
   - Wait 5-10 minutes for sync

3. **Verify in Service Catalog**
   - Check all 12 services appear
   - Verify team assignments
   - Click each service to see metadata

4. **Test Alert Routing**
   - Create test alert on cartservice
   - Should route to `@squad-checkout`
   - Verify Slack notification

5. **Ready for Interview**
   - Show Datadog Service Catalog
   - Highlight team ownership model
   - Show incident routing automation

---

## 🎉 Result

You now have:
- ✅ Clear service ownership
- ✅ Squad-based organization
- ✅ Automatic alert routing
- ✅ Business responsibility tracking
- ✅ Tech stack classification
- ✅ Unified Datadog integration
- ✅ GitHub PR owner assignment
- ✅ Interview-ready demo
