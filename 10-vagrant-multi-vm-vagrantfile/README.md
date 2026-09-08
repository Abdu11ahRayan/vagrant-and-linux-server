#  Multi VM Vagrantfile

> **Repository:** DevOps Vagrant & Linux Server Notes — Section 10
>
> Running multiple VMs from a single Vagrantfile — the setup that actually mirrors real infrastructure, where a web server and a database server live on separate machines.

---

##  Why Multiple VMs?

So far, every Vagrantfile in this repo has defined **one** VM. Real environments rarely look like that — a typical app has a web/app server talking to a separate database server, maybe a load balancer, maybe a cache layer. A multi-VM Vagrantfile lets you practice that architecture locally.

```
        ┌─────────────┐        ┌─────────────┐
        │  web (VM 1) │──────▶│   db (VM 2) │
        │ 192.168.56.10│        │192.168.56.11│
        └─────────────┘        └─────────────┘
```

---

##  Basic Multi-VM Vagrantfile Structure

Use `config.vm.define` to declare each VM as its own named block inside one Vagrantfile:

```ruby
Vagrant.configure("2") do |config|

  # ---------- Web Server VM ----------
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/focal64"
    web.vm.hostname = "webserver"
    web.vm.network "private_network", ip: "192.168.56.10"

    web.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
      vb.name = "web-vm"
    end

    web.vm.provision "shell", path: "provision-web.sh"
  end

  # ---------- Database Server VM ----------
  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/focal64"
    db.vm.hostname = "dbserver"
    db.vm.network "private_network", ip: "192.168.56.11"

    db.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
      vb.name = "db-vm"
    end

    db.vm.provision "shell", path: "provision-db.sh"
  end

end
```

>  Note the pattern: `config.vm.define "name" do |alias|` — then every setting inside uses `alias.vm...` instead of `config.vm...`.

---

##  Managing Multiple VMs

```bash
abdullah@DevOps:~/multi-vm$ vagrant up                # Bring up ALL VMs defined in the Vagrantfile
abdullah@DevOps:~/multi-vm$ vagrant up web              # Bring up only the "web" VM
abdullah@DevOps:~/multi-vm$ vagrant ssh web              # SSH into a specific VM by name
abdullah@DevOps:~/multi-vm$ vagrant ssh db
abdullah@DevOps:~/multi-vm$ vagrant halt db               # Stop a specific VM
abdullah@DevOps:~/multi-vm$ vagrant destroy web -f          # Destroy a specific VM
abdullah@DevOps:~/multi-vm$ vagrant status                   # Show status of ALL VMs
Current machine states:
web    running (virtualbox)
db     running (virtualbox)
```

---

##  Making VMs Talk to Each Other

Since both VMs are on the same `private_network`, they can reach each other by IP directly:

```bash
abdullah@DevOps:~/multi-vm$ vagrant ssh web
vagrant@webserver:~$ ping 192.168.56.11        # Reaches the db VM
vagrant@webserver:~$ mysql -h 192.168.56.11 -u wp_user -p    # App connects to the remote DB
```

This is exactly how a real web app connects to a separate database server — just at practice scale.

---

##  Practical Example — Web + DB Split

**provision-web.sh:**
```bash
#!/bin/bash
set -e
apt update
apt install -y apache2 php libapache2-mod-php php-mysql
systemctl enable apache2
systemctl restart apache2
```

**provision-db.sh:**
```bash
#!/bin/bash
set -e
apt update
apt install -y mysql-server
systemctl enable mysql
systemctl start mysql

mysql -u root <<MYSQL_SCRIPT
CREATE DATABASE IF NOT EXISTS appdb;
CREATE USER IF NOT EXISTS 'appuser'@'%' IDENTIFIED BY 'AppPass123!';
GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
MYSQL_SCRIPT
```

>  Note `'appuser'@'%'` (any host) instead of `'appuser'@'localhost'` — the database needs to accept connections **from the web VM**, not just from itself. MySQL also needs `bind-address` in its config updated to allow remote connections (`/etc/mysql/mysql.conf.d/mysqld.cnf`), which a full script would also handle.

```bash
abdullah@DevOps:~/multi-vm$ vagrant up
# Both VMs provision automatically; web VM's app can now reach db VM's MySQL
```

---

##  Provisioning Shared Across All VMs

If a step applies to **every** VM (e.g. a common package), define it outside any `config.vm.define` block — it runs for all of them:

```ruby
Vagrant.configure("2") do |config|

  # Runs on every VM defined below
  config.vm.provision "shell", inline: "apt update && apt install -y curl vim"

  config.vm.define "web" do |web|
    # ...
  end

  config.vm.define "db" do |db|
    # ...
  end

end
```

---

##  Quick Reference

| Command | Purpose |
|---|---|
| `config.vm.define "name" do \|alias\| ... end` | Declare one VM within a multi-VM Vagrantfile |
| `vagrant up` | Bring up all defined VMs |
| `vagrant up <name>` | Bring up only one named VM |
| `vagrant ssh <name>` | SSH into a specific VM |
| `vagrant halt <name>` / `vagrant destroy <name>` | Stop/destroy a specific VM |
| `vagrant status` | Show status of every VM in the Vagrantfile |
| Shared `config.vm.provision` (outside `define`) | Runs on every VM |

---

##  Key Takeaways

- `config.vm.define "name" do |alias| ... end` is the core pattern for multiple VMs in one Vagrantfile — everything inside uses the block's alias, not `config`, to scope settings to that specific VM.
- `vagrant up`/`ssh`/`halt`/`destroy` all accept a VM name to target a specific machine instead of all of them.
- VMs on the same `private_network` can reach each other directly by IP — this is how you simulate a real web-server-to-database-server architecture locally.
- Remember: a database VM accepting connections from another VM needs `'user'@'%'` (not `'localhost'`) and a MySQL `bind-address` config change — a common gotcha when splitting services across VMs for the first time.
- Provisioning steps written outside any `define` block run on **every** VM — useful for common baseline packages (like `vim`, `curl`) every machine needs.

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
- [x] Copilot AI for Coding
- [x] Multi VM Vagrantfile
- [ ] Systemctl & Tomcat 10

---

*Course: DecodingDevOps by Imran Teli (Udemy) | Status: ✅ Completed*
