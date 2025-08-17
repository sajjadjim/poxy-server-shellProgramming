# Ubuntu Proxy Server Project (Squid) — Shell-Based Setup

A complete, ready-to-run project to **install, configure, and manage a secure HTTP/HTTPS forward proxy** on Ubuntu (ideal for VirtualBox, WSL, or bare metal). It uses **Squid** with **Basic Authentication**, **allow/block lists**, and handy **Bash management scripts**.

---

## ✨ Features

- **One-command setup** (`setup_proxy.sh`): installs Squid and applies a hardened configuration.
- **User Authentication**: Basic auth via `htpasswd` (Apache2-utils).
- **Domain Filtering**:
  - **Blocklist**: deny specific domains.
  - **Allowlist**: optionally restrict to only listed domains.
- **Testing Tools**: `test_proxy.sh` (HTTP/HTTPS checks).
- **User Management**: `add_user.sh` to add/update passwords; manual delete example.
- **Logging & Monitoring**: `show_logs.sh` tails `access.log` in a readable format.
- **Maintenance**: `rotate_logs.sh` for safe log rotation.
- **Clean Uninstall**: `uninstall_proxy.sh` removes packages and leaves backups/logs.

---

## 🗂️ Project Structure

```
ubuntu-proxy-project/
├─ setup_proxy.sh          # One-click installer & configurator
├─ proxy.conf              # Squid configuration template
├─ allowlist.txt           # Domains allowed (optional; one per line)
├─ blocklist.txt           # Domains blocked (one per line)
├─ add_user.sh             # Add or update a proxy user
├─ add_block.sh            # Block a domain immediately
├─ test_proxy.sh           # Quick connectivity tests via curl
├─ show_logs.sh            # Pretty tail of access.log
├─ rotate_logs.sh          # Rotate Squid logs
├─ uninstall_proxy.sh      # Clean removal
└─ README.md               # This file
```

---

## ✅ Requirements

- **Ubuntu** 20.04 / 22.04 / 24.04 (works in VirtualBox or metal)
- **Internet access** to install packages
- **sudo privileges** for installation and service control
- Recommended VM resources: **2 GB RAM**, **10 GB disk**

---

## ⚙️ Quick Start

1. **Clone / Unzip** the project directory on your Ubuntu VM.
2. **Open a terminal** in the project directory and make scripts executable:
   ```bash
   chmod +x *.sh
   ```
3. **Run setup** (uses sudo, and creates your first user):
   ```bash
   sudo ./setup_proxy.sh
   ```
   - Port: **3128**
   - Config files installed to `/etc/squid/`
   - Prompts you to create a **username/password**

4. **Test the proxy**:
   ```bash
   ./test_proxy.sh
   ```
   Or manually with curl:
   ```bash
   curl --proxy http://USER:PASS@localhost:3128 http://example.com -I
   curl --proxy http://USER:PASS@localhost:3128 https://ifconfig.me
   ```

---

## 🔐 User Management

### Add or Update a User
Use the helper script (creates or changes password):
```bash
sudo ./add_user.sh <username>
# Example:
sudo ./add_user.sh student2
```

### Delete a User (manual example)
```bash
sudo htpasswd -D /etc/squid/passwd <username>
# Example:
sudo htpasswd -D /etc/squid/passwd student2
sudo systemctl reload squid
```

---

## 🌐 Allowlist & Blocklist

- **Block domain** immediately:
  ```bash
  sudo ./add_block.sh example.com
  ```

- **Edit lists manually**:
  ```bash
  sudo nano /etc/squid/allowlist.txt
  sudo nano /etc/squid/blocklist.txt
  sudo systemctl reload squid
  ```

> **Policy logic**  
> 1) If a domain is in the **blocklist**, access is denied.  
> 2) If the **allowlist** contains entries, only those domains are allowed for authenticated users.  
> 3) If the allowlist is **empty**, authenticated users can access any domain (except blocked ones).

---

## 🧪 Testing

Scripted test:
```bash
./test_proxy.sh
```
Manual tests:
```bash
curl --proxy http://USER:PASS@localhost:3128 http://example.com -I
curl --proxy http://USER:PASS@localhost:3128 https://ifconfig.me
```

Browser test (Firefox/Chrome):
- Set **HTTP/HTTPS Proxy** to `localhost`, **Port** `3128`
- When prompted, enter the created **username/password**

---

## 📜 Logs & Monitoring

- Live logs (pretty printer):
  ```bash
  ./show_logs.sh
  ```
- Raw log tail:
  ```bash
  sudo tail -f /var/log/squid/access.log
  ```
- Rotate logs:
  ```bash
  sudo ./rotate_logs.sh
  ```

---

## 🔧 Service Control (systemd)

```bash
sudo systemctl status squid
sudo systemctl restart squid
sudo systemctl reload squid
sudo systemctl enable squid
sudo systemctl stop squid
```

---

## 🚪 Firewall & Port

Squid listens on **3128**. If you use UFW:
```bash
sudo ufw allow 3128/tcp
sudo ufw status
```

---

## 🔐 Security Notes

- Uses **Basic Auth** (over HTTPS CONNECT is fine; for LAN ensure clients trust your network).
- No **SSL intercept** (no SSL bump) by default — avoids certificate issues.
- Keep `/etc/squid/passwd` **root/proxy-owned** and **640** perms (handled by setup).
- Use allowlist for stricter egress control in labs/classrooms.

---

## 🧹 Uninstall

```bash
sudo ./uninstall_proxy.sh
```
- Stops and purges Squid packages
- Leaves logs (`/var/log/squid/`) and your old backups of `squid.conf`

---

## 🧩 Troubleshooting

- **Auth fails repeatedly**
  - Verify user exists: `sudo grep '^username:' /etc/squid/passwd`
  - Reset password: `sudo ./add_user.sh username`
  - Reload: `sudo systemctl reload squid`

- **Can’t connect to proxy**
  - Check service: `sudo systemctl status squid`
  - Check port: `ss -ltnp | grep 3128`
  - Firewall: `sudo ufw allow 3128/tcp`

- **Blocklist/Allowlist changes not applied**
  - Ensure one domain per line (no `http://` or slashes)
  - Reload Squid: `sudo systemctl reload squid`

- **DNS problems in VM**
  - Try `sudo systemd-resolve --flush-caches`
  - Ensure VM has internet access (NAT/Bridged mode)

---

## 📘 How It Works (Architecture)

- **Squid** handles proxying and caching; listens on **3128**.
- **basic_ncsa_auth** validates credentials against `/etc/squid/passwd`.
- **ACLs** implement policy:
  - `blocked_sites` → `/etc/squid/blocklist.txt`
  - `allowed_sites` → `/etc/squid/allowlist.txt`
  - `authenticated` → `proxy_auth REQUIRED`
- **Scripts** automate installation, config templating, and maintenance.

---

## 📄 License & Attribution

- Squid is licensed under GPL (see Squid docs).  
- Project scripts are provided **for educational use**; adapt as needed for your environment.

---

## 🙋 FAQ

**Q: Can I change the port?**  
A: Yes. Edit `/etc/squid/squid.conf` `http_port 3128` → your port, then `sudo systemctl restart squid`.

**Q: Can I allow a specific subnet without auth?**  
A: Add an ACL for your subnet and allow it before the `deny all` rule. Example:
```conf
acl mylan src 192.168.56.0/24
http_access allow mylan
```
Then reload Squid.

**Q: Does this intercept HTTPS?**  
A: No. This passes **CONNECT** through after authentication (no SSL bump).

**Q: Where are logs?**  
A: `/var/log/squid/access.log` and `/var/log/squid/cache.log`.

---

Happy proxying! If you want this README embedded into your existing project folder, tell me your path and I’ll place it there for you.
