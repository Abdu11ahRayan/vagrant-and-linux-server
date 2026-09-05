#  WordPress Setup

> **Repository:** DevOps Vagrant & Linux Server Notes — Section 6
>
> Manually deploying WordPress on your Vagrant VM using a full LAMP stack (Linux, Apache, MySQL, PHP) — one of the most common real-world web server setups.

---

##  What You're Building

```
Apache (web server)  →  PHP (runs WordPress code)  →  MySQL/MariaDB (stores content)
                                    ↓
                              WordPress files
```

---

##  Step 1 — Base VM (Reusing Earlier Config)

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.hostname = "wordpress-server"
  config.vm.network "private_network", ip: "192.168.56.20"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"     # WordPress + MySQL need more RAM than a plain static site
    vb.cpus = 2
  end
end
```

```bash
abdullah@DevOps:~/wordpress-vm$ vagrant up
abdullah@DevOps:~/wordpress-vm$ vagrant ssh
```

---

##  Step 2 — Install Apache, MySQL & PHP (LAMP Stack)

```bash
vagrant@wordpress-server:~$ sudo apt update
vagrant@wordpress-server:~$ sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql php-curl php-gd php-xml php-mbstring -y
```

| Package | Role |
|---|---|
| `apache2` | Web server |
| `mysql-server` | Database server — stores posts, users, settings |
| `php`, `libapache2-mod-php` | Runs WordPress's PHP code inside Apache |
| `php-mysql` | Lets PHP talk to MySQL |
| `php-curl`, `php-gd`, `php-xml`, `php-mbstring` | Required PHP extensions WordPress depends on |

---

##  Step 3 — Secure MySQL & Create the WordPress Database

```bash
vagrant@wordpress-server:~$ sudo mysql_secure_installation
```
Answer the prompts (set a root password, remove anonymous users, disallow remote root login, remove test database, reload privileges).

**Create a dedicated database and user for WordPress:**

```bash
vagrant@wordpress-server:~$ sudo mysql -u root -p
```

```sql
CREATE DATABASE wordpress_db;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

>  Never use WordPress with the MySQL `root` user directly — always create a dedicated, limited-privilege user like `wp_user` above.

---

##  Step 4 — Download & Extract WordPress

```bash
vagrant@wordpress-server:~$ cd /tmp
vagrant@wordpress-server:/tmp$ wget https://wordpress.org/latest.tar.gz
vagrant@wordpress-server:/tmp$ tar -xzf latest.tar.gz
vagrant@wordpress-server:/tmp$ sudo cp -r wordpress/* /var/www/html/
```

---

##  Step 5 — Configure `wp-config.php`

```bash
vagrant@wordpress-server:~$ cd /var/www/html
vagrant@wordpress-server:/var/www/html$ sudo cp wp-config-sample.php wp-config.php
vagrant@wordpress-server:/var/www/html$ sudo vim wp-config.php
```

Update the database connection details to match what you created in Step 3:

```php
define( 'DB_NAME', 'wordpress_db' );
define( 'DB_USER', 'wp_user' );
define( 'DB_PASSWORD', 'StrongPassword123!' );
define( 'DB_HOST', 'localhost' );
```

>  Also generate unique "Authentication Keys and Salts" from https://api.wordpress.org/secret-key/1.1/salt/ and paste them into `wp-config.php` — improves security by making cookies/sessions harder to forge.

---

##  Step 6 — Fix File Permissions & Ownership

WordPress needs Apache's user (`www-data`) to own the files so it can write uploads, install plugins/themes, and manage updates.

```bash
vagrant@wordpress-server:/var/www/html$ sudo chown -R www-data:www-data /var/www/html
vagrant@wordpress-server:/var/www/html$ sudo chmod -R 755 /var/www/html
```

---

##  Step 7 — Restart Apache & Complete Setup in the Browser

```bash
vagrant@wordpress-server:~$ sudo systemctl restart apache2
```

From your **host** browser, visit:

```
http://192.168.56.20
```

The WordPress install wizard will walk you through:

1. Selecting your language.
2. Site title, admin username, admin password, admin email.
3. Click **Install WordPress**.
4. Log in at `http://192.168.56.20/wp-admin`.

---

##  Quick Reference

| Command | Purpose |
|---|---|
| `sudo apt install apache2 mysql-server php ...` | Install the LAMP stack + required PHP extensions |
| `sudo mysql_secure_installation` | Harden the default MySQL install |
| `CREATE DATABASE wordpress_db;` | Create the WordPress database |
| `wget https://wordpress.org/latest.tar.gz` | Download WordPress |
| `cp wp-config-sample.php wp-config.php` | Create the working config file |
| `chown -R www-data:www-data /var/www/html` | Give Apache ownership so WordPress can write files |
| `sudo systemctl restart apache2` | Apply changes |

---

##  Key Takeaways

- WordPress needs a full **LAMP stack**: Apache (serve), MySQL (store data), PHP (run the code) — plus several PHP extensions (`curl`, `gd`, `xml`, `mbstring`).
- Always create a **dedicated MySQL user** for WordPress — never point it at the `root` database account.
- `wp-config.php` is the single file connecting WordPress to its database — get the `DB_NAME`/`DB_USER`/`DB_PASSWORD`/`DB_HOST` right or the install wizard fails immediately.
- `www-data` (Apache's user) must **own** the WordPress files, or plugin/theme installs and media uploads will fail with permission errors later.
- This entire multi-step manual process is exactly what the next section, **Automate WordPress Setup**, turns into a single provisioning script.

---

##  Topics Covered in This Repository (VAGRANT SECTION)

- [x] Vagrant VMs
- [x] Vagrant IP, RAM & CPU
- [x] Vagrant Sync Directories
- [x] Provisioning
- [x] Website Setup
- [x] WordPress Setup
- [ ] Automate Website Setup
- [ ] Automate WordPress Setup
- [ ] Copilot AI for Coding
- [ ] Multi VM Vagrantfile
- [ ] Systemctl & Tomcat 10

---

*Course: DecodingDevOps by Imran Teli (Udemy) | Status: ✅ Completed*
