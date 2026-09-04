#  Website Setup

> **Repository:** DevOps Vagrant & Linux Server Notes — Section 5
>
> Manually setting up a web server inside your Vagrant VM, step by step — before automating it in the next section. Doing it by hand first builds the understanding that automation later just codifies.

---

##  Goal

Get a working web server (Apache) running inside your Vagrant VM, serving a simple site from the synced folder, reachable from your host browser.

---

##  Step 1 — Bring Up the VM

Using the Vagrantfile from earlier sections (private IP + synced folder):

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.hostname = "webserver"
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.synced_folder "./website", "/var/www/html"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
  end
end
```

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant up
abdullah@DevOps:~/my-vagrant-vm$ vagrant ssh
```

---

##  Step 2 — Install Apache

```bash
vagrant@webserver:~$ sudo apt update
vagrant@webserver:~$ sudo apt install apache2 -y
```

**Verify it's running:**

```bash
vagrant@webserver:~$ sudo systemctl status apache2
● apache2.service - The Apache HTTP Server
   Active: active (running)
```

---

##  Step 3 — Allow Traffic Through the Firewall (if enabled)

```bash
vagrant@webserver:~$ sudo ufw allow 'Apache'
vagrant@webserver:~$ sudo ufw status
```

>  On a fresh Vagrant box, `ufw` is usually inactive by default — check with `sudo ufw status` before assuming this step is needed.

---

##  Step 4 — Confirm It's Reachable

From your **host** machine's browser or terminal:

```bash
abdullah@DevOps:~$ curl http://192.168.56.10
<html><body><h1>Apache2 Default Page</h1>...
```

Visiting `http://192.168.56.10` in a browser shows Apache's default "It works!" page.

---

##  Step 5 — Deploy Your Own Site

Since `/var/www/html` is synced with your host's `./website` folder, you can edit files **on the host** and see them reflected instantly.

```bash
# On the HOST, not inside the VM
abdullah@DevOps:~/my-vagrant-vm$ mkdir -p website
abdullah@DevOps:~/my-vagrant-vm$ cat > website/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Abdullah's DevOps Practice Site</title></head>
<body>
  <h1>Hello from my Vagrant VM!</h1>
</body>
</html>
EOF
```

Refresh `http://192.168.56.10` — your custom page replaces the Apache default.

---

##  Optional — Adding PHP Support

```bash
vagrant@webserver:~$ sudo apt install php libapache2-mod-php -y
vagrant@webserver:~$ sudo systemctl restart apache2
```

**Test PHP is working:**

```bash
# On the HOST
abdullah@DevOps:~/my-vagrant-vm$ echo "<?php phpinfo(); ?>" > website/info.php
```

Visit `http://192.168.56.10/info.php` — you should see the full PHP configuration page.

>  Delete `info.php` after testing — `phpinfo()` reveals detailed server configuration and shouldn't be left exposed, even in a practice VM.

---

##  Understanding Apache's Config Layout

| Path | Purpose |
|---|---|
| `/var/www/html` | Default document root — where site files live |
| `/etc/apache2/apache2.conf` | Main Apache configuration file |
| `/etc/apache2/sites-available/` | Available virtual host configs |
| `/etc/apache2/sites-enabled/` | Symlinks to active virtual host configs |
| `/var/log/apache2/access.log` | Request log |
| `/var/log/apache2/error.log` | Error log — first place to check when something's broken |

```bash
vagrant@webserver:~$ sudo tail -f /var/log/apache2/error.log      # Watch errors live
```

---

##  Quick Reference

| Command | Purpose |
|---|---|
| `sudo apt install apache2 -y` | Install Apache |
| `sudo systemctl status apache2` | Check Apache is running |
| `sudo systemctl restart apache2` | Restart Apache after config/PHP changes |
| `sudo ufw allow 'Apache'` | Open the firewall for HTTP traffic |
| `curl http://<vm-ip>` | Quick reachability test from the host |
| `sudo tail -f /var/log/apache2/error.log` | Live-watch Apache errors |

---

##  Key Takeaways

- The core manual flow: `vagrant up` → SSH in → `apt install apache2` → verify with `systemctl status` → confirm reachability from the host.
- Because `/var/www/html` is synced to your host's `./website` folder, you edit site files with your normal editor on the host — no need to SSH in just to change HTML.
- PHP support is a simple `apt install php libapache2-mod-php` away — always followed by an Apache restart to load the new module.
- Apache's error log (`/var/log/apache2/error.log`) is the first place to check when a page won't load — `tail -f` it while testing.
- Doing this manually first matters — the next section (**Automate Website Setup**) turns these exact same steps into a provisioning script, and it'll make a lot more sense having done it by hand once.

---

##  Topics Covered in This Repository (VAGRANT SECTION)

- [x] Vagrant VMs
- [x] Vagrant IP, RAM & CPU
- [x] Vagrant Sync Directories
- [x] Provisioning
- [x] Website Setup
- [ ] WordPress Setup
- [ ] Automate Website Setup
- [ ] Automate WordPress Setup
- [ ] Copilot AI for Coding
- [ ] Multi VM Vagrantfile
- [ ] Systemctl & Tomcat 10

---

*Course: DecodingDevOps by Imran Teli (Udemy) | Status: ✅ Completed*
