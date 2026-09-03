#  Provisioning

> **Repository:** DevOps Vagrant & Linux Server Notes — Section 4
>
> Automatically configuring a VM the moment it boots — installing packages, starting services, running setup scripts — so you never manually set up the same server twice.

---

##  What is Provisioning?

Provisioning is the process of automatically installing software and configuring a VM right after it's created — instead of manually SSH-ing in and running commands every time. Vagrant runs your provisioning steps automatically on the **first** `vagrant up`.

```
vagrant up  →  VM boots  →  provisioning runs automatically  →  fully-configured VM, ready to use
```

---

##  Inline Shell Provisioning

The simplest option — commands written directly in the Vagrantfile.

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.network "private_network", ip: "192.168.56.10"

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt update
    sudo apt install -y apache2
    sudo systemctl enable apache2
    sudo systemctl start apache2
  SHELL
end
```

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant up
==> default: Running provisioner: shell...
    default: Reading package lists...
    default: Setting up apache2 ...
```

By the time `vagrant up` finishes, Apache is already installed and running — no manual SSH step needed.

---

##  External Shell Script Provisioning

For longer setups, keep the script in its own file — cleaner and reusable.

```ruby
config.vm.provision "shell", path: "provision.sh"
```

```bash
# provision.sh
#!/bin/bash
sudo apt update
sudo apt install -y apache2 php libapache2-mod-php
sudo systemctl enable apache2
sudo systemctl start apache2
echo "Provisioning complete!"
```

```bash
abdullah@DevOps:~/my-vagrant-vm$ chmod +x provision.sh
abdullah@DevOps:~/my-vagrant-vm$ vagrant up
```

>  Keep `provision.sh` in the same folder as the Vagrantfile — the `path:` is relative to it.

---

##  Re-running Provisioning

By default, provisioning only runs on the **first** `vagrant up` for a given VM. To force it to run again:

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant provision                 # Re-run provisioning on the current VM
abdullah@DevOps:~/my-vagrant-vm$ vagrant up --provision              # Force provisioning during an up
abdullah@DevOps:~/my-vagrant-vm$ vagrant reload --provision          # Restart + re-run provisioning
```

---

##  Named Provisioners

Give each provisioning block a name so you can run them selectively:

```ruby
config.vm.provision "install-apache", type: "shell", path: "apache.sh"
config.vm.provision "install-php", type: "shell", path: "php.sh"
```

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant provision --provision-with install-php
```

---

##  Other Provisioner Types

Shell is the simplest and most common for a course/learning context, but Vagrant supports several provisioner backends:

| Provisioner | Use Case |
|---|---|
| `shell` | Plain Bash/shell scripts — simplest, most flexible |
| `ansible` | Run an Ansible playbook against the VM |
| `puppet` | Apply Puppet manifests |
| `chef` | Apply Chef cookbooks |
| `docker` | Provision using Docker |

```ruby
config.vm.provision "ansible" do |ansible|
  ansible.playbook = "playbook.yml"
end
```

>  Shell provisioning is the foundation — once comfortable with it, tools like Ansible (`ansible` provisioner) let you manage far more complex, idempotent configurations.

---

##  Practical Example — Full Web Server Provisioning

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

  config.vm.provision "shell", inline: <<-SHELL
    apt update
    apt install -y apache2
    systemctl enable apache2
    systemctl start apache2
  SHELL
end
```

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant up
# ... visit http://192.168.56.10 in a browser — Apache serving your synced website/ folder
```

---

##  Quick Reference

| Command / Config | Purpose |
|---|---|
| `config.vm.provision "shell", inline: "..."` | Inline provisioning script |
| `config.vm.provision "shell", path: "script.sh"` | External script file provisioning |
| `vagrant provision` | Re-run provisioning manually |
| `vagrant up --provision` | Force provisioning during `up` |
| `vagrant reload --provision` | Restart + re-provision |
| `vagrant provision --provision-with <name>` | Run only a specific named provisioner |

---

##  Key Takeaways

- Provisioning automates VM setup — install packages, start services, deploy configs — so `vagrant up` hands you a fully-ready server, not a blank OS.
- Provisioning normally runs only once (on first `up`) — use `vagrant provision` or `--provision` to force a re-run after editing your script.
- Inline scripts are fine for quick setups; external `.sh` files are cleaner for anything longer or reused across projects.
- Shell provisioning is the simplest entry point — Ansible/Puppet/Chef provisioners exist for more advanced, idempotent configuration management later in the DevOps stack.
- This is exactly the mechanism the next two sections (**Automate Website Setup**, **Automate WordPress Setup**) build directly on top of.

---

##  Topics Covered in This Repository (VAGRANT SECTION)

- [x] Vagrant VMs
- [x] Vagrant IP, RAM & CPU
- [x] Vagrant Sync Directories
- [x] Provisioning
- [ ] Website Setup
- [ ] WordPress Setup
- [ ] Automate Website Setup
- [ ] Automate WordPress Setup
- [ ] Copilot AI for Coding
- [ ] Multi VM Vagrantfile
- [ ] Systemctl & Tomcat 10

---

*Course: DecodingDevOps by Imran Teli (Udemy) | Status: ✅ Completed*
