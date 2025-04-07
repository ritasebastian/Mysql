# 📘 Cloud SQL for MySQL - 50 Tough Interview Questions & Answers

## 1. How does Cloud SQL handle high availability (HA) for MySQL?
Cloud SQL provides HA using regional instances and synchronous replication. It deploys a primary and a standby in different zones. Failover is automatic via a shared IP, and can be manually triggered with `gcloud sql instances failover`.

## 2. How do you monitor and alert on Cloud SQL (MySQL) performance issues?
Use Cloud Monitoring for system metrics. Enable slow query logs and Query Insights. Set alerts on CPU, memory, disk, and replication lag.

## 3. How do you manage schema changes in Cloud SQL (MySQL) with zero downtime?
Use tools like `gh-ost` or `pt-online-schema-change`. Apply changes on replicas first. Avoid large transactions. Monitor query latency.

## 4. How do you handle backup and disaster recovery for Cloud SQL (MySQL)?
Enable automated backups and point-in-time recovery. Take on-demand backups before changes. Use multi-region replicas or export to Cloud Storage.

## 5. Describe a scenario where you had to recover from a corrupted Cloud SQL instance.
Clone the instance from backup, replay binary logs, validate data, promote the clone, and redirect traffic. Document and audit the incident.

## 6. Can you explain how to use Cloud SQL with private IP and VPC?
Use VPC peering with private services access. Assign private IP via `gcloud`. Access Cloud SQL securely without exposing to public internet.

## 7. What are the limitations of Cloud SQL for MySQL compared to self-managed MySQL?
Limited access to `my.cnf`, no plugins, restricted privileges (no SUPER), 64TB storage cap, and no OS-level access.

## 8. How do you ensure secure access to Cloud SQL instances?
Use IAM roles, Cloud SQL Proxy, SSL enforcement, Secret Manager for credentials, and private IP. Rotate secrets regularly.

## 9. How do you scale Cloud SQL for MySQL horizontally?
Use read replicas for scaling reads. For writes, use application-level sharding. Consider offloading analytics to BigQuery.

## 10. How do you diagnose replication lag in MySQL read replicas in Cloud SQL?
Check `SHOW REPLICA STATUS`. Monitor `Seconds_Behind_Master`. Optimize writes, index slow queries, and tune disk IOPS.

## 11. How do you handle connection limit errors in Cloud SQL?
Enable connection pooling, reduce idle timeouts, monitor connection count, and increase `max_connections` if needed.

## 12. How do you debug slow inserts or updates?
Use `EXPLAIN`, Query Insights, and slow query logs. Optimize queries, reduce row scans, add indexes, and batch writes.

## 13. What happens if you restart a Cloud SQL instance?
Connections are dropped. Config flags are re-applied. Expect brief downtime (~1–2 min). Ensure retry logic is implemented.

## 14. How do you manage scheduled maintenance windows?
Configure preferred maintenance day/time. Monitor upcoming events via Cloud Console or Pub/Sub alerts.

## 15. How do you handle deadlocks in Cloud SQL?
Analyze `SHOW ENGINE INNODB STATUS`. Fix lock order, reduce transaction scope, retry error 1213, and tune isolation levels.

## 16. What is the maximum storage size and how does autoscaling work?
Cloud SQL supports up to 64TB. Auto-scaling grows storage in 25GB increments when usage exceeds 95%.

## 17. How do you handle timezone issues in Cloud SQL?
Store in UTC. Let app handle conversions. Load time zone tables with `mysql_tzinfo_to_sql`.

## 18. How do you diagnose high CPU usage in Cloud SQL?
Use Cloud Monitoring, `SHOW PROCESSLIST`, and `performance_schema`. Optimize queries and tune buffer pool size.

## 19. What is the difference between failover replica vs read replica?
Failover replica is HA with synchronous replication. Read replica is for scaling reads and is asynchronously replicated.

## 20. How would you migrate encrypted data into Cloud SQL securely?
Encrypt dumps using GPG. Use Cloud Storage with CMEK. Use `gcloud sql import` with IAM-restricted access.

## 21. How do you reduce Cloud SQL costs?
Right-size instances, use private IP, scale replicas wisely, turn off idle resources, and use sustained use discounts.

## 22. What is Cloud SQL Proxy and when to use it?
A secure way to connect using IAM-based auth. Ideal for local dev, CI/CD, and serverless apps.

## 23. How to migrate from AWS RDS to Cloud SQL with minimal downtime?
Use Database Migration Service. Set up ongoing replication. Cut over during off-peak time.

## 24. What maintenance flags have you tuned?
`innodb_buffer_pool_size`, `max_connections`, `slow_query_log`, `long_query_time`, `log_output`, etc.

## 25. How to troubleshoot "server closed connection unexpectedly"?
Check `wait_timeout`, connection pooling, crash logs, out-of-memory issues, or app-level retry logic.

## 26. What’s a good backup strategy?
Enable PITR + daily automated + on-demand pre-deployment. Test restore periodically.

## 27. How to set up cross-region DR?
Use read replicas in other regions, store backups in multi-region buckets, and simulate failover scenarios.

## 28. How to diagnose replication lag spike?
Check network, large writes, indexes, IOPS limits, and binlog format.

## 29. How do you store and rotate MySQL credentials securely?
Use Secret Manager with IAM roles. Use IAM DB authentication where possible.

## 30. Compare Cloud SQL vs MySQL on GCE.
Cloud SQL is managed with limited access. GCE gives full control but requires ops overhead.

## 31. How to enable audit logging?
Use Cloud Audit Logs. Native MySQL audit plugins not supported. Use GCE MySQL for advanced audit needs.

## 32. How to respond to sensitive data exposure?
Isolate instance, rotate secrets, check logs, restore backup, notify stakeholders, and improve controls.

## 33. How to upgrade MySQL version without downtime?
Replicate, promote, upgrade replica, cut over after validation. Test compatibility in staging.

## 34. How to scale write throughput?
Vertical scaling, sharding, batching, optimize binlog and transaction settings.

## 35. What is Query Insights?
Built-in tool for analyzing top queries, latency, execution plans, and bottlenecks.

## 36. How to enforce least privilege?
Use IAM roles for instance access, SQL-level grants, and rotate secrets.

## 37. How to monitor binlog size?
Use `SHOW BINARY LOGS`. Set `binlog_expire_logs_seconds`. Monitor disk usage.

## 38. How to handle timezone-aware apps?
Store in UTC. Use `CONVERT_TZ()` if needed. Handle logic in app layer.

## 39. How to structure multi-tenant MySQL in Cloud SQL?
Use shared schema (with tenant_id), schema-per-tenant, or instance-per-tenant based on use case.

## 40. Is Cloud SQL HIPAA/SOC2/GDPR compliant?
Yes, with BAA signed for HIPAA. Use encryption, audit logs, and VPC-SC for compliance.

## 41. How to tune `max_connections` safely?
Set based on workload. Use pooling. Monitor thread count and errors. Test under load.

## 42. How to optimize Cloud SQL for OLAP-style queries?
Use BigQuery for analytics. If in MySQL, tune buffer pool, indexes, and temp tables.

## 43. How to track changes in users/permissions?
Use IAM audit logs. For SQL changes, maintain audit tables or use GCE with plugins.

## 44. How to set up alerts in Cloud Monitoring?
Create metrics for CPU, disk, connections, replication lag. Integrate with Slack, OpsGenie, PagerDuty.

## 45. What is `performance_schema` and how is it used?
Built-in profiler for query diagnostics. Use it to track wait events, memory usage, and lock contention.

## 46. How to handle rogue queries impacting performance?
Find via `PROCESSLIST` or Insights. Kill if needed. Apply limits or rewrite. Use firewall rules if external.

## 47. Can Cloud SQL handle 10K+ connections?
Not directly. Use horizontal scaling with replicas, pooling layers, or rethink architecture.

## 48. How to test failover in HA setup?
Use `gcloud sql instances failover`. Monitor app reconnect logic and ensure data integrity.

## 49. How to migrate data from a self-hosted MySQL to Cloud SQL?
Use `mysqldump`, `mydumper`, or DMS. Validate collation, version, triggers, and roles before import.

## 50. How to automate Cloud SQL tasks?
Use Terraform, Cloud Scheduler, Cloud Functions, and `gcloud` CLI. Combine with Monitoring for alerting and healing.
