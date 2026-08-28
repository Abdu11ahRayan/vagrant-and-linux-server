#  Vagrant VMs

> **Repository:** DevOps Vagrant & Linux Server Notes — Section 1
>
> Vagrant lets you spin up disposable, reproducible virtual machines from a simple config file — perfect for practicing Linux server setups without touching your host OS.

---

##  What is Vagrant?

Vagrant is an open-source tool (by HashiCorp) for building and managing virtual machine environments through a single, version-controllable configuration file — the **Vagrantfile**.

Instead of manually clicking through VirtualBox/VMware to create a VM, installing an OS, and configuring it every time, Vagrant automates the whole thing from one text file that anyone on the team can share.

---

##  Why Vagrant for DevOps?

- **Reproducible environments** — the exact same VM every time, on any machine, from one config file.
- **Disposable** — break something? `vagrant destroy` and `vagrant up` again in seconds — no reinstalling an OS.
- **Team consistency** — everyone on the team runs an identical dev/test environment, eliminating "works on my machine" issues.
- **Practice ground** — safely practice Linux administration, server setups, and provisioning without risking a real production box.

---

##  How Vagrant Fits Together

```
Vagrantfile  →  Vagrant (CLI)  →  Provider (VirtualBox/VMware/etc.)  →  Virtual Machine
```

| Component | Role |
|---|---|
| **Vagrantfile** | Config file — defines the box, network, resources, and provisioning |
| **Provider** | The underlying virtualization software (VirtualBox is the free, default choice) |
| **Box** | A pre-packaged base VM image (e.g. `ubuntu/focal64`, `centos/7`) |

---

##  Installing Vagrant & VirtualBox

```bash
# Install VirtualBox (the provider) first
abdullah@DevOps:~$ sudo apt update
abdullah@DevOps:~$ sudo apt install virtualbox -y

# Install Vagrant
abdullah@DevOps:~$ sudo apt install vagrant -y

# Verify
abdullah@DevOps:~$ vagrant --version
Vagrant 2.4.1
```

>  On Windows/macOS, download installers directly from https://www.vagrantup.com/downloads and https://www.virtualbox.org/.

---

##  Creating Your First VM

```bash
abdullah@DevOps:~$ mkdir my-vagrant-vm && cd my-vagrant-vm
abdullah@DevOps:~/my-vagrant-vm$ vagrant init ubuntu/focal64
A `Vagrantfile` has been placed in this directory...
```

`vagrant init <box>` generates a starter `Vagrantfile` referencing the given box.

**Minimal generated Vagrantfile:**

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
end
```

---

##  Bringing the VM Up

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/focal64'...
==> default: Booting VM...
==> default: Machine booted and ready!
```

The **first** `vagrant up` downloads the box (can take a while); subsequent runs are fast since the box is cached locally.

---

##  Connecting to the VM — `vagrant ssh`

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant ssh
vagrant@ubuntu-focal:~$
```

Drops you straight into an SSH session inside the VM — no separate key setup needed, Vagrant handles it automatically.

---

##  Stopping, Destroying & Status

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant halt         # Gracefully shut down the VM (keeps it on disk)
abdullah@DevOps:~/my-vagrant-vm$ vagrant up            # Bring it back up later, right where you left off
abdullah@DevOps:~/my-vagrant-vm$ vagrant destroy        # Completely remove the VM (frees disk space)
abdullah@DevOps:~/my-vagrant-vm$ vagrant destroy -f      # Skip the confirmation prompt
abdullah@DevOps:~/my-vagrant-vm$ vagrant status          # Show whether the VM is running, saved, or not created
```

| Command | Effect |
|---|---|
| `vagrant halt` | Shuts the VM down, disk state preserved |
| `vagrant destroy` | Deletes the VM entirely — disk gone, config file untouched |
| `vagrant reload` | Restart the VM, re-reading the Vagrantfile (needed after network/resource changes) |
| `vagrant status` | Check current VM state |
| `vagrant global-status` | List all Vagrant VMs across your whole system |

---

##  Finding & Choosing Boxes

Base box images are browsable at **https://app.vagrantup.com/boxes/search**.

Common boxes used in this course:

| Box | OS |
|---|---|
| `ubuntu/focal64` | Ubuntu 20.04 LTS |
| `ubuntu/jammy64` | Ubuntu 22.04 LTS |
| `centos/7` | CentOS 7 |
| `generic/rhel8` | RHEL 8 |

```bash
abdullah@DevOps:~$ vagrant box list          # List boxes already downloaded locally
abdullah@DevOps:~$ vagrant box add centos/7  # Pre-download a box without creating a VM yet
abdullah@DevOps:~$ vagrant box remove centos/7   # Remove a downloaded box to free disk space
```

---

##  Quick Reference

| Command | Purpose |
|---|---|
| `vagrant init <box>` | Generate a starter Vagrantfile |
| `vagrant up` | Create/start the VM |
| `vagrant ssh` | SSH into the running VM |
| `vagrant halt` | Shut down the VM (keeps disk) |
| `vagrant reload` | Restart, re-applying Vagrantfile changes |
| `vagrant destroy` | Delete the VM entirely |
| `vagrant status` | Check current VM state |
| `vagrant box list` | List locally downloaded boxes |
| `vagrant global-status` | List every Vagrant VM on the system |

---

##  Key Takeaways

- Vagrant automates VM creation from a single, shareable **Vagrantfile** — no manual clicking through a hypervisor's GUI.
- `vagrant up` creates/starts, `vagrant halt` stops (keeps disk), `vagrant destroy` deletes entirely.
- `vagrant ssh` gives instant access with zero manual SSH key setup — Vagrant handles it.
- Boxes are pre-built base images (`ubuntu/focal64`, `centos/7`, etc.) — the first `vagrant up` downloads and caches the box; later runs are fast.
- This is the ideal safe sandbox for practicing everything from the Linux and Sudo/Permissions notes without any risk to a real machine.

---

##  Topics Covered in This Repository (VAGRANT SECTION)

- [x] Vagrant VMs
- [ ] Vagrant IP, RAM & CPU
- [ ] Vagrant Sync Directories
- [ ] Provisioning
- [ ] Website Setup
- [ ] WordPress Setup
- [ ] Automate Website Setup
- [ ] Automate WordPress Setup
- [ ] Copilot AI for Coding
- [ ] Multi VM Vagrantfile
- [ ] Systemctl & Tomcat 10

---

*Course: DecodingDevOps by Imran Teli (Udemy) | Status: ✅ Completed*
