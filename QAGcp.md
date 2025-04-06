
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
