#  Systemctl & Tomcat 10

> **Repository:** DevOps Vagrant & Linux Server Notes — Section 11 (Final Section)
>
> Deploying Apache Tomcat 10 as a proper `systemd` service inside your Vagrant VM — the standard way to run Java web applications on Linux.

---

##  What is Tomcat?

**Apache Tomcat** is a Java Servlet container — it runs Java web applications (`.war` files) rather than PHP or static HTML like Apache/Nginx serve. It's the standard runtime for Java-based web apps in many enterprise and DevOps environments.

Tomcat 10 specifically moved from the old `javax.*` namespace to `jakarta.*` — a notable breaking change from Tomcat 9 that matters if you're deploying an older app.

---

##  Step 1 — Base VM

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.hostname = "tomcat-server"
  config.vm.network "private_network", ip: "192.168.56.30"
  config.vm.network "forwarded_port", guest: 8080, host: 8080

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end
end
```

```bash
abdullah@DevOps:~/tomcat-vm$ vagrant up
abdullah@DevOps:~/tomcat-vm$ vagrant ssh
```

---

##  Step 2 — Install Java (Tomcat's Prerequisite)

Tomcat 10 requires Java 11 or newer.

```bash
vagrant@tomcat-server:~$ sudo apt update
vagrant@tomcat-server:~$ sudo apt install openjdk-17-jdk -y
vagrant@tomcat-server:~$ java -version
openjdk version "17.0.9" ...
```

---

##  Step 3 — Download & Install Tomcat 10

```bash
vagrant@tomcat-server:~$ cd /opt
vagrant@tomcat-server:/opt$ sudo wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.19/bin/apache-tomcat-10.1.19.tar.gz
vagrant@tomcat-server:/opt$ sudo tar -xzf apache-tomcat-10.1.19.tar.gz
vagrant@tomcat-server:/opt$ sudo mv apache-tomcat-10.1.19 tomcat10
```

>  Always check https://tomcat.apache.org/download-10.cgi for the current stable version — version numbers age quickly.

---

##  Step 4 — Create a Dedicated Tomcat User

Never run Tomcat as `root` — a compromised web app shouldn't mean a compromised system.

```bash
vagrant@tomcat-server:~$ sudo useradd -m -d /opt/tomcat10 -U -s /bin/false tomcat
vagrant@tomcat-server:~$ sudo chown -R tomcat:tomcat /opt/tomcat10
vagrant@tomcat-server:~$ sudo chmod -R 755 /opt/tomcat10
```

| Flag | Meaning |
|---|---|
| `-m` | Create a home directory |
| `-d /opt/tomcat10` | Set that home directory to Tomcat's install path |
| `-U` | Create a matching group with the same name |
| `-s /bin/false` | No login shell — this account can't be used to log in interactively |

---

##  Step 5 — Create a systemd Service File

Rather than starting Tomcat manually with `./startup.sh` every time, wrap it as a proper systemd service — so it starts on boot, restarts on failure, and integrates with `systemctl` like any other service.

```bash
vagrant@tomcat-server:~$ sudo vim /etc/systemd/system/tomcat.service
```

```ini
[Unit]
Description=Apache Tomcat 10
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
Environment="CATALINA_HOME=/opt/tomcat10"
Environment="CATALINA_PID=/opt/tomcat10/temp/tomcat.pid"

ExecStart=/opt/tomcat10/bin/startup.sh
ExecStop=/opt/tomcat10/bin/shutdown.sh

Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
vagrant@tomcat-server:~$ sudo systemctl daemon-reload
vagrant@tomcat-server:~$ sudo systemctl enable tomcat
vagrant@tomcat-server:~$ sudo systemctl start tomcat
vagrant@tomcat-server:~$ sudo systemctl status tomcat
● tomcat.service - Apache Tomcat 10
   Active: active (running)
```

>  `systemctl daemon-reload` is required any time you create or edit a `.service` file — systemd won't pick up the change otherwise.

---

##  Step 6 — Verify It's Running

```bash
abdullah@DevOps:~$ curl http://192.168.56.30:8080
# Or via the forwarded port from the host:
abdullah@DevOps:~$ curl http://localhost:8080
```

Visiting `http://192.168.56.30:8080` shows the Tomcat welcome page.

---

##  Step 7 — Enable the Manager & Host-Manager Apps

By default, Tomcat's web-based Manager App (for deploying `.war` files through the browser) requires a user with the right role.

```bash
vagrant@tomcat-server:~$ sudo vim /opt/tomcat10/conf/tomcat-users.xml
```

```xml
<tomcat-users>
  <role rolename="manager-gui"/>
  <role rolename="admin-gui"/>
  <user username="abdullah" password="StrongPassword123!" roles="manager-gui,admin-gui"/>
</tomcat-users>
```

```bash
vagrant@tomcat-server:~$ sudo systemctl restart tomcat
```

Now `http://192.168.56.30:8080/manager/html` prompts for the credentials above.

---

##  Step 8 — Deploying a `.war` Application

**Option A — Drop into the `webapps` folder (simplest):**

```bash
vagrant@tomcat-server:~$ sudo cp myapp.war /opt/tomcat10/webapps/
```

Tomcat auto-detects and deploys it — accessible at `http://192.168.56.30:8080/myapp`.

**Option B — Deploy via the Manager App UI:**

```
1. Go to http://192.168.56.30:8080/manager/html
2. Log in with the credentials from Step 7
3. Under "WAR file to deploy", browse to myapp.war and click Deploy
```

---

##  Quick Reference

| Command | Purpose |
|---|---|
| `sudo apt install openjdk-17-jdk -y` | Install Java (Tomcat prerequisite) |
| `useradd -m -d <path> -U -s /bin/false tomcat` | Create a dedicated, no-login Tomcat user |
| `sudo systemctl daemon-reload` | Load a new/changed `.service` file |
| `sudo systemctl enable tomcat` | Start Tomcat automatically on boot |
| `sudo systemctl start/stop/restart/status tomcat` | Standard service lifecycle management |
| `cp app.war /opt/tomcat10/webapps/` | Deploy a WAR file by dropping it in |
| `/manager/html` | Web UI for deploying/undeploying apps |

---

##  Key Takeaways

- Tomcat serves Java web apps (`.war` files) — a different role than Apache/Nginx, which serve static/PHP content.
- Always run Tomcat under a **dedicated, no-login user** (`useradd ... -s /bin/false`), never as root.
- Wrapping Tomcat in a proper `systemd` service (`/etc/systemd/system/tomcat.service`) gives you standard `systemctl start/stop/status/enable` control instead of manually running shell scripts — and `Restart=on-failure` means it recovers automatically if it crashes.
- `systemctl daemon-reload` is required every time the service file itself changes — a step it's easy to forget.
- `.war` files can be deployed either by dropping them straight into `webapps/`, or through the browser-based Manager App once `tomcat-users.xml` grants a user the `manager-gui` role.

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
- [x] Systemctl & Tomcat 10

---

*Course: DecodingDevOps by Imran Teli (Udemy) | Status: ✅ Vagrant & Linux Server section complete — 11/11*
