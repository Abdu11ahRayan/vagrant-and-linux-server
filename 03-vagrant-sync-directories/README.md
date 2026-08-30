#  Vagrant Sync Directories

> **Repository:** DevOps Vagrant & Linux Server Notes — Section 3
>
> Sharing files between your host machine and the VM in real time — edit code in your favorite editor on your host, run it instantly inside the VM.

---

##  What is a Synced Folder?

A synced folder mirrors a directory on your **host machine** into a directory inside the **VM**, keeping both in sync automatically. Edit a file on your host, and the change appears instantly inside the VM — no manual copying, no `scp`.

```
Host: ~/my-vagrant-vm/website/   <-- synced -->   VM: /var/www/html/
```

---

##  Default Synced Folder

Every Vagrant VM automatically syncs the folder containing the `Vagrantfile` to `/vagrant` inside the VM — no configuration needed.

```bash
abdullah@DevOps:~/my-vagrant-vm$ ls
Vagrantfile

abdullah@DevOps:~/my-vagrant-vm$ vagrant ssh
vagrant@devserver:~$ ls /vagrant
Vagrantfile
```

Any file you create in `~/my-vagrant-vm/` on your host appears in `/vagrant` inside the VM automatically, and vice versa.

---

##  Adding Custom Synced Folders

For real projects, you'll usually want a dedicated folder mapped to a meaningful path (like a web server's document root):

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.network "private_network", ip: "192.168.56.10"

  config.vm.synced_folder "./website", "/var/www/html"
end
```

```bash
abdullah@DevOps:~/my-vagrant-vm$ mkdir website
abdullah@DevOps:~/my-vagrant-vm$ echo "<h1>Hello from host!</h1>" > website/index.html
abdullah@DevOps:~/my-vagrant-vm$ vagrant reload
```

```bash
abdullah@DevOps:~/my-vagrant-vm$ vagrant ssh
vagrant@devserver:~$ cat /var/www/html/index.html
<h1>Hello from host!</h1>
```

>  Path syntax: `config.vm.synced_folder "<host-path>", "<guest-path>"` — host path is relative to the Vagrantfile's location unless given as absolute.

---

##  Disabling the Default Synced Folder

If you don't want the automatic `/vagrant` mapping (e.g. to keep the VM's filesystem cleaner):

```ruby
config.vm.synced_folder ".", "/vagrant", disabled: true
```

---

##  Synced Folder Options

```ruby
config.vm.synced_folder "./website", "/var/www/html",
  owner: "www-data",
  group: "www-data",
  mount_options: ["dmode=755", "fmode=644"]
```

| Option | Purpose |
|---|---|
| `owner:` | Sets the file owner inside the VM (default is usually `vagrant`) |
| `group:` | Sets the file group inside the VM |
| `mount_options:` | Fine-tune mount permissions (`dmode` for directories, `fmode` for files) |
| `type:` | Sync mechanism — `"virtualbox"` (default), `"nfs"`, `"rsync"`, `"smb"` |

---

##  Sync Mechanisms

| Type | Speed | Notes |
|---|---|---|
| **VirtualBox shared folders** (default) | Slower | Works out of the box, no extra setup |
| **NFS** | Faster | Better performance for large projects; needs NFS installed on host |
| **rsync** | One-way only | Good for large repos where live 2-way sync isn't needed |

```ruby
config.vm.synced_folder "./website", "/var/www/html", type: "nfs"
```

---

##  Quick Reference

| Config Line | Purpose |
|---|---|
| `/vagrant` (automatic) | Default sync of the Vagrantfile's own directory |
| `config.vm.synced_folder "host", "guest"` | Add a custom synced folder |
| `disabled: true` | Turn off the default `/vagrant` sync |
| `owner:` / `group:` | Set file ownership inside the VM |
| `type: "nfs"` | Use NFS instead of default VirtualBox sharing (faster) |
| `vagrant reload` | Apply new/changed synced folder settings |

---

##  Key Takeaways

- Every VM automatically syncs its Vagrantfile's directory to `/vagrant` inside the VM — no config required.
- `config.vm.synced_folder` maps any host folder to any guest path — ideal for pointing a web server's document root straight at your host-side project folder.
- Changes made on the host appear instantly inside the VM (and usually vice versa) — this is what makes local development against a VM practical.
- Default VirtualBox sync is convenient but slow for large projects — NFS is the faster alternative when performance matters.
- This is exactly the mechanism the upcoming **Website Setup** and **WordPress Setup** sections rely on — syncing your site files from the host into the VM's web server directory.

---

##  Topics Covered in This Repository (VAGRANT SECTION)

- [x] Vagrant VMs
- [x] Vagrant IP, RAM & CPU
- [x] Vagrant Sync Directories
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
