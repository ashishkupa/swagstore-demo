# Swagstore Interview Demo: Datadog Multi-Persona Performance Monitoring

## Overview
Demonstrate Datadog's value across three key personas by simulating real-world performance issues and showing how each persona benefits from unified observability.

---

## Demo Architecture

### Three Scenarios (Run sequentially or in parallel)

#### **Scenario 1: Developer - N+1 Query Problem**
**Story**: "Frontend is slow when loading products. Where's the bottleneck?"

**What happens**:
- User adds item to cart
- Frontend calls checkout service
- Checkout calls product catalog (1 query)
- Product catalog calls database (N queries - one per product detail)
- Result: Checkout takes 2-3 seconds instead of 200ms

**Datadog Shows**:
- Distributed trace showing service call chain
- Database query waterfall
- Automatic N+1 detection (multiple queries in sequence)
- Performance profile showing time spent per database call
- Code path visualization

**Developer's "Aha" Moment**: 
"Oh! I can see EXACTLY where time is spent - that's the N+1 query problem. One database query spawned 5 more. Let me batch the queries."

---

#### **Scenario 2: SRE - Cascading Latency & Memory Leak**
**Story**: "Traffic is normal but customers report 10-second checkout delays. Site isn't down but response times are degrading."

**What happens**:
- cartservice has a memory leak (accumulates objects each request)
- Memory pressure causes GC pauses
- GC pauses add 500ms latency per checkout
- Each slow checkout times out other requests
- Cascade effect: frontend waits longer, timeout backlog grows, error rate climbs

**Datadog Shows**:
- Service dependency map showing cascade
- Latency heatmap: normal→slow→timeout over time
- Correlated memory metrics + latency metrics
- Pod resource exhaustion warning
- Trace flame graph showing GC pause in critical path

**SRE's "Aha" Moment**:
"This memory spike is DIRECTLY causing the latency cascade. Datadog traced it end-to-end and showed me the exact correlation. I can see which pod is the culprit and restart it now instead of guessing for 2 hours."

---

#### **Scenario 3: CISO - Security & Compliance**
**Story**: "We need to prove we're detecting anomalies and threats. Can we see suspicious activity?"

**What happens**:
1. **SQL Injection attempt** - attacker sends malicious payload
2. **Brute force attempt** - repeated failed auth attempts
3. **Unusual data access pattern** - user accessing products they shouldn't have access to
4. **Credential exposure** - API key accidentally logged
5. **Compliance violation** - sensitive data not encrypted in transit

**Datadog Shows**:
- Security signals for each attack
- User behavior baseline vs anomaly
- Automatic PII/secret detection and masking
- Audit trail: who accessed what, when
- Compliance dashboard: HIPAA/PCI-DSS violations
- Threat intelligence: matched against known attack patterns

**CISO's "Aha" Moment**:
"Datadog automatically detected and logged ALL of this. I have forensic evidence, audit trails, and automatic alerts. No manual log digging. This PROVES we have visibility for compliance audits."

---

## Code Changes Required

### 1. **Cartservice - Intentional Latency (for Scenario 1 - N+1)**

File: `src/cartservice/src/Program.cs`

```csharp
// Add artificial latency to simulate N+1 query problem
private async Task<Cart> GetCartWithNPlusOne(string userId)
{
    var cart = await redis.GetAsync(userId);
    
    // Simulate N+1: query database once per item
    foreach (var item in cart.Items)
    {
        // This should be batch query, but we're doing it 1-by-1
        var product = await db.GetProductAsync(item.ProductId);
        await Task.Delay(500); // Intentional 500ms per item delay
        item.Product = product;
    }
    
    return cart;
}
```

**Datadog will show**: Service spans with individual database calls visible in trace waterfall.

---

### 2. **Cartservice - Memory Leak (for Scenario 2 - Latency Cascade)**

File: `src/cartservice/src/CartService.cs`

```csharp
private static List<byte[]> leakyCache = new(); // Memory leak!

public async Task<Cart> GetCartAsync(string userId)
{
    // Simulate memory leak - accumulate garbage each request
    leakyCache.Add(new byte[10_000]); // 10KB per request
    
    if (leakyCache.Count > 10_000)
    {
        // Every 10k requests, trigger GC pause
        GC.Collect();
        GC.WaitForPendingFinalizers();
        await Task.Delay(1000); // 1 second pause (simulated GC)
    }
    
    // Normal operation
    return await redis.GetAsync(userId);
}
```

**Datadog will show**: Memory spike → GC pause → latency spike → cascade effect.

---

### 3. **Frontend - SQL Injection Attempt (for Scenario 3 - Security)**

File: `src/frontend/handlers/products.go`

```go
// Vulnerable endpoint (for demo purposes ONLY)
func SearchProducts(w http.ResponseWriter, r *http.Request) {
    search := r.URL.Query().Get("q")
    
    // This is vulnerable - but we'll catch it with Datadog ASM
    query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", search)
    
    // Datadog ASM (Application Security Monitoring) detects:
    // - "' OR '1'='1" patterns
    // - SQL keywords in user input
    // - Attack signatures
    
    products := db.Query(query)
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(products)
}
```

**Datadog will show**: ASM signal detecting SQL injection pattern, blocking or alerting.

---

### 4. **Checkout Service - Credential Exposure (for Scenario 3 - Security)**

File: `src/checkoutservice/main.py`

```python
@app.route('/process-payment', methods=['POST'])
def process_payment():
    data = request.get_json()
    
    # Intentional credential exposure (for demo)
    api_key = os.getenv('STRIPE_API_KEY')
    
    # This gets logged (bad practice)
    logger.info(f"Processing payment with key: {api_key}")  # EXPOSED!
    
    # Datadog Secret Scanner detects this and:
    # - Masks the key in logs
    # - Alerts security team
    # - Creates audit trail
    
    return process_stripe_payment(data, api_key)
```

**Datadog will show**: PII/Secret detection, automatic masking, compliance violation.

---

### 5. **Product Catalog - Anomalous Access Pattern (for Scenario 3 - Security)**

File: `src/productcatalogservice/main.go`

```go
func GetProduct(w http.ResponseWriter, r *http.Request) {
    userID := r.Header.Get("X-User-ID")
    productID := r.URL.Query().Get("id")
    
    // Datadog tracks:
    // - Normal user access patterns (usually 5-10 products per session)
    // - Anomalies (user requesting 1000 products in 60 seconds)
    // - Baseline comparison (Thursday noon vs 3am)
    
    product := getProductFromDB(productID)
    
    // Set tags for Datadog User Behavior Analysis
    span.SetTag("user.id", userID)
    span.SetTag("product.id", productID)
    span.SetTag("user.role", getUserRole(userID))
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(product)
}
```

**Datadog will show**: User behavior baseline vs current activity, anomaly detection alerts.

---

### 6. **Email Service - Error Rate Simulation (for Scenario 2 - SRE)**

File: `src/emailservice/email_server.py`

```python
@app.route('/send-confirmation-email', methods=['POST'])
def send_confirmation():
    order_id = request.json.get('order_id')
    
    # Simulate increasing error rate under load
    if random.random() < 0.1:  # 10% error rate
        logger.error(f"Failed to send email for order {order_id}")
        # Datadog tracks this error
        return jsonify({"error": "Failed"}), 500
    
    # Datadog sees:
    # - Error rate climbing from 0.1% to 10%
    # - Error spike correlates with latency spike
    # - Alert triggers automatically
    
    send_email(order_id)
    return jsonify({"success": True}), 200
```

**Datadog will show**: Error rate monitor, correlation with other metrics.

---

## Demo Script & Flow

### **Part 1: Setup (2 minutes)**
1. Show Datadog dashboard with all services healthy
2. Explain three personas: Developer, SRE, CISO
3. Show unified platform: all data in one place

### **Part 2: Developer Scenario (5 minutes)**

```
Interviewer: "Our checkout is slow. Help us debug."

1. Show frontend - checkout button takes 2 seconds
2. Open Datadog APM → Traces
3. Filter by checkout service
4. Show trace waterfall:
   - Frontend call (50ms)
   - Checkout service call (1900ms) ← TOO SLOW
   - CartService call (500ms × 4) ← N+1 QUERY!
5. Click into CartService span
6. Show database query timeline
7. Point out: "4 sequential database queries for 1 checkout"
8. Show code: "Should batch these into 1 query"

Developer Action: "I'll fix the N+1 query by batching"
Datadog Benefit: "Distributed tracing showed exact bottleneck without code changes"
```

### **Part 3: SRE Scenario (5 minutes)**

```
Interviewer: "Customers report slow site but we have no alerts. What's happening?"

1. Show Datadog Service Map
2. Highlight cartservice with latency spike
3. Open dashboards:
   - Latency heatmap: normal→slow over 5 minutes
   - Memory graph: memory climbing
   - GC pause timing: spike at exact same time as latency
4. Show correlation:
   - Memory spike at 14:05:00
   - Latency spike at 14:05:00
   - Error spike at 14:05:00
5. Show affected pods:
   - cartservice-xxxx using 90% memory
6. Show impact cascade:
   - cartservice slow → checkout slow → frontend slow

SRE Action: "Restart that pod immediately" (or auto-remediation)
Datadog Benefit: "Automatic correlation + visual cascade = 5-minute diagnosis instead of 2 hours"
```

### **Part 4: CISO Scenario (5 minutes)**

```
Interviewer: "We need to prove we're secure. What threats did you detect?"

1. Show Datadog Security Signals
2. List detected threats:
   - SQL injection attempt (blocked)
   - Brute force attempt (3 failed logins)
   - Credential exposure (API key in logs, masked)
   - Anomalous data access (user accessed 500 products in 60 seconds)
   - Compliance violation (PII transmitted unencrypted)
3. Click each signal:
   - Show full context (user, time, endpoint, payload)
   - Show automatic remediation (rate limit applied)
   - Show audit trail (timestamped events)
4. Show Compliance Dashboard:
   - PCI-DSS: 100% (no credit card data logged)
   - HIPAA: 100% (no health data leaked)
   - SOC2: 98% (1 temporary alert)

CISO Action: "Download compliance report for auditor"
Datadog Benefit: "Automatic threat detection + compliance proof = audit readiness"
```

### **Part 5: The "One Platform" Message (3 minutes)**

```
Show side-by-side:

LEFT (Without Datadog):
- Developer: SSH into pod, grep logs, write custom queries
- SRE: Prometheus + Grafana + ELK + PagerDuty (5 tools)
- CISO: Manual log review, spreadsheet for compliance

RIGHT (With Datadog):
- Developer: Open Traces, see N+1 in 30 seconds
- SRE: Open Service Map, correlate metrics, fix in 5 minutes
- CISO: Open Security Signals, export compliance report

"One platform. All personas. All use cases."
```

---

## Datadog Configuration for Demo

### Monitors to Create

```yaml
# For Developer
- N+1 Query Monitor: Alert if database calls > 5 per trace
- Slow Endpoint Monitor: Alert if p99 latency > 500ms

# For SRE
- Memory Pressure Monitor: Alert if memory > 80%
- Latency Cascade Monitor: Alert if latency spike + error spike + memory spike all correlate
- Pod Restart Monitor: Alert if pod restarts > 2 in 5 minutes

# For CISO
- SQL Injection Attempt Monitor: Alert on ASM SQL injection signals
- Credential Exposure Monitor: Alert on secret scanner detections
- User Behavior Anomaly Monitor: Alert on unusual access patterns
- Compliance Violation Monitor: Alert on PCI/HIPAA violations
```

### Dashboards to Create

```yaml
# Developer Dashboard
- Service dependency map
- Latency distribution (p50, p95, p99)
- Top slowest traces
- Error rate by endpoint
- Database query analysis

# SRE Dashboard
- Pod resource utilization
- Service health status
- Error rates + latency correlation
- Incident timeline
- Performance trends

# CISO Dashboard
- Security signals (threats detected)
- Compliance status (PCI, HIPAA, SOC2)
- User access audit trail
- Vulnerability trends
- PII/Secret exposure events
```

---

## Key Datadog Features to Highlight

### **For Developer**
- ✅ Distributed Tracing (end-to-end request flow)
- ✅ Service Map (dependencies + latency)
- ✅ Database Query Analysis (N+1 detection)
- ✅ Code-level profiling (where CPU time goes)
- ✅ Error tracking (stack traces + frequency)

### **For SRE**
- ✅ Infrastructure metrics (CPU, memory, disk)
- ✅ Service health monitoring
- ✅ Alert correlation (metric A + metric B = action)
- ✅ Incident response (timeline, context, suggested actions)
- ✅ On-call integration (PagerDuty, Slack)

### **For CISO**
- ✅ Application Security Monitoring (ASM)
- ✅ Threat detection (SQL injection, RCE, etc.)
- ✅ Compliance dashboards (PCI-DSS, HIPAA, SOC2)
- ✅ User behavior analysis (anomaly detection)
- ✅ Audit logging (who accessed what, when)
- ✅ Secret scanning + masking
- ✅ Forensics (drill into any incident)

---

## Interview Impact

### Narrative Arc
1. **Problem**: Real-world performance issues (everyone can relate)
2. **Confusion**: Traditional tools require context switching and guesswork
3. **Datadog Solution**: One platform shows the answer
4. **Value**: Each persona sees their role's value immediately
5. **Message**: "Datadog is the platform that unifies all observability"

### Expected Reactions
- **Developer**: "I can finally see where time is actually spent!"
- **SRE**: "This correlation saved me hours of troubleshooting!"
- **CISO**: "We're actually compliant and secure - I have proof!"

---

## Timing & Deliverables

| Phase | Task | Time |
|-------|------|------|
| **Phase 1** | Code changes (latency, memory leak, security events) | 2 hours |
| **Phase 2** | Datadog dashboard setup | 1 hour |
| **Phase 3** | Monitor creation + alerting | 1 hour |
| **Phase 4** | Demo script + walk-through practice | 1 hour |
| **Interview** | Live demo (15 minutes) | 15 min |

---

## Ready to Build?
Would you like me to:
1. Create the code changes for all 5 scenarios?
2. Generate the Datadog dashboard JSON?
3. Write the monitor definitions?
4. Create the full demo script with timing cues?

Let me know which you'd like to start with!
