
---

### ✅ **1. Basic MySQL + GCP Setup**

**Q1. How do you deploy MySQL in a GCP environment?**  
**A:**  
You can deploy MySQL in GCP using:  
- **Cloud SQL for MySQL**: Fully managed MySQL instance by Google.  
- **GCE (Google Compute Engine)**: Install and configure MySQL manually on a VM.  
- **GKE (Google Kubernetes Engine)**: Run MySQL as a containerized application (StatefulSet with PVC).  

**Best practice:** For production, prefer **Cloud SQL** due to automated backups, failover, monitoring, and patching.

---

### ✅ **2. High Availability (HA) and Failover**

**Q2. How does Cloud SQL provide high availability for MySQL?**  
**A:**  
- Cloud SQL uses **regional instances** with synchronous replication across zones.  
- It automatically promotes a standby replica on failure.  
- Uses **Cloud SQL Proxy** or **private IP** for secure connections and DNS-based failover.  

**Q3. Can you set up MySQL replication manually in GCP?**  
**A:**  
Yes, using GCE or GKE:  
- Configure a master-slave (async) or group replication.  
- Use GCP features like **Load Balancer**, **Managed Instance Groups**, and **Persistent Disks** for reliability.

---

### ✅ **3. Backup and Restore**

**Q4. How are backups handled in Cloud SQL MySQL?**  
**A:**  
- Automated backups (daily, customizable window).  
- Manual backups supported.  
- PITR (Point-in-Time Recovery) is available using binary logs.  
- Backups are stored in GCS (Google Cloud Storage), encrypted by default.  

**Command-line:**  
```bash
gcloud sql backups list --instance=my-instance
```

---

### ✅ **4. Monitoring and Logging**

**Q5. How do you monitor a MySQL instance on GCP?**  
**A:**  
- **Cloud Monitoring** (formerly Stackdriver): Metrics for CPU, memory, queries, connections.  
- **Cloud Logging**: Slow query logs, error logs, audit logs.  
- Integration with **Grafana**, **Prometheus** via Ops Agent on GCE.  

---

### ✅ **5. Performance Optimization**

**Q6. How do you tune performance of MySQL in Cloud SQL?**  
**A:**  
- Use **Query Insights** for analysis.  
- Modify DB flags like `innodb_buffer_pool_size`, `max_connections`, etc.  
- Use **read replicas** for read scaling.  
- Enable slow query logging and index recommendations.

---

### ✅ **6. Security**

**Q7. How is MySQL secured in GCP?**  
**A:**  
- **IAM roles** for access control.  
- **SSL/TLS** for encryption in transit.  
- **At-rest encryption** by default using Google-managed keys or CMEK.  
- Private IP or VPC peering for secure communication.  
- **Cloud SQL Auth Proxy** for secure IAM-based authentication.

---

### ✅ **7. Troubleshooting**

**Q8. MySQL instance is unreachable from my app on GCP. How do you troubleshoot?**  
**A:**  
- Check if using **Cloud SQL Auth Proxy** or **private IP** correctly.  
- Ensure **IAM permissions** and **network/VPC settings** (firewall rules) are correct.  
- Check connection limits and database status (`gcloud sql instances describe`).  
- Look into logs via **Cloud Logging**.

---

### ✅ **8. Scalability & Replication**

**Q9. How do you horizontally scale MySQL on GCP?**  
**A:**  
- Use **read replicas** (up to 10 in Cloud SQL).  
- Use **external replicas** for cross-region disaster recovery.  
- Application must implement read/write split.  

---

### ✅ **9. Migration**

**Q10. How do you migrate MySQL from on-prem to GCP?**  
**A:**  
Options:  
- **Database Migration Service (DMS)**: Managed, minimal downtime.  
- `mysqldump` for small datasets.  
- **Percona Xtrabackup** or `mysqlpump` for large live datasets.  
- Replication-based migration (set source as master, GCP as replica).  

---

### ✅ **10. Cost Optimization**

**Q11. How do you control MySQL cost in GCP?**  
**A:**  
- Use **committed use discounts**.  
- Stop development instances when not in use.  
- Choose **custom machine types** with optimized memory.  
- Right-size storage and enable auto storage increase.  
- Monitor with **Budgets & Alerts** in GCP Billing.

---

### ✅ **1. Architecture & Design**

**Q1. How would you design a highly available and scalable MySQL setup on GCP for a global application?**  
**A:**  
- **Primary in Cloud SQL** with regional HA (automatic failover).  
- **Read replicas** across regions for read scaling and disaster recovery.  
- **Cloud Load Balancer + Cloud SQL Auth Proxy** or Proxyless connection with **Private IP + VPC Peering**.  
- Use **Cloud Armor**, **IAM**, and **Cloud Audit Logs** for access control and auditing.  
- For latency-sensitive apps: Use **Cloud Spanner** if consistency and global writes are critical.

---

### ✅ **2. Performance Optimization**

**Q2. What are the most critical MySQL flags or settings you'd tune in a production Cloud SQL instance?**  
**A:**  
- `innodb_buffer_pool_size`: 60–80% of RAM for InnoDB-heavy workloads.  
- `max_connections`: Based on concurrency needs.  
- `query_cache_type=0`, `query_cache_size=0`: Avoid query cache in newer MySQL.  
- `innodb_flush_log_at_trx_commit=2`: Trade-off between durability and performance.  
- Enable **slow query logs**, use **Query Insights**, and push logs to **Cloud Logging**.  
- Use **gcloud sql flags update** or set during instance creation.

---

### ✅ **3. Monitoring & Observability**

**Q3. How do you implement full observability for MySQL running on Cloud SQL?**  
**A:**  
- **Cloud Monitoring dashboards**: Query latency, QPS, connection usage, buffer pool hit rate.  
- **Cloud Logging**: Enable error log, slow query log, audit log.  
- Enable **Query Insights** with digest analysis.  
- Use **Prometheus exporters** and **Grafana** dashboards for custom metrics via Ops Agent (GCE-based MySQL).  
- Set up **SLOs/SLAs**, **alerting policies**, and **uptime checks**.

---

### ✅ **4. Disaster Recovery & Backups**

**Q4. How do you design an RTO/RPO strategy for MySQL in GCP?**  
**A:**  
- Enable **automated backups** with **Point-In-Time Recovery** (PITR).  
- Store backups in **GCS**, encrypted with **CMEK** if required.  
- RTO < 5 mins with failover replicas. RPO < 5 mins using binary log streaming.  
- Periodically test backup restoration using `gcloud sql backups restore`.  
- Use **Database Migration Service** for zero-downtime DR tests.

---

### ✅ **5. Security**

**Q5. What GCP security features would you integrate for securing MySQL?**  
**A:**  
- **IAM roles + Cloud SQL IAM database authentication**.  
- Use **Private IP** and **VPC Service Controls**.  
- Enable **SSL/TLS** for client-server encryption.  
- Audit with **Cloud Audit Logs**, integrate with **SIEM** tools.  
- Use **Customer-Managed Encryption Keys (CMEK)** if regulatory compliance needed.  

---

### ✅ **6. Automation & IaC**

**Q6. How would you manage MySQL deployments and configuration using Infrastructure-as-Code?**  
**A:**  
- Use **Terraform** modules for Cloud SQL instance provisioning.  
- Store DB parameters in **Terraform variables**, version-controlled.  
- Use **Cloud Build** or **GitHub Actions** for CI/CD to test DB changes.  
- Automate DB migrations using tools like **Flyway**, **Liquibase**, or custom scripts triggered post-deploy.

---

### ✅ **7. Real-Time Troubleshooting**

**Q7. Users report increased latency during peak hours. How do you investigate in Cloud SQL?**  
**A:**  
1. Check **Cloud Monitoring** for CPU, memory, disk IOPS.  
2. Review **Query Insights** for slow or expensive queries.  
3. Verify **connection pool saturation** or thread locking.  
4. Validate `SHOW ENGINE INNODB STATUS` via Cloud SQL connection.  
5. If disk IO is a bottleneck, consider **scaling storage tier** (e.g., SSD → high-performance tier).  
6. Optionally, spin up read replicas for analytical offloading.

---

### ✅ **8. Migration & Upgrade Strategy**

**Q8. How do you handle version upgrades for MySQL in GCP?**  
**A:**  
- Cloud SQL supports **in-place major version upgrades** with validation steps.  
- Best practice:  
  - Clone instance → upgrade clone → test thoroughly → promote to prod.  
- For self-managed GCE MySQL: use **Percona’s apt/yum repos**, test in **staging**, use **replication-based cutover**.

---

### ✅ **9. Cost Management**

**Q9. How do you optimize cost without compromising performance in GCP MySQL setup?**  
**A:**  
- Use **custom machine types** tailored to workload (e.g., high-memory vs balanced).  
- Enable **auto-storage increase**, disable over-provisioning.  
- **Read replicas** with smaller specs for analytics.  
- Use **Cloud Scheduler + gcloud CLI** to shut down non-prod instances during off-hours.  
- Monitor cost with **Billing Reports** and **Budgets + Alerts**.

---

### ✅ **10. Leadership Scenario**

**Q10. How would you lead a database incident postmortem where MySQL latency affected a customer-facing app?**  
**A:**  
- Conduct **blameless postmortem** with timeline, impact, root cause.  
- Share **runbook** and alerts involved, note where they failed or succeeded.  
- Identify **human factors**, **tooling gaps**, **lack of observability**, or **capacity planning issues**.  
- Propose **action items**: automation, better alerting, scaling policies.  
- Add learnings to **internal documentation**, schedule follow-up **chaos testing**.

---

