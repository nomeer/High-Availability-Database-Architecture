# Highly Available Web Architecture: HAProxy + MariaDB Galera Cluster + Web2py

## Overview
A fully fault-tolerant web application architecture demonstrating zero-downtime database failover and advanced cluster recovery. This project integrates a Python-based web framework with a 3-node MariaDB Galera multi-master cluster, using HAProxy as a TCP-layer traffic controller. 



## Architecture & Deployment
* **Database Tier:** 3-Node MariaDB Galera Cluster providing synchronous, multi-master replication. 
* **Load Balancing:** HAProxy configured for TCP-mode (`mode tcp`) round-robin load balancing across the database nodes.
* **Application Tier:** Web2py framework communicating via Python's `pymysql` driver through the HAProxy frontend.
* **Process Management:** PM2 utilized to run the application persistently as a background daemon.

## Core Technical Challenges Solved

### 1. Cluster Bootstrapping & Crash Recovery
Mastered the Galera Primary Component architecture. Successfully deployed the `galera_new_cluster` command to establish the initial master node. Engineered recovery procedures for full-cluster crashes by manipulating the `/var/lib/mysql/grastate.dat` file to force `safe_to_bootstrap: 1`, successfully electing a surviving node to rebuild the cluster from scratch.

### 2. Network-Layer Health Checks
Engineered a secure, password-less health check user (`haproxy_check`) allowing HAProxy to continuously ping database nodes without exposing application data or triggering MariaDB connection lockouts.

### 3. TCP Timeout Optimization
Resolved "Error 2013 (Lost connection)" by configuring precise HAProxy client and server TCP timeouts, ensuring long-running database queries remain stable during network routing.

### 4. Strict Hostname Resolution
Overcame MariaDB's strict reverse-DNS connection rejections by explicitly mapping the proxy's VIP application users to specific fully qualified domain names (FQDNs) rather than relying on wildcard IP rules.

### 5. Security & Firewall Configuration
Secured the infrastructure by implementing strict network-layer firewall rules (UFW/iptables) to isolate the database tier from the public internet.
* **Public Access:** Restricted public ingress strictly to the web application and monitoring interfaces.
* **Internal Subnet Isolation:** Locked down direct MariaDB communication (Port 3306) and Galera cluster synchronization ports (4567, 4568, 4444) strictly to the internal network IPs.
* **Proxy Layer Security:** Isolated the HAProxy routing port (Port 3307) to localhost and approved application servers, completely neutralizing direct external database connection attempts.
