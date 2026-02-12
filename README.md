# PostgreSQL HA Lab

Production-style PostgreSQL high-availability cluster powered by **Patroni** and **etcd**, fully containerized with Docker Compose.

This project explores distributed consensus, leader election, and automatic failover in a real PostgreSQL cluster.

---

# 🎯 Project Goal

I built this lab to deeply understand how PostgreSQL high availability works in real-world production environments.

Specifically, I wanted to explore:

* How Patroni manages leader election
* How etcd stores distributed cluster state
* How streaming replication behaves under failure
* What actually happens during failover
* How a failed node rejoins the cluster

This is not just a “run docker-compose” demo — it’s a hands-on distributed systems experiment.

---

# 🏗 Architecture

### Cluster Components

* 3 PostgreSQL nodes (managed by Patroni)
* 1 etcd node (Distributed Configuration Store)
* Docker Compose for orchestration
* Local network simulation

```
           +------------------+
           |      etcd        |
           |  (Consensus DCS) |
           +--------+---------+
                    |
        --------------------------------
        |              |               |
   +----+----+    +----+----+     +----+----+
   | pg-node1|    | pg-node2|     | pg-node3|
   | Leader  |    | Replica |     | Replica |
   +---------+    +---------+     +---------+
```

All nodes run as isolated containers but simulate a distributed cluster.

---

# 🚀 Quick Start

```bash
git clone https://github.com/yourusername/pg-ha-lab.git
cd pg-ha-lab
docker-compose up -d
```

Check cluster status:

```bash
docker ps
```

Check Patroni API:

```bash
curl http://localhost:8008
```

---

# 🧪 Devlog — What I Explored

## 1️⃣ Initial Cluster Boot

When the cluster starts:

* Patroni instances compete for leadership
* etcd stores cluster metadata
* One node initializes the database
* Other nodes clone from the leader

Observation:
Leader election happens very quickly — within seconds.

---

## 2️⃣ Testing Failover

Experiment:

```bash
docker stop pg-node1
```

Results:

* etcd detected leader unavailability
* Replica promoted automatically
* Cluster recovered in ~5 seconds

Insight:
Failover is not instant — there is a small election window.
This delay is configurable via TTL and loop_wait in Patroni.

---

## 3️⃣ Restarting the Old Leader

After restarting the stopped node:

* It did NOT become leader
* It rejoined as a replica
* Replication resumed automatically

Key Learning:
Leadership is determined by consensus state in etcd, not startup order.

---

## 4️⃣ Breaking Replication on Purpose

I manually interrupted replication to observe behavior.

Result:

* Patroni detected unhealthy state
* Node marked as replica but not promotable
* Logs provided detailed state diagnostics

Lesson:
Observability is critical in distributed systems.

---

# 🔬 Technical Concepts Demonstrated

* Distributed consensus (etcd)
* Leader election
* Streaming replication
* WAL shipping
* Replica promotion
* Cluster reconfiguration
* Split-brain prevention mechanisms

---

# ⚙️ Key Configuration Highlights

* Replication slots enabled
* REST API exposed for health checks
* Synchronous mode optional (future enhancement)
* Custom PostgreSQL parameters applied via Patroni

---

# 📊 What This Project Taught Me

* High availability is about coordination, not just replication.
* Distributed state management is harder than local redundancy.
* Failover timing is a tradeoff between availability and split-brain safety.
* Logs are essential to understanding cluster behavior.
* Docker is excellent for simulating distributed environments locally.

---

# 🔮 Future Improvements


* 3-node etcd cluster (true distributed consensus)
* HAProxy for connection routing
* PgBouncer integration
* Prometheus + Grafana monitoring
* Chaos testing (automated failure injection)
* Kubernetes deployment version

---

# 🧠 Why This Project Matters

High-availability PostgreSQL is widely used in:

* FinTech
* SaaS platforms
* E-commerce
* Enterprise systems

Understanding Patroni + etcd architecture provides real-world infrastructure knowledge used in production environments.

---

# 📄 License

MIT

---
