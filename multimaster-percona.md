### **🚀 Deploying a 3-Node Percona XtraDB Cluster (PXC) on AWS EC2**
This guide will walk you through setting up a **3-node Percona XtraDB Cluster (PXC) on AWS EC2 instances** to achieve **high availability and multi-master replication**.

---

## **🔹 Architecture Overview**
| **Node Type**  | **EC2 Instance** | **Private IP**  |
|---------------|----------------|---------------|
| **Node 1** (Primary) | `pxc-node-1` | `192.168.1.10` |
| **Node 2** | `pxc-node-2` | `192.168.1.11` |
| **Node 3** | `pxc-node-3` | `192.168.1.12` |

- **Region**: Choose the same AWS **VPC and security group** for all instances.
- **Instance Type**: Use **t3.medium** (2 vCPUs, 4GB RAM) or higher.
- **OS**: Ubuntu 22.04 LTS (or RHEL 8)

---

## **🔹 Step 1: Create AWS EC2 Instances**
1. **Launch 3 EC2 instances**:
   - Select **Ubuntu 22.04 LTS** or **RHEL 8**.
   - Choose **t3.medium** or higher.
   - Use **a common security group** to allow intra-cluster communication.

2. **Configure Security Group (Inbound Rules)**:
   - Allow **MySQL PXC ports** for intra-cluster communication.
   - Open **ports 3306, 4444, 4567, 4568** between nodes.

| **Port**  | **Protocol** | **Usage** |
|-----------|------------|-----------|
| **3306**  | TCP        | MySQL client connections |
| **4444**  | TCP        | State Snapshot Transfer (SST) |
| **4567**  | TCP/UDP    | Galera replication |
| **4568**  | TCP        | Incremental State Transfer (IST) |

---

## **🔹 Step 2: Install Percona XtraDB Cluster on All Nodes**
### **Install PXC on Ubuntu**
```sh
wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
sudo apt update
sudo apt install -y percona-xtradb-cluster-server
```

### **Install PXC on RHEL**
```sh
sudo yum install -y https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo yum install -y percona-xtradb-cluster
```

---

## **🔹 Step 3: Configure the First Node (`Node 1`)**
### **Modify PXC Configuration (`/etc/mysql/mysql.conf.d/mysqld.cnf` on Ubuntu)**
```ini
[mysqld]
wsrep_on=ON
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_cluster_name="PXC_Cluster"
wsrep_cluster_address="gcomm://192.168.1.10,192.168.1.11,192.168.1.12"
wsrep_node_name="pxc-node-1"
wsrep_node_address="192.168.1.10"

# SST Method
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth="pxc_sst_user:password"

# Binary Logging (Required for Replication)
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
```

### **Start the First Node (`Node 1`)**
```sh
sudo systemctl stop mysql
sudo systemctl start mysql@bootstrap.service
```

---

## **🔹 Step 4: Configure Additional Nodes (`Node 2` & `Node 3`)**
### **Modify PXC Configuration on Each Node**
Replace `wsrep_node_name` and `wsrep_node_address` in `/etc/mysql/mysql.conf.d/mysqld.cnf`:

🔹 **Node 2 (`192.168.1.11`)**
```ini
wsrep_node_name="pxc-node-2"
wsrep_node_address="192.168.1.11"
```
🔹 **Node 3 (`192.168.1.12`)**
```ini
wsrep_node_name="pxc-node-3"
wsrep_node_address="192.168.1.12"
```

### **Start Nodes 2 & 3**
Run on **Node 2 & Node 3**:
```sh
sudo systemctl stop mysql
sudo systemctl start mysql
```

---

## **🔹 Step 5: Verify the Cluster**
Run on **any node**:
```sh
mysql -uroot -p -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
```
Expected output:
```
+--------------------+-------+
| Variable_name     | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```
✔ **The cluster is running with 3 nodes!** 🎉

---

## **🔹 Step 6: Create a Database & Test Replication**
Run on **Node 1**:
```sh
mysql -uroot -p -e "CREATE DATABASE test_db;"
mysql -uroot -p -e "USE test_db; CREATE TABLE users (id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(100)) ENGINE=NDBCLUSTER;"
```
Run on **Node 2 or Node 3**:
```sh
mysql -uroot -p -e "SHOW DATABASES;"
```
✔ **`test_db` should appear on all nodes!** 🎉

---

## **🔹 Step 7: (Optional) Configure HAProxy Load Balancer**
To **distribute client connections** across nodes, install **HAProxy** on a separate EC2 instance:
```sh
sudo apt install -y haproxy  # (Ubuntu)
sudo yum install -y haproxy  # (RHEL)
```
Modify `/etc/haproxy/haproxy.cfg`:
```ini
frontend mysql_front
    bind *:3306
    mode tcp
    default_backend mysql_back

backend mysql_back
    mode tcp
    balance roundrobin
    server pxc-node-1 192.168.1.10:3306 check
    server pxc-node-2 192.168.1.11:3306 check
    server pxc-node-3 192.168.1.12:3306 check
```
Restart HAProxy:
```sh
sudo systemctl restart haproxy
```
✔ **Now, client applications can connect to the HAProxy IP, and traffic will be evenly distributed across nodes!** 🎉

---

## **🚀 Final Summary**
### **✅ What We Did**
✔ **Launched 3 AWS EC2 instances**  
✔ **Installed Percona XtraDB Cluster**  
✔ **Configured each node** for **multi-master replication**  
✔ **Verified replication** across all nodes  
✔ **Set up HAProxy** (optional) for **load balancing**  

### **✅ Cluster Architecture**
```
     +------------------------+
     | Load Balancer (HAProxy) |
     +------------------------+
                |
    --------------------------------
    |              |              |
+--------+    +--------+    +--------+
| Node 1 |<-->| Node 2 |<-->| Node 3 |
|  PXC   |    |  PXC   |    |  PXC   |
+--------+    +--------+    +--------+
```

---

## **🔥 Next Steps**
1️⃣ **Test HA by stopping a node** and checking cluster availability.  
2️⃣ **Scale Up** – Add more nodes dynamically.  
3️⃣ **Automate with Terraform & Ansible** for production setups.  

Would you like an **Ansible Playbook or Kubernetes Helm chart** for automating this? 🚀
