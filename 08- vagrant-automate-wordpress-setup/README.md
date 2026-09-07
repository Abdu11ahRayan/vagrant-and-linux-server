#  Automate WordPress Setup

> **Repository:** DevOps Vagrant & Linux Server Notes — Section 8
>
> Turning the full manual WordPress install from Section 6 — LAMP stack, database creation, `wp-config.php`, permissions — into one provisioning script. This is the bigger, more realistic automation challenge.

---

##  Why This Is Harder Than Automating the Website

Unlike a plain static site, WordPress automation needs to handle:

- Installing **multiple** services (Apache, MySQL, PHP + extensions), not just one.
- **Non-interactive** MySQL setup — `mysql_secure_installation` normally asks interactive questions, which breaks automation.
- Programmatically creating the database and user (no manual `mysql -u root -p` session).
- **Templating** `wp-config.php` with the right credentials automatically instead of hand-editing it.
- Handling the fact that provisioning may run **more than once** (idempotency) — the script shouldn't break if the database already exists.

---

##  The Full Provisioning Script

```bash
# provision-wordpress.sh
#!/bin/bash
set -e

DB_NAME="wordpress_db"
DB_USER="wp_user"
DB_PASS="StrongPassword123!"
DOC_ROOT="/var/www/html"

echo ">>> Setting MySQL root password non-interactively..."
export DEBIAN_FRONTEND=noninteractive
apt update

echo ">>> Installing LAMP stack..."
apt install -y apache2 mysql-server php libapache2-mod-php php-mysql php-curl php-gd php-xml php-mbstring wget

echo ">>> Starting and enabling services..."
systemctl enable apache2 mysql
systemctl start apache2 mysql

echo ">>> Creating WordPress database and user (idempotent)..."
mysql -u root <<MYSQL_SCRIPT
CREATE DATABASE IF NOT EXISTS ${DB_NAME};
CREATE USER IF NOT EXISTS '${DB_USER}'@'localhost' IDENTIFIED BY '${DB_PASS}';
GRANT ALL PRIVILEGES ON ${DB_NAME}.* TO '${DB_USER}'@'localhost';
FLUSH PRIVILEGES;
MYSQL_SCRIPT

echo ">>> Downloading WordPress if not already present..."
if [ ! -f "${DOC_ROOT}/wp-config-sample.php" ]; then
  cd /tmp
  wget -q https://wordpress.org/latest.tar.gz
  tar -xzf latest.tar.gz
  cp -r wordpress/* ${DOC_ROOT}/
fi

echo ">>> Generating wp-config.php..."
if [ ! -f "${DOC_ROOT}/wp-config.php" ]; then
  cp ${DOC_ROOT}/wp-config-sample.php ${DOC_ROOT}/wp-config.php
  sed -i "s/database_name_here/${DB_NAME}/" ${DOC_ROOT}/wp-config.php
  sed -i "s/username_here/${DB_USER}/" ${DOC_ROOT}/wp-config.php
  sed -i "s/password_here/${DB_PASS}/" ${DOC_ROOT}/wp-config.php
fi

echo ">>> Fixing ownership and permissions..."
chown -R www-data:www-data ${DOC_ROOT}
chmod -R 755 ${DOC_ROOT}

echo ">>> Restarting Apache..."
systemctl restart apache2

echo ">>> WordPress provisioning complete! Visit the site to finish setup in the browser."
```

---

##  Breaking Down the Key Automation Techniques

### `sed` for Config File Templating

Instead of manually editing `wp-config.php`, `sed -i "s/find/replace/"` swaps the placeholder values in-place:

```bash
sed -i "s/database_name_here/${DB_NAME}/" wp-config.php
```

This is the standard pattern for templating any plain-text config file in a shell script.

### Idempotent SQL — `IF NOT EXISTS`

```sql
CREATE DATABASE IF NOT EXISTS wordpress_db;
CREATE USER IF NOT EXISTS 'wp_user'@'localhost' IDENTIFIED BY '...';
```

Running the script twice (e.g. `vagrant provision` after a Vagrantfile tweak) won't error out trying to recreate an existing database/user.

### Idempotent File Checks

```bash
if [ ! -f "${DOC_ROOT}/wp-config.php" ]; then
  # only generate config if it doesn't already exist
fi
```

Prevents overwriting a working config (and losing any manual tweaks) on a re-provision.

---

##  Wiring It Into the Vagrantfile

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.hostname = "wordpress-server"
  config.vm.network "private_network", ip: "192.168.56.20"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  config.vm.provision "shell", path: "provision-wordpress.sh"
end
```

```bash
abdullah@DevOps:~/wordpress-vm$ chmod +x provision-wordpress.sh
abdullah@DevOps:~/wordpress-vm$ vagrant up
```

Once provisioning finishes, visit `http://192.168.56.20` — the WordPress install wizard (language, site title, admin account) is the only manual step left, since credentials can't be safely hardcoded into an automated script.

---

##  Full Rebuild Test

```bash
abdullah@DevOps:~/wordpress-vm$ vagrant destroy -f
abdullah@DevOps:~/wordpress-vm$ vagrant up
```

A properly automated script gets you from nothing to a working WordPress install (minus the final browser wizard) in one command, every time.

---

##  Quick Reference

| Technique | Purpose |
|---|---|
| `mysql -u root <<MYSQL_SCRIPT ... MYSQL_SCRIPT` | Run SQL non-interactively from a shell script (heredoc) |
| `CREATE DATABASE IF NOT EXISTS` | Idempotent — safe to run more than once |
| `sed -i "s/old/new/" file` | In-place text replacement — used to template `wp-config.php` |
| `if [ ! -f file ]; then ... fi` | Skip a step if it's already been done |
| `export DEBIAN_FRONTEND=noninteractive` | Prevents `apt` from hanging on interactive prompts |
| `vagrant destroy -f && vagrant up` | Full clean rebuild — the real automation test |

---

##  Key Takeaways

- Automating WordPress is harder than a static site because it involves **multiple services, a database, and config file templating** — not just one `apt install`.
- `sed -i` is the standard shell tool for templating plain-text config files like `wp-config.php` without manual editing.
- **Idempotency matters** — `IF NOT EXISTS` in SQL and file-existence checks in Bash mean the script can safely run more than once without breaking anything.
- Real secrets (DB passwords) hardcoded in a script are fine for a local practice VM, but in production these would come from a secrets manager or Ansible Vault — worth keeping in mind as you move further into the DevOps stack.
- The WordPress install wizard (site title, admin account) is intentionally left as the one manual step — you generally don't want to fully automate creating admin credentials in a public-facing script.

---

##  Topics Covered in This Repository (VAGRANT SECTION)

- [x] Vagrant VMs
- [x] Vagrant IP, RAM & CPU
- [x] Vagrant Sync Directories
- [x] Provisioning
- [x] Website Setup
- [x] WordPress Setup
- [x] Automate Website Setup
- [x] Automate WordPress Setup
- [ ] Copilot AI for Coding
- [ ] Multi VM Vagrantfile
- [ ] Systemctl & Tomcat 10

---

*Course: DecodingDevOps by Imran Teli (Udemy) | Status: ✅ Completed*
