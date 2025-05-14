Below is your **complete production-grade deployment plan** for streaming data replication from **MongoDB → Kafka (Debezium) → MySQL Galera Cluster**, using **3 Amazon Linux 2023 EC2 instances** without Docker. Each node will host MongoDB, Kafka + Debezium, and MySQL Galera.

---

## 🌐 Overall Architecture

| Component        | Node 1          | Node 2    | Node 3    |
| ---------------- | --------------- | --------- | --------- |
| MongoDB          | Primary         | Secondary | Secondary |
| Kafka Broker     | Broker 1        | Broker 2  | Broker 3  |
| Debezium Connect | Worker 1        | Worker 2  | Worker 3  |
| MySQL (Galera)   | Master (writer) | Reader    | Reader    |

---

## 🔧 Prerequisites on All EC2 Nodes

Before installing anything:

```bash
sudo dnf update -y
sudo dnf install -y git wget unzip curl net-tools tar java-17-amazon-corretto nc telnet
sudo hostnamectl set-hostname nodeX  # Replace X with 1, 2, or 3
```

Add each other’s private IPs to `/etc/hosts` on all 3 nodes:

```bash
# Example
10.0.0.11 node1
10.0.0.12 node2
10.0.0.13 node3
```

---

## 1️⃣ MongoDB Replica Set Setup (Primary + 2 Secondaries)

### 🔹 Install MongoDB 6.x on All Nodes

```bash
sudo tee /etc/yum.repos.d/mongodb.repo<<EOF
[mongodb-org-6.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2023/mongodb-org/6.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-6.0.asc
EOF

sudo dnf install -y mongodb-org
```

### 🔹 Configure `mongod.conf` on All Nodes

```yaml
replication:
  replSetName: rs0
net:
  bindIp: 0.0.0.0
```

### 🔹 Start & Enable Service

```bash
sudo systemctl enable --now mongod
```

### 🔹 Initialize Replica Set on `node1`

```bash
mongo
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "node1:27017" },
    { _id: 1, host: "node2:27017" },
    { _id: 2, host: "node3:27017" }
  ]
})
```

---

## 2️⃣ Install MySQL Galera Cluster (Percona XtraDB Cluster 8)

### 🔹 Add Percona Repo

```bash
sudo dnf install -y https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo percona-release enable pxc-80 release
sudo dnf install -y percona-xtradb-cluster
```

### 🔹 Configure Galera on All Nodes

Edit `/etc/my.cnf.d/wsrep.cnf`:

```ini
[mysqld]
wsrep_on=ON
wsrep_provider=/usr/lib64/galera4/libgalera_smm.so
wsrep_cluster_name="pxc-cluster"
wsrep_cluster_address="gcomm://node1,node2,node3"
wsrep_node_address="nodeX"  # Replace with actual hostname
wsrep_node_name="nodeX"
wsrep_sst_method=rsync
pxc_strict_mode=ENFORCING
binlog_format=ROW
default_storage_engine=InnoDB
```

### 🔹 Bootstrap Cluster from First Node

On node1 only:

```bash
sudo systemctl start mysql@bootstrap.service
```

On node2 and node3:

```bash
sudo systemctl start mysql
```

Check status:

```bash
mysql -uroot -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
```

---

## 3️⃣ Kafka 3.6.1 Cluster Setup (on All 3 Nodes)

### 🔹 Download & Install Kafka

```bash
cd ~
wget https://downloads.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz
tar -xvzf kafka_2.13-3.6.1.tgz
mv kafka_2.13-3.6.1 kafka
```

### 🔹 Configure Kafka (server.properties)

```ini
broker.id=X  # 1, 2, 3
listeners=PLAINTEXT://0.0.0.0:9092
log.dirs=/tmp/kafka-logs
controller.quorum.voters=1@node1:9093,2@node2:9093,3@node3:9093
process.roles=broker,controller
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
node.id=X
```

Start Kafka:

```bash
kafka/bin/kafka-storage.sh format -t $(uuidgen) -c kafka/config/kraft/server.properties
nohup kafka/bin/kafka-server-start.sh kafka/config/kraft/server.properties > kafka.log 2>&1 &
```

---

## 4️⃣ Install Debezium Kafka Connect (on all 3)

### 🔹 Setup Connect

```bash
mkdir ~/debezium
cd ~/debezium
wget https://downloads.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz
tar -xvzf kafka_2.13-3.6.1.tgz
mv kafka_2.13-3.6.1 connect
```

### 🔹 Add Debezium MongoDB Connector

```bash
mkdir -p connect/plugins
cd connect/plugins
wget https://repo1.maven.org/maven2/io/debezium/debezium-connector-mongodb/2.5.0.Final/debezium-connector-mongodb-2.5.0.Final-plugin.tar.gz
tar -xvzf *.tar.gz
```

### 🔹 Configure `connect-distributed.properties`

```ini
bootstrap.servers=node1:9092,node2:9092,node3:9092
group.id=connect-cluster
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
plugin.path=/home/ec2-user/debezium/connect/plugins
config.storage.topic=connect-configs
offset.storage.topic=connect-offsets
status.storage.topic=connect-status
```

Start each connect node:

```bash
nohup connect/bin/connect-distributed.sh connect/config/connect-distributed.properties > connect.log 2>&1 &
```

---

## 5️⃣ Register MongoDB Source Connector

```bash
curl -X POST http://node1:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "mongo-source",
    "config": {
      "connector.class": "io.debezium.connector.mongodb.MongoDbConnector",
      "mongodb.hosts": "rs0/node1:27017,node2:27017,node3:27017",
      "mongodb.name": "mongo_cluster",
      "database.include.list": "inventory",
      "collection.include.list": "inventory.products",
      "tasks.max": "1"
    }
  }'
```

---

## 6️⃣ Register MySQL Sink Connector (on Debezium)

Install JDBC sink connector (on node1):

```bash
cd ~/debezium/connect/plugins
wget https://packages.confluent.io/maven/io/confluent/kafka-connect-jdbc/10.7.1/kafka-connect-jdbc-10.7.1.jar
```

Restart Connect worker if needed, then register connector:

```bash
curl -X POST http://node1:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "mysql-sink",
    "config": {
      "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
      "connection.url": "jdbc:mysql://node1:3306/testdb?user=root&password=yourpassword",
      "topics": "mongo_cluster.inventory.products",
      "insert.mode": "insert",
      "auto.create": "true"
    }
  }'
```

---

## 🛡️ High Availability & Recovery

| Component     | Recovery Plan                                          |
| ------------- | ------------------------------------------------------ |
| MongoDB       | Automatic primary failover (replica set election)      |
| MySQL Galera  | Automatic node failure handling, failover via ProxySQL |
| Kafka         | Broker replication & quorum voting (KRaft)             |
| Kafka Connect | Distributed mode, restart worker on failure            |

Use `systemd` or `monit` to auto-restart Kafka/Debezium.

---

## ✅ Result

* **MongoDB Replica Set** streams changes into Kafka via **Debezium**
* **Kafka Connect** (with JDBC sink) writes to **MySQL Galera**
* System tolerates node failures and auto-recovers

---


