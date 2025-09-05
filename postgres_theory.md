# 📖 PostgreSQL — Theory & Attack Surface

## 🗺️ Overview
PostgreSQL is an advanced open-source relational database system.  
It’s widely used in enterprise environments, supports complex queries, and extends via plugins, procedural languages, and custom functions.  

From a security perspective, its **flexibility is also its attack surface**:  
- Multiple authentication methods (`trust`, `password`, `md5`, `scram-sha-256`, `cert`)  
- Remote connections controlled by `pg_hba.conf`  
- Extensions and languages (`plpgsql`, `plpythonu`, `plperl`, UDFs)  
- Replication, foreign data wrappers, and database links  

---

## 🔑 Authentication & Access Control
- **Methods (in pg_hba.conf)**:  
  - `trust`: no password → **total bypass**  
  - `password`: cleartext over the wire (unsafe without TLS)  
  - `md5`: hashed but crackable offline  
  - `scram-sha-256`: modern and preferred  
  - `cert`: SSL client certificates  

- **Role system**:  
  - Roles can act as users or groups.  
  - Attributes: `SUPERUSER`, `CREATEROLE`, `REPLICATION`.  
  - Dangerous misconfigs: too many superusers, PUBLIC grants.  

---

## 🧩 Extensions & Procedural Languages
- **Extensions** expand functionality (`\dx`).  
- Risky ones:  
  - `dblink`: connect to other DBs (pivoting).  
  - `file_fdw`: read files from server FS.  
  - `plpythonu`, `plperl`: unsafe procedural code execution.  
- **C extensions / UDFs**: allow arbitrary native code → full RCE.  

---

## 📂 Filesystem Interaction
- **COPY command**:  
  ```sql
  COPY table TO '/tmp/dump.csv';
  COPY (SELECT 'data') TO '/etc/cron.d/rootjob';
  ```
- Allows **file read/write** if role is superuser or has permissions.  
- Abuse: write SSH keys, cronjobs, or backdoor scripts.  

---

## 🔌 Replication & Links
- Replication accounts often over-privileged.  
- `dblink` can leak data from other DBs if not isolated.  
- Abuse of **logical replication** may expose credentials or schema.  

---

## 🔐 Encryption & TLS
- PostgreSQL supports SSL/TLS for client connections.  
- Common flaws:  
  - SSL not enforced (`ssl = off`)  
  - Self-signed certs accepted without validation  
  - Downgrade to plaintext connections  

---

## 🕵️ Monitoring & Logging
- **pg_stat_activity**: shows active sessions (can leak creds in queries).  
- Logs: depending on `log_statement` may expose sensitive SQL.  
- Audit: `pg_audit` is available but often disabled.  

---

## 🎯 Common Attack Paths
1. **Exposed port 5432 with weak creds**  
2. **pg_hba.conf misconfig (`trust` / `password`)**  
3. **SUPERUSER role abuse (RCE via COPY/PROGRAM)**  
4. **Extension abuse (dblink/file_fdw)**  
5. **UDF in C → load malicious shared object**  
6. **Replication or backup accounts used for pivoting**  

---

## 🛡️ Hardening Recommendations
- Restrict `listen_addresses` → avoid world exposure.  
- Enforce `scram-sha-256` authentication.  
- Limit `SUPERUSER` to bare minimum.  
- Disable/remove unneeded extensions.  
- Enable TLS with client certs.  
- Enable auditing (`pg_audit`).  
- Monitor queries and role changes.  
