# 🟥 Redis — Theory & Attack Surface

## 1. What is Redis?
- **Redis = REmote DIctionary Server**
- Open-source, in-memory **key–value store**
- Used as: database, cache, message broker, session store
- Supports persistence:
  - **RDB** snapshots
  - **AOF** (Append Only File)
- Default port: **6379** (26379 for Sentinel, 16379+ for cluster)

---

## 2. Core Concepts
- **In-memory**: ultra-fast, all data lives in RAM
- **Key–value model**: flexible data types
  - `string`
  - `hash`
  - `list`
  - `set`
  - `zset` (sorted set)
  - `stream`
- **Persistence**: optional, but can dump memory → disk

---

## 3. Security Model
- **Pre-Redis 6**:
  - Only `requirepass` → single password, weak protection
  - Historically many instances deployed with **no password**
- **Redis 6+**:
  - **ACLs**: multiple users, granular permissions
  - Categories like `@read`, `@write`, `@dangerous`
  - Replace legacy `rename-command` trick
- **Protected Mode**:
  - Default ON for new installs
  - Rejects external connections unless `bind` + `auth` properly configured

---

## 4. Attack Surface
- **No authentication**: exposed instances are instant targets
- **Weak passwords**: easily bruteforced
- **Dangerous commands**:
  - `CONFIG SET` + `SAVE` → write files (SSH keys, cronjobs, webshells)
  - `REPLICAOF` → rogue master → malicious module injection
  - `MODULE LOAD` → load `.so` with arbitrary commands
  - `EVAL` → Lua scripts (abuse if unrestricted)
- **Cluster / Sentinel**:
  - Extra ports exposed (26379, 16379+)
  - Potential to hijack topology or extract internal IPs
- **SSRF chaining**:
  - If Redis isn’t exposed, can still be reached via SSRF using `gopher://`

---

## 5. Common Exploitation Paths
1. **Memory → File Write**  
   - Abuse `CONFIG SET dir/dbfilename` + `SAVE`  
   - Write to `.ssh/authorized_keys`, `/etc/cron.d/`, or webroot
2. **File Write → RCE**  
   - Login via SSH, cronjob execution, or webshell
3. **Replication Attack**  
   - Force victim to replicate from attacker’s rogue Redis
   - Drop malicious `.so` module → RCE
4. **Module Abuse**  
   - Direct `MODULE LOAD` if ACL misconfigured
5. **Lua Exploits**  
   - Use `EVAL` for logic abuse
   - Debian/Ubuntu bug (**CVE-2022-0543**) → Lua sandbox escape → RCE
6. **Data Looting**  
   - Dump tokens, sessions, credentials from keys

---

## 6. Real-World Misconfigurations
- Redis open on **0.0.0.0:6379** with no auth
- Weak or default passwords
- `CONFIG`, `MODULE`, `REPLICAOF` left enabled
- Sentinel accessible from outside
- Developers using Redis as a quick session store → sensitive data exposed

---

## 7. Indicators of Risk
- `protected-mode: no`  
- `ACL LIST` → `default` user has `on +@all`  
- `CONFIG SET` / `SAVE` / `MODULE LOAD` allowed  
- `role: master` with connected slaves (replication enabled)  
- Debian/Ubuntu + Redis < patched versions → CVE-2022-0543 risk

---

## 8. Mitigations (for reporting)
- Never expose Redis directly to the internet
- Enforce strong auth & ACLs (`-@dangerous`)
- Use TLS for cluster/Sentinel communications
- Limit bind address to localhost or private VLAN
- Firewall rules (iptables, security groups)
- Monitor `ACL LOG`, `commandstats` for suspicious use

---

## 9. Key Takeaways for Pentesters
- Redis is **not just a DB** → it’s a memory engine with file system hooks
- Exploits flow as:  
  **Talk → Write → Where → Execute**
- Most “exploits” are really **misconfigurations**
- Always think in **chains**: Redis → creds → pivot → full environment compromise

---
