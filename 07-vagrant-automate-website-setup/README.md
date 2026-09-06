#  Automate Website Setup

> **Repository:** DevOps Vagrant & Linux Server Notes — Section 7
>
> Turning the manual Apache website setup from Section 5 into a single provisioning script — the whole server configures itself the moment `vagrant up` runs.

---

##  The Goal

Everything done by hand in **Website Setup** — install Apache, deploy a site, verify it's running — happens **automatically** now, with zero manual SSH steps.

```
Manual (Section 5):  vagrant up → ssh → apt install → deploy → verify   (multiple manual steps)
Automated (this):    vagrant up → done, site is already live            (one command)
```

---

##  Step 1 — Write the Provisioning Script

```bash
# provision-website.sh
#!/bin/bash
set -e   # Exit immediately if any command fails

echo ">>> Updating package lists..."
apt update

echo ">>> Installing Apache..."
apt install -y apache2

echo ">>> Enabling Apache to start on boot..."
systemctl enable apache2

echo ">>> Starting Apache..."
systemctl restart apache2

echo ">>> Setting correct ownership on document root..."
chown -R www-data:www-data /var/www/html

echo ">>> Website provisioning complete!"
```

>  `set -e` is important — without it, a failed command (e.g. a bad package name) lets the script silently continue instead of stopping, hiding the real problem.

---

##  Step 2 — Wire It Into the Vagrantfile

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.hostname = "webserver"
  config.vm.network "private_network", ip: "192.168.56.10"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
  end

  config.vm.synced_folder "./website", "/var/www/html"

  config.vm.provision "shell", path: "provision-website.sh"
end
```

```bash
abdullah@DevOps:~/my-vagrant-vm$ chmod +x provision-website.sh
abdullah@DevOps:~/my-vagrant-vm$ vagrant up
==> default: Running provisioner: shell...
    default: >>> Updating package lists...
    default: >>> Installing Apache...
    default: >>> Website provisioning complete!
```

By the time this finishes, `http://192.168.56.10` is already serving your synced `website/index.html`.

---

##  Step 3 — Verify Automatically (Optional Health Check)

Add a self-check at the end of the script so provisioning fails loudly if something's actually wrong, instead of silently "succeeding":

```bash
echo ">>> Verifying Apache is responding..."
if curl -s http://localhost | grep -q "html"; then
  echo ">>> SUCCESS: Website is live!"
else
  echo ">>> ERROR: Website did not respond as expected!"
  exit 1
fi
```

---

##  Testing the Full Cycle From Scratch

The real value of automation is being able to destroy and rebuild identically, every time:

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant destroy -f
abdullah@DevOps:~/my-vagrant-vm$ vagrant up
```

If the site comes back up fully configured with zero manual steps, the automation is working correctly.

---

##  Making It Reusable — Variables Instead of Hardcoded Values

```bash
#!/bin/bash
set -e

PACKAGES="apache2"
DOC_ROOT="/var/www/html"

apt update
apt install -y $PACKAGES
systemctl enable apache2
systemctl restart apache2
chown -R www-data:www-data $DOC_ROOT
```

Small changes like this make the script easier to adapt later — e.g. adding more packages without hunting through the whole file.

---

##  Quick Reference

| Command / Line | Purpose |
|---|---|
| `set -e` | Stop the script immediately on any command failure |
| `config.vm.provision "shell", path: "script.sh"` | Wire an external script into the Vagrantfile |
| `chmod +x script.sh` | Make the script executable before Vagrant runs it |
| `vagrant destroy -f && vagrant up` | Full clean rebuild — the real test of automation |
| `curl -s http://localhost \| grep "html"` | Simple in-script health check |

---

##  Key Takeaways

- Automation here just means: take the exact manual steps from Website Setup and put them in a script, then point `config.vm.provision` at it.
- `set -e` at the top of any provisioning script is a small habit that saves hours of confused debugging — fail fast instead of continuing on broken state.
- The real proof of good automation is `vagrant destroy -f && vagrant up` reliably producing an identical, working server every single time.
- Adding a simple `curl` health check at the end of the script turns "it probably worked" into "it definitely worked, or the script told me it didn't."
- This same pattern — manual first, then scripted — is exactly what the next section applies to the much more complex **WordPress Setup**.

---

##  Topics Covered in This Repository (VAGRANT SECTION)

- [x] Vagrant VMs
- [x] Vagrant IP, RAM & CPU
- [x] Vagrant Sync Directories
- [x] Provisioning
- [x] Website Setup
- [x] WordPress Setup
- [x] Automate Website Setup
- [ ] Automate WordPress Setup
- [ ] Copilot AI for Coding
- [ ] Multi VM Vagrantfile
- [ ] Systemctl & Tomcat 10

---

*Course: DecodingDevOps by Imran Teli (Udemy) | Status: ✅ Completed*
