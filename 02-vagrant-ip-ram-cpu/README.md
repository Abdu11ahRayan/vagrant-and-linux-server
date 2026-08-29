#  Vagrant IP, RAM & CPU

> **Repository:** DevOps Vagrant & Linux Server Notes — Section 2
>
> Configuring networking and hardware resources for your Vagrant VM — giving it a fixed IP you can reach, and enough RAM/CPU to actually run real workloads like a web server or WordPress.

---

##  Networking Options in Vagrant

By default, a fresh Vagrant VM only has **NAT** networking — it can reach the internet, but your host machine can't easily reach *it* by a predictable address. To fix that, you configure networking explicitly in the Vagrantfile.

### Private Network — Fixed, Host-Only IP

The most common setup for practice/dev environments — gives the VM a static IP reachable only from your host machine.

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.network "private_network", ip: "192.168.56.10"
end
```

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant reload
```

Now you can reach the VM directly from your host:

```bash
abdullah@DevOps:~$ ping 192.168.56.10
abdullah@DevOps:~$ ssh vagrant@192.168.56.10
```

### Public Network (Bridged) — VM Joins Your LAN

Makes the VM appear as its own device on your actual network (gets an IP from your router/DHCP).

```ruby
config.vm.network "public_network"
```

>  On `vagrant up`, Vagrant will prompt you to choose which physical network interface to bridge to.

### Port Forwarding — Expose a Specific Port

Maps a port inside the VM to a port on your host machine — useful for reaching a web server without setting up a private network.

```ruby
config.vm.network "forwarded_port", guest: 80, host: 8080
```

```bash
# Now http://localhost:8080 on your HOST reaches port 80 INSIDE the VM
```

| Networking Mode | Use Case |
|---|---|
| `private_network` (static IP) | Reach the VM directly from your host by a fixed, predictable IP — most common for dev/practice |
| `public_network` (bridged) | VM needs to be reachable from other devices on your LAN |
| `forwarded_port` | Quick single-port access without configuring a full private network |

---

##  Allocating RAM & CPU

By default, Vagrant gives a VM fairly minimal resources — not enough for anything beyond basic testing. Customize via the `provider` block:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.network "private_network", ip: "192.168.56.10"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"    # RAM in MB
    vb.cpus = 2            # Number of CPU cores
    vb.name = "my-devops-vm"
  end
end
```

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant reload
```

>  Resource changes require `vagrant reload` (or `vagrant halt` + `vagrant up`) to take effect — they won't apply to an already-running VM.

| Setting | Purpose |
|---|---|
| `vb.memory` | RAM allocated to the VM, in MB (e.g. `"2048"` = 2GB) |
| `vb.cpus` | Number of virtual CPU cores |
| `vb.name` | Friendly name shown in the VirtualBox GUI |
| `vb.gui = true` | Boot with the VirtualBox GUI window visible (default is headless) |

---

##  Assigning a Hostname

```ruby
config.vm.hostname = "devserver"
```

Sets the VM's internal hostname — useful when you have multiple VMs and want clear prompts (`vagrant@devserver:~$` instead of a generic name).

---

##  Putting It Together — A Complete Vagrantfile

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.hostname = "devserver"
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
    vb.name = "devserver-vm"
  end
end
```

---

##  Quick Reference

| Config Line | Purpose |
|---|---|
| `config.vm.network "private_network", ip: "x.x.x.x"` | Fixed, host-reachable IP |
| `config.vm.network "public_network"` | Bridge VM onto your LAN |
| `config.vm.network "forwarded_port", guest: G, host: H` | Map a VM port to a host port |
| `config.vm.provider "virtualbox" do \|vb\| ... end` | Block to configure RAM/CPU/name |
| `vb.memory = "2048"` | Set RAM (MB) |
| `vb.cpus = 2` | Set CPU core count |
| `config.vm.hostname = "name"` | Set the VM's internal hostname |
| `vagrant reload` | Apply Vagrantfile changes to a running VM |

---

##  Key Takeaways

- A fresh Vagrant VM only has NAT by default — add `private_network` with a static IP to reach it directly from your host.
- `forwarded_port` is the quickest way to test a single service (like a web server on port 80) without full network configuration.
- RAM and CPU are set inside the `config.vm.provider "virtualbox"` block — defaults are usually too small for real workloads like WordPress or Tomcat.
- Any Vagrantfile change (network, resources) requires `vagrant reload` to actually take effect on a running VM.
- A static private IP (e.g. `192.168.56.10`) is the standard setup for the rest of this course's exercises — website/WordPress setup, provisioning, and multi-VM configs will all build on this.

---

##  Topics Covered in This Repository (VAGRANT SECTION)

- [x] Vagrant VMs
- [x] Vagrant IP, RAM & CPU
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
