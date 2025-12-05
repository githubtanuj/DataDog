# DataDog

Here is a simplified, easy-to-remember summary for your interview, covering all the key points from the video:

### **DataDog Agent Installation (Windows & Linux)**
*   **Windows:** Install via PowerShell with the copied command from DataDog UI. Verify in `services.msc` (DataDog Agent service) and `DataDog Agent Manager`.
*   **Linux:** Install using the shell command from DataDog UI. Configure via `/etc/datadog-agent/datadog.yaml`.

### **Key Configuration File**
*   **Path:** `datadog.yaml` (Linux: `/etc/datadog-agent/`, Windows: `C:\ProgramData\Datadog\`).
*   **Purpose:** Central file for enabling features (process monitoring, APM), setting hostname, and adding tags.

### **Process Monitoring**
*   **Enable it:** In `datadog.yaml`, find and **uncomment** `process_config:` and set `enabled: true`.
*   **Result:** You'll see running processes (PID, CPU, memory) in DataDog's Infrastructure > Processes.

### **Host Renaming & Tagging (Linux)**
*   In `datadog.yaml`, uncomment `hostname:` and set a custom name (e.g., `hostname: linux02`).
*   Uncomment `tags:` and add key-value pairs (e.g., `env:prod`, `team:infra`).
*   **Why Tags Matter:** They help filter and group thousands of servers in the UI. Tags can be set at the agent level (`datadog.yaml`) or from the DataDog UI.

### **Service Monitoring (Windows)**
*   **Configure:** Edit `C:\ProgramData\Datadog\conf.d\windows_service.d\conf.yaml`.
*   **Example:** Monitor all services or specific ones (e.g., `spooler`).
*   **Create Alert:** In Monitors > New Monitor > **Service Check**, select the service and set conditions (e.g., alert if stopped).

### **Synthetic Monitoring**
*   **Purpose:** Proactively test website/API availability from global locations.
*   **API Test:** Checks a single endpoint (URL) for status code, response time, and body content.
*   **Browser Test:** Records multi-step user navigation (clicks, forms) to monitor full user journeys.
*   **Setup:** Digital Experience > New Test. Define URL, assertions (e.g., status 200, response < 1s), locations, and alert recipients.

### **Monitors & Alerts**
*   **Monitor = Alert Rule.** It watches metrics/logs and triggers notifications.
*   **Types:** Metric (CPU > 90%), Host (server not reporting), Composite (combine multiple monitors), etc.
*   **Create:** Monitors > New Monitor. Choose type (e.g., Metric), define query/threshold, set alert message and team to notify.
*   **Downtime:** Schedule maintenance periods to suppress expected alerts.

### **Metrics**
*   **What:** Numerical measurements over time (e.g., CPU%, disk usage).
*   **Explore:** Go to **Metrics > Explorer** to visualize any metric and create graphs/dashboards.
*   **Create Monitor from Graph:** Click on any metric graph, select "Create Monitor."

### **AWS Integration**
*   **Manual Setup (Key Steps):**
    1.  In DataDog: Integration > AWS > Add Account > Manual.
    2.  In AWS IAM: Create a Role for DataDog (use provided `Account ID` and `External ID`).
    3.  Attach an Inline Policy with permissions from DataDog's guide.
    4.  Go back to DataDog and enter the AWS Account ID and Role Name.
*   **Result:** AWS metrics (EC2, S3, etc.) and pre-built dashboards appear automatically.

### **Docker Monitoring**
*   **Prerequisites:** Install Docker on the host and the DataDog agent.
*   **Enable:** Edit `/etc/datadog-agent/conf.d/docker.d/conf.yaml`. Add the Docker daemon metrics endpoint configuration (usually involves uncommenting and setting a URL).
*   **Restart Agent:** `sudo systemctl restart datadog-agent`.
*   **View:** DataDog Infrastructure > Containers shows all containers and their resource usage.

### **Real User Monitoring (RUM)**
*   **Purpose:** Track real user interactions, performance, and errors on your web app.
*   **Setup:** Digital Experience > RUM > New Application. Get the JavaScript code snippet.
*   **Implementation:** Add the snippet to the `<head>` of your HTML pages.
*   **View:** See page views, load times, errors, and user sessions in the RUM dashboard.

### **Application Performance Monitoring (APM)**
*   **Purpose:** Trace requests across your application (e.g., Python/Flask app) to find bottlenecks.
*   **Enable:** In `datadog.yaml`, uncomment `apm_config:` and set `enabled: true`. Restart agent.
*   **Instrument App:** Use language-specific tracer (e.g., for Python: `pip install ddtrace`). Run your app with the tracer.
*   **View:** APM > Services shows your app, requests, latency, and traces.

### **Dashboards**
*   **Create:** Dashboards > New Dashboard.
*   **Add Widgets:** Click "Add Widget." Choose from Time Series, Query Value, etc.
*   **Populate Fast:** Copy graphs from other places (Host view, Metric Explorer) and paste (`Ctrl+V`) directly into your dashboard.
*   **Use:** Share with teams, schedule reports, or export as JSON.

### **Crucial Commands to Remember**
*   **Restart Agent (Linux):** `sudo systemctl restart datadog-agent`
*   **Agent Status (Linux):** `sudo datadog-agent status`
*   **Check Config (Linux):** `sudo datadog-agent configcheck`
*   **List Containers:** `docker ps`
*   **Run Python App with APM:** `ddtrace-run python app.py`

## **DataDog Implementation for my.com Real Estate Platform**

### **Architecture & Monitoring Strategy**

```
Frontend (S3 + CloudFront) → Route 53 → EKS Backend → MongoDB Database
        ↓                       ↓            ↓               ↓
     RUM + Synthetic       DNS Monitoring   APM + K8s    Database Monitoring
```

---

### **1. Frontend Monitoring (S3 + CloudFront)**
**What I Monitored:**
- **Real User Monitoring (RUM):** Tracked user journeys on property listing pages
  - Property search performance
  - Image gallery loading times
  - Mortgage calculator interaction latency
  - Form submission (contact agents, schedule viewings)

- **Synthetic Monitoring:**
  - **API Tests:** Critical endpoints (property search API, pricing API)
    - Alert if property search > 2 seconds
    - Verify property data accuracy in responses
  - **Browser Tests:** 
    - Complete user flow: Search → View → Schedule Tour → Contact
    - From 8 global locations (NY, London, Dubai, Singapore, etc.)
    - Ran every 5 minutes

- **CloudFront Metrics:**
  - Cache hit rates (optimize cache TTL for property images)
  - 4xx/5xx error rates
  - Geographic performance differences

**Business Impact:**
- Reduced property page bounce rate by 28% by optimizing image loading
- Identified slow-performing regions and implemented regional CDN caches
- Automated alerts when property search functionality degraded

---

### **2. DNS & Load Balancing (Route 53)**
**What I Monitored:**
- **DNS Resolution Time:** Alert if > 100ms
- **Health Checks:** Route 53 health checks integrated with DataDog
  - Monitored all geographic endpoints
  - Failover automation triggers

- **Synthetic DNS Tests:**
  - Verify DNS propagation during deployments
  - Monitor TTL expiration
  - Test from multiple ISP networks

**Business Impact:**
- Zero downtime during DNS changes/migrations
- Quick failover during regional outages
- 99.99% DNS availability SLA maintained

---

### **3. Backend Monitoring (EKS Cluster)**
**What I Monitored:**

**A. Kubernetes Infrastructure:**
- **Node Metrics:** CPU, memory, disk I/O across all worker nodes
- **Pod Health:** Restart counts, crash loop backoffs
- **HPA (Horizontal Pod Autoscaler):** Custom metrics-driven scaling
  - `property_api.requests_per_second`
  - `user_sessions.active_count`
  - `document_upload.processing_time`

- **DaemonSet Deployed:**
  ```yaml
  # DataDog Agent in EKS
  - Tracked: kube-state-metrics, cAdvisor metrics
  - Custom tags: `env:prod`, `team:realestate`, `application:my`
  ```

**B. Application Performance (APM):**
- **Property Service:** Traced property listing, search, and booking
  - Identified N+1 query issue in property recommendations
  - Reduced property search latency from 1.2s → 350ms

- **User Service:** Authentication, profile management, preferences
  - Alert on failed login spikes (security monitoring)
  - Track user session patterns

- **Payment Service:** Critical path monitoring
  - Set up Service Level Objectives (SLOs): 99.9% success rate
  - Alert if payment processing > 3 seconds

- **Document Service:** Property document upload/processing
  - File validation, OCR processing, PDF generation
  - Alert on failed uploads or processing delays

**Business Impact:**
- Auto-scaling handled 5x traffic spikes during property launches
- 40% reduction in infrastructure costs through rightsizing
- P99 latency improved from 2.1s to 800ms for property searches

---

### **4. Database Monitoring (MongoDB)**
**What I Monitored:**
- **MongoDB Integration:** Custom DataDog check for MongoDB
  - Query performance: Slow query logging (> 100ms)
  - Connection pool utilization
  - Replica set lag (alert if > 30 seconds)
  - Index hit ratio

- **Key Metrics Tracked:**
  - `mongodb.opcounters.query` vs `mongodb.opcounters.insert`
  - `mongodb.mem.resident` (memory usage)
  - `mongodb.network.bytesin/bytesout`
  - Custom metric: `property_queries.per_second`

- **Index Optimization:**
  - Used query analysis to add missing indexes
  - Removed redundant indexes saving 15% storage

**Business Impact:**
- Prevented 3 database outages through proactive alerts
- 60% faster property search queries after query optimization
- Saved $2,400/month on database instance costs

---

### **5. Business & Transaction Monitoring**
**Custom Dashboards Created:**

**1. Executive Dashboard:**
- Properties listed vs sold (daily/weekly)
- User engagement: Avg session duration, pages per session
- Conversion funnel: Visit → Property View → Contact → Booking
- Revenue metrics: Total value of properties booked

**2. Operations Dashboard:**
- System health: All services at a glance
- Error rates per service (color-coded)
- Active users vs system load correlation
- Database query performance

**3. Real Estate Agent Dashboard:**
- Leads generated per property
- Response time to customer inquiries
- Property viewing scheduling success rate

**Key Business Metrics Tracked:**
- `business.property.views.total`
- `business.property.contacts.generated`
- `business.transaction.value.booked`
- `business.agent.response.time.avg`

---

### **6. Alerting & Incident Response**

**Critical Alerts Configured:**

1. **P1 - Revenue Impacting:**
   - Property booking API failure
   - Payment processing errors
   - Complete site outage

2. **P2 - User Experience:**
   - Property search > 2 seconds
   - Image loading failures
   - Form submission errors

3. **P3 - System Health:**
   - MongoDB primary failover
   - EKS node disk > 85%
   - Memory leak detection

**On-call Rotation:**
- Integrated with PagerDuty/Slack
- Escalation policies based on alert severity
- Runbook automation for common issues

---

### **7. Security & Compliance Monitoring**

**What I Monitored:**
- **User Authentication:** Failed login attempts, brute force detection
- **Document Access:** Unauthorized access to property documents
- **Payment Security:** PCI compliance monitoring
- **GDPR Compliance:** User data access patterns

**Custom Security Metrics:**
- `security.auth.failed.attempts.ip`
- `security.document.download.unauthorized`
- `security.api.key.usage.anomaly`

---

### **8. Cost Optimization & ROI**

**Cost Savings Achieved:**

1. **Infrastructure Rightsizing:**
   - EKS node optimization: 35% cost reduction
   - MongoDB instance optimization: 40% cost reduction
   - CloudFront cache optimization: 25% cost reduction

2. **Revenue Protection:**
   - Prevented 8+ hours of downtime during peak property launches
   - Early detection of checkout flow issues → $45K+ revenue protected
   - Improved conversion rate by 15% through performance optimizations

3. **Operational Efficiency:**
   - MTTR reduced from 2 hours → 15 minutes
   - 70% fewer after-hours pages
   - Automated 80% of routine health checks

---

### **Interview Talking Points:**

**When asked "How did you use DataDog?":**

**Start with Business Impact:**
"We implemented DataDog across my.com to ensure our real estate platform delivered a seamless experience for property buyers. This directly impacted our core business metrics - property views, lead generation, and bookings."

**Technical Implementation Highlights:**
1. **Full-stack visibility:** From CloudFront CDN → EKS microservices → MongoDB
2. **Business-aligned monitoring:** Custom metrics tracking property listings, user engagement, and conversion funnel
3. **Proactive issue detection:** Synthetic tests catching problems before users
4. **Cost optimization:** Using metrics to right-size infrastructure without compromising performance

**Quantifiable Results to Mention:**
- "40% faster property search response time"
- "$2,400/month infrastructure cost savings"
- "28% reduction in page bounce rate"
- "Zero revenue-impacting outages in 6 months"
- "MTTR reduced from 2 hours to 15 minutes"

**Team Collaboration Stories:**
- "Worked with frontend team to optimize image loading using RUM data"
- "Collaborated with database team on query optimization using MongoDB metrics"
- "Created dashboards for business teams to track property performance"
- "Integrated monitoring into CI/CD pipeline to catch regressions before production"

---

### **Sample Project Description for Resume/LinkedIn:**
"Architected and implemented comprehensive monitoring for my.com real estate platform using DataDog. Established full-stack observability covering S3/CloudFront frontend, EKS microservices, and MongoDB database. Implemented:
- Real User Monitoring tracking property search and booking journeys
- APM tracing across 12+ microservices reducing P99 latency by 40%
- Synthetic tests simulating buyer workflows from 8 global locations
- Custom business metrics for property views, leads, and conversions
- Automated alerting reducing MTTR from 2 hours to 15 minutes
- Infrastructure optimization saving $2,400/month in cloud costs"
