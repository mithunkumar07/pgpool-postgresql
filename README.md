
# Pgpool-II Setup Guide

## **What is Pgpool-II?**

Pgpool-II 🖧 is a middleware that can run on Linux and Solaris between applications and **PostgreSQL** databases.

Its main features include:
- 🚀 **Load balancing**
- 📦 **Replication**
- 🔗 **Connection pooling**
- 🛠️ **Automatic failover**
- 🔄 **Online recovery**

---

### **Streaming Replication** 🔁

Pgpool-II supports replication using its own capabilities or PostgreSQL's **streaming replication**, which ships transaction logs (WALs) from the primary PostgreSQL to standby instances.

![Streaming Replication](https://cdn.hashnode.com/res/hashnode/image/upload/v1717155835808/933f440b-f58b-4c4d-9f2f-119db160f68e.png?auto=compress,format&format=webp)

---

### **Load Balancing and Connection Pooling** ⚖️

- **Scale-out**: Increase database processing capacity by adding servers. Pgpool-II efficiently distributes read-only queries to balance workloads.
- **Connection Pooling**: Reduces overhead by reusing existing connections.

![Load Balancing](https://cdn.hashnode.com/res/hashnode/image/upload/v1717155874993/f25dc0f6-5d0b-4b5f-aed5-081e0b23d5e1.png?auto=compress,format&format=webp)

---

### **Steps to Set Up and Implement Pgpool-II**

#### **1. Set Up Master-Slave Architecture for PostgreSQL Servers** 🛠️

- Master handles read/write operations, and slaves handle read-only tasks.
- Uses **streaming replication** to keep the slave up-to-date with WAL records.

![Master-Slave](https://cdn.hashnode.com/res/hashnode/image/upload/v1717155920593/73f0fada-c9cd-4bd4-b734-a94ca9bdd0eb.png?auto=compress,format&format=webp)

#### **Prerequisites** 📋
- PostgreSQL installed on both master and slave.
- Both servers should communicate with each other.

#### **Configure the Master Server**
1. **Enable Networking**:
   ```bash
   vim /etc/postgresql/16/main/postgresql.conf
   listen_addresses = '*'
   ```
2. **Create Replication User**:
   ```sql
   CREATE USER repuser WITH LOGIN REPLICATION;
   ```
3. **Allow Remote Access**:
   ```bash
   vim /etc/postgresql/16/main/pg_hba.conf
   host replication repuser 172.31.0.0/16 md5
   ```
4. **Restart the Master**:
   ```bash
   systemctl restart postgresql@16-main
   ```

...

#### **Configure the Slave Server**

1. Stop PostgreSQL Service:
   ```bash
   systemctl stop postgresql@16-main
   ```
2. Backup & Initialize Data Directory:
   ```bash
   mv main main_backup
   mkdir main
   ```
3. Perform Base Backup:
   ```bash
   pg_basebackup -h <master_ip> -U repuser -D /var/lib/postgresql/16/main/ -R
   ```
4. Change Permissions & Restart:
   ```bash
   chown -R postgres:postgres /var/lib/postgresql/16/main
   systemctl restart postgresql@16-main
   ```

#### **Verification**
Master:
```sql
SELECT * FROM pg_stat_replication;
```
Slave:
```sql
SELECT * FROM pg_stat_wal_receiver;
```

---

### **Pgpool-II Setup** 🔧

#### **Install Pgpool-II**
```bash
apt install pgpool2
```

#### **Configuration**
Update `/etc/pgpool2/pgpool.conf`:
```bash
listen_addresses = '*'
port = 9999
backend_hostname0 = '<master_ip>'
backend_hostname1 = '<slave_ip>'
```

---

This concludes the guide for Pgpool-II setup with PostgreSQL!
