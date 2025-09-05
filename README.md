# 🧨 DB-warROOOOOOM

**No auth. No mercy.**  
This is my war log for **database exploitation**.

<div align="center">
  <img src="https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExbWU4NGRyMTN6bmhvYWhvam85aXNiZnhjaWU5a3pvNm83dmEzODBpbSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/3o6Mbi9e1UmeCCyO0E/giphy.gif" width="600"/>
</div>  

---

# 🗺️ Overview
A curated collection of attack flows, field notes, and hands-on exploits  
covering misconfigurations, weak auth, privilege escalation, and RCE across different DBMS:  

- Redis  
- PostgreSQL  
- ...more to come. 

---

# 🧰 Tools in Use
- `psql`, `mysql`, `redis-cli`  
- `nmap` + NSE (db-specific scripts)  
- `netcat` (banner grab, raw payloads)  
- `hydra` / `medusa` for brute-force  
- `Metasploit` for replication/RCE modules  
- Custom PoCs & rogue servers  
- `tshark`/Wireshark for protocol inspection  

---

# 🧪 Attack Philosophy
- Manual attacks only  
- No “auto-exploit” one-click toys  
- Start from **banner grab**, end with **shell**  
- Emphasis on chaining: weak creds → misconfig → file/system access → RCE  

---

# ❤️ Author
Crafted by @DanteSec  
Born out of frustration, refined by obsession.  
