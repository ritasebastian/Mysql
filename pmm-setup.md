Here’s the **PMM step-by-step guide** to install and configure **Percona Monitoring and Management (PMM 2)** on **CentOS EC2**, **excluding MySQL installation**, assuming MySQL is already installed and running on the same instance:

---

## ✅ PMM 2 Client + Server Setup on CentOS EC2 (MySQL already installed)

---

### 🟩 **1. Launch EC2 (CentOS 7 or 8)**
- Choose CentOS from AWS Marketplace
- Open these **Security Group ports**:
  - `22` (SSH)
  - `80`, `443` (PMM Web UI)
  - `3306` (MySQL if remote access is needed)

---

### 🟩 **2. Install PMM 2 Client (Download + Install)**

```bash
wget https://ftpmirror.your.org/pub/percona/pmm2-client/yum/release/7/RPMS/x86_64/pmm2-client-2.42.0-6.el7.x86_64.rpm
sudo yum install -y pmm2-client-2.42.0-6.el7.x86_64.rpm
```

---

### 🟩 **3. Install Docker & Run PMM Server**
```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce
sudo systemctl enable --now docker

# Pull and run PMM Server
docker pull percona/pmm-server:2
docker create -v /srv --name pmm-data percona/pmm-server:2 /bin/true
docker run -d -p 80:80 -p 443:443 --volumes-from pmm-data --name pmm-server percona/pmm-server:2
```

---

### 🟩 **4. Configure PMM Client**
```bash
sudo pmm-admin config --server-url=https://admin:admin@<EC2-PUBLIC-IP> --insecure
```

Example:
```bash
sudo pmm-admin config --server-url=https://admin:admin@34.224.25.97 --insecure
```

---

### 🟩 **5. Register Local MySQL to PMM**
```bash
sudo pmm-admin add mysql --username=root --password=dbaonly123 --query-source=perfschema
```

✅ Output should show:
```
MySQL Service added.
Table statistics collection enabled.
```

---

### 🟩 **6. Access PMM Web Dashboard**
Open in browser:
```
http://<EC2-PUBLIC-IP>/
```

Default Login:
- **Username:** `admin`
- **Password:** `admin` → prompt to change

Navigate to:
- Dashboards → **MySQL Overview**, **Query Analytics**

---

### 🟩 **7. (Optional) Set Up Grafana Alerts**

#### Example: Alert if CPU > 80%
1. Go to **Dashboards → Node Summary**
2. Click CPU panel → **Edit**
3. Go to **Alert → Create Alert**
4. Set condition:
   ```
   WHEN avg() OF query (e.g. node_load1) IS ABOVE 80 FOR 5m
   ```
5. Add Notification (Slack, Email, etc.)

To create notification:
- Go to **Alerting → Contact Points → Add Contact Point**

---

### 🟩 **8. Verify PMM is Monitoring MySQL**
```bash
sudo pmm-admin list
```

You should see:
- Agent status: `running`
- Service: `mysql`
- Query Source: `perfschema`

---

