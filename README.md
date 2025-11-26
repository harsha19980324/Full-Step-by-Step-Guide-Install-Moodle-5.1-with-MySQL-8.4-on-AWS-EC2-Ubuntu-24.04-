<p>
  <h1 align="center" style="color: #0078D4; font-size: 42px; font-weight: 900; margin: 20px 0; text-shadow: 0 4px 8px rgba(0,0,0,0.2);">
    Full Step-by-Step Guide: Install Moodle 5.1 with MySQL 8.4 on AWS EC2 (Ubuntu 24.04)
  </h1>
</p>
---
## Prerequisites
1. Launch an **AWS EC2 instance** running **Ubuntu 24.04 LTS** (t3.medium or larger recommended for production)
2. Assign a **public IPv4 address** (Elastic IP recommended for stability)
3. Configure **Security Group** to allow inbound traffic:
   - Port 22 (SSH) from your IP
   - Port 80 (HTTP) from 0.0.0.0/0
   - Port 443 (HTTPS) from 0.0.0.0/0 (for later SSL setup)
4. SSH into your instance as a user with **sudo privileges** (e.g., `ubuntu` user)

> **<span style="color:red;">Note: Ensure your EC2 key pair is downloaded and permissions set to 400</span>**

<p align="center">
  <img src="https://example.com/ec2-launch.png"
       alt="AWS EC2 Launch Ubuntu 24.04"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>AWS EC2 Launch — Select Ubuntu 24.04 LTS AMI and t3.medium instance type</em>
</p>

-----------------------------------------------------------------------------------------------------------------------------------------------
## Phase 1: Install MySQL 8.4 (Latest Community Version)
1. **Check existing MySQL**:
   ```bash
   mysql --version
   ```
   → Verifies if any MySQL is pre-installed and displays its version (should be empty or outdated).

2. **Update system packages**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
   → Refreshes package lists and upgrades all installed software to latest versions (critical for security and compatibility).

3. **Clean up unused packages**:
   ```bash
   sudo apt autoremove
   ```
   → Removes orphaned dependencies to free space and prevent conflicts.

4. **Create download directory**:
   ```bash
   mkdir mysql8.4 && cd mysql8.4
   ```
   → Organizes downloads in a dedicated folder.

5. **Download MySQL 8.4 DEB bundle**:
   ```bash
   wget https://dev.mysql.com/get/Downloads/MySQL-8.4/mysql-server_8.4.3-1ubuntu24.04_amd64.deb-bundle.tar
   ```
   → Fetches the official MySQL 8.4 tarball for Ubuntu 24.04 (check https://dev.mysql.com/downloads/mysql/8.4.html for the absolute latest version).

<p align="center">
  <img src="https://example.com/mysql-download.png"
       alt="MySQL 8.4 Download"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>MySQL Download Page — Select DEB Bundle for Ubuntu 24.04 AMD64</em>
</p>
-----------------------------------------------------------------------------------------------------------------------------------------------
6. **Verify download**:
   ```bash
   ls
   ```
   → Lists files to confirm the tarball is present.

7. **Extract packages**:
   ```bash
   tar -xf mysql-server_8.4.3-1ubuntu24.04_amd64.deb-bundle.tar
   ```
   → Unpacks all required .deb files (server, client, libraries, etc.).

8. **List extracted files**:
   ```bash
   ls
   ```
   → Displays .deb packages like `mysql-community-server_*.deb`.

9. **Fix dependencies**:
   ```bash
   sudo apt update --fix-missing
   ```
   → Resolves any missing package links.

10. **Install MySQL packages**:
    ```bash
    sudo apt install ./mysql-*.deb
    ```
    → Installs all MySQL components via wildcard. During prompts:
    - Set **root password** (e.g., `XXX@XXX` — use a strong one!)
    - Select **"Use Legacy Authentication Method"** (essential for PHP/Moodle compatibility).

11. **Verify installation**:
    ```bash
    mysql --version
    ```
    → Outputs: `mysql  Ver 8.4.3 for Linux on x86_64 (MySQL Community Server - GPL)`

> **<span style="color:red;">Note: If errors occur, run `sudo apt --fix-broken install` and retry</span>**

<p align="center">
  <img src="https://example.com/mysql-install-prompt.png"
       alt="MySQL Installation Prompts"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>MySQL Setup — Choose Legacy Auth and set root password</em>
</p>

-----------------------------------------------------------------------------------------------------------------------------------------------
## Phase 2: Install Apache, PHP, and Required Extensions
1. **Install web stack**:
   ```bash
   sudo apt install apache2 php libapache2-mod-php php-mysql -y
   ```
   → Deploys Apache2 server, PHP 8.3 (default on Ubuntu 24.04), and MySQL connector.

2. **Check Apache status**:
   ```bash
   sudo systemctl status apache2
   ```
   → Confirms Apache is `active (running)`.

3. **Get IP addresses**:
   ```bash
   ip -br a
   ```
   → Shows network interfaces. Note:
   - **Public IP** (e.g., `16.171.168.250`)
   - **Private IP** (e.g., `172.31.36.120`)

4. **Test web server**:
   → Open browser → `http://YOUR_PUBLIC_IP`  
   → You should see **Apache2 Ubuntu Default Page**!

> **<span style="color:red;">Note: If page doesn't load, check Security Group inbound rules for port 80</span>**

<p align="center">
  <img src="https://example.com/apache-default.png"
       alt="Apache Default Page"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>Apache2 Default Page — Confirms web server is operational</em>
</p>

-----------------------------------------------------------------------------------------------------------------------------------------------
## Phase 3: Download and Extract Moodle 5.1
1. **Download Moodle**:
   ```bash
   wget https://download.moodle.org/download.php/direct/stable501/moodle-latest-501.tgz
   ```
   → Grabs the latest Moodle 5.1.x stable release.

2. **Move to web root**:
   ```bash
   sudo mv moodle-latest-501.tgz /var/www/
   ```
   → Places file in Apache's document root.

3. **Extract archive**:
   ```bash
   cd /var/www/
   sudo tar -xf moodle-latest-501.tgz
   ```
   → Creates `/var/www/moodle` directory.

4. **Verify extraction**:
   ```bash
   ls
   ```
   → Lists `moodle` folder and `.tgz` file.

5. **Remove default site**:
   ```bash
   sudo rm -r html
   ```
   → Clears Apache's sample page.

<p align="center">
  <img src="https://example.com/moodle-extract.png"
       alt="Moodle Extraction"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>Moodle Files Extracted — Ready for Apache configuration</em>
</p>

-----------------------------------------------------------------------------------------------------------------------------------------------
## Phase 4: Configure Apache Virtual Host for Moodle
1. **Navigate to sites-available**:
   ```bash
   cd /etc/apache2/sites-available/
   sudo nano moodle.conf
   ```

2. **Paste configuration** (replace IPs):
   ```apache
   <VirtualHost *:80>
       ServerName 16.171.168.250          # ← Your Public IP
       ServerAlias 172.31.36.120          # ← Your Private IP

       DocumentRoot /var/www/moodle

       <Directory /var/www/moodle>
           Options -Indexes +FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/moodle_error.log
       CustomLog ${APACHE_LOG_DIR}/moodle_access.log combined
   </VirtualHost>
   ```
   → Save: `Ctrl+O` → Enter → `Ctrl+X`.

3. **Enable Moodle site**:
   ```bash
   sudo a2ensite moodle.conf
   ```
   → Activates the virtual host.

4. **Disable default site**:
   ```bash
   sudo a2dissite 000-default.conf
   ```
   → Deactivates sample page.

5. **Reload Apache**:
   ```bash
   sudo systemctl reload apache2
   ```
   → Applies config without downtime.

6. **Test**:
   → Browser: `http://YOUR_PUBLIC_IP` → **Moodle installer appears**!

> **<span style="color:red;">Note: Syntax check with `sudo apache2ctl configtest` if reload fails</span>**

<p align="center">
  <img src="https://example.com/apache-vhost.png"
       alt="Apache Virtual Host Config"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>Moodle.conf — Custom Virtual Host for seamless routing</em>
</p>

-----------------------------------------------------------------------------------------------------------------------------------------------
## Phase 5: Create Moodle Data Directory (Important for Security)
1. **Create data folder**:
   ```bash
   cd /var/www/
   sudo mkdir moodledata
   ```
   → Secure storage for user files (outside web root).

2. **Set permissions**:
   ```bash
   sudo chown -R www-data:www-data moodledata
   sudo chmod 770 moodledata
   ```
   → Ownership to Apache user; restricted access.

> **<span style="color:red;">Warning: Never place moodledata inside /var/www/ for production security</span>**

<p align="center">
  <img src="https://example.com/moodledata-perm.png"
       alt="Moodle Data Permissions"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>Data Directory Setup — Ensures secure file handling</em>
</p>

-----------------------------------------------------------------------------------------------------------------------------------------------
## Phase 6: Configure MySQL Database for Moodle
1. **Login to MySQL**:
   ```bash
   mysql -u root -p
   ```
   → Enter root password.

2. **Create database** (inside MySQL):
   ```sql
   CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```
   → Unicode-ready database.

3. **Create user**:
   ```sql
   CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'StrongPassword123!';
   ```

4. **Grant permissions**:
   ```sql
   GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, DROP, INDEX, ALTER ON moodle.* TO 'moodleuser'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

5. **Optional: Optimize MySQL config**:
   ```bash
   sudo nano /etc/mysql/my.cnf
   ```
   Add:
   ```ini
   [client]
   default-character-set = utf8mb4

   [mysqld]
   innodb_file_per_table = 1
   character-set-server = utf8mb4
   collation-server = utf8mb4_unicode_ci
   skip-character-set-client-handshake

   [mysql]
   default-character-set = utf8mb4
   ```
   → Save & restart:
   ```bash
   sudo systemctl restart mysql
   ```

<p align="center">
  <img src="https://example.com/mysql-db-create.png"
       alt="MySQL Database Setup"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>Create Moodle DB & User — Minimal privileges for security</em>
</p>

-----------------------------------------------------------------------------------------------------------------------------------------------
## Phase 7: Create config.php for Moodle
1. **Access installer in browser**:
   → `http://YOUR_PUBLIC_IP` → Proceed to database config screen → Copy generated `config.php`.

2. **Create file**:
   ```bash
   cd /var/www/moodle
   sudo nano config.php
   ```
   Paste example:
   ```php
   <?php
   unset($CFG);
   $CFG = new stdClass();

   $CFG->dbtype    = 'mysqli';
   $CFG->dblibrary = 'native';
   $CFG->dbhost    = 'localhost';
   $CFG->dbname    = 'moodle';
   $CFG->dbuser    = 'moodleuser';
   $CFG->dbpass    = 'StrongPassword123!';
   $CFG->prefix    = 'mdl_';
   $CFG->dboptions = array (
     'dbpersist' => 0,
     'dbsocket' => 0,
   );

   $CFG->wwwroot   = 'http://16.171.168.250';
   $CFG->dataroot  = '/var/www/moodledata';
   $CFG->admin     = 'admin';

   require_once(__DIR__ . '/lib/setup.php');
   ```
   → Save & exit.

3. **Set permissions**:
   ```bash
   sudo chown -R www-data:www-data /var/www/moodle
   sudo chmod -R 755 /var/www/moodle
   ```

> **<span style="color:red;">Note: Update $CFG->wwwroot with your actual IP/domain</span>**

<p align="center">
  <img src="https://example.com/moodle-config.png"
       alt="Moodle config.php Generation"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>Installer-Generated Config — Paste directly into nano</em>
</p>

-----------------------------------------------------------------------------------------------------------------------------------------------
## Phase 8: Install Missing PHP Extensions (Required by Moodle)
1. **Install core extensions**:
   ```bash
   sudo apt install php-xml php-mbstring php-curl php-zip php-gd php-intl php-soap php-xmlrpc php-ldap -y
   ```

2. **PHP 8.3 specifics**:
   ```bash
   sudo apt install php8.3-curl php8.3-zip php8.3-gd php8.3-intl php8.3-soap php8.3-mbstring php8.3-xml php8.3-xmlrpc -y
   ```

3. **Tune PHP limits**:
   ```bash
   sudo nano /etc/php/8.3/apache2/php.ini
   ```
   Search (`Ctrl+W`) & update:
   ```ini
   max_input_vars = 5000
   upload_max_filesize = 100M
   post_max_size = 100M
   memory_limit = 512M
   ```
   → Save & restart:
   ```bash
   sudo systemctl restart apache2
   ```

<p align="center">
  <img src="https://example.com/php-extensions.png"
       alt="PHP Extensions Installation"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>Required Extensions — Fixes Moodle environment checks</em>
</p>

-----------------------------------------------------------------------------------------------------------------------------------------------
## Phase 9: Install Composer (Required for Moodle Plugins & Updates)
1. **Download installer**:
   ```bash
   cd ~
   php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
   ```

2. **Verify hash**:
   ```bash
   php -r "if (hash_file('sha384', 'composer-setup.php') === 'c8b085408188070d5f52bcfe4ecfbee5f727afa458b2573b8eaaf77b3419b0bf2768dc67c86944da1544f06fa544fd47') { echo 'Installer verified'.PHP_EOL; } else { echo 'Installer corrupt'.PHP_EOL; unlink('composer-setup.php'); }"
   ```

3. **Run & install**:
   ```bash
   php composer-setup.php
   php -r "unlink('composer-setup.php');"
   sudo mv composer.phar /usr/local/bin/composer
   ```

4. **Install Moodle deps**:
   ```bash
   cd /var/www/moodle
   sudo composer install --no-dev --optimize-autoloader
   ```

> **<span style="color:red;">Note: Run as www-data if permission issues: `sudo -u www-data composer install`</span>**

<p align="center">
  <img src="https://example.com/composer-install.png"
       alt="Composer Setup"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>Composer Verification — Ensures secure dependency management</em>
</p>

-----------------------------------------------------------------------------------------------------------------------------------------------
## Phase 10: Complete Moodle Installation via Web Browser
1. **Resume installer**:
   → `http://YOUR_PUBLIC_IP` → Click **Continue** on all green checks.

2. **Admin setup**:
   - **Site name**: Your LMS Name
   - **Admin username**: `admin`
   - **Password**: Strong password
   - **Email**: `admin@example.com`

3. **Finalize**:
   → Click **"Update profile"** → **Moodle installed successfully**!

4. **Post-install**:
   → Login as admin → Explore dashboard.

> **<span style="color:red;">Success! Your Moodle 5.1 is live. Next: Enable HTTPS with Certbot</span>**

<p align="center">
  <img src="https://example.com/moodle-complete.png"
       alt="Moodle Installation Complete"
       width="800"
       style="border:2px solid #0078D4; border-radius:8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  <br><em>Admin Profile Setup — Welcome to your new LMS</em>
</p>

<div align="center" style="background: #f8f9fa; padding: 20px; border-radius: 12px; border: 2px solid #0078D4; box-shadow: 0 4px 12px rgba(0,0,0,0.1); max-width: 600px; margin: 30px auto;">
  <h3 style="color: #00BCF2; margin: 0 0 10px;">Get in Touch</h3>
  <p style="margin: 5px 0; color: #333;">
    <strong>Harsha Jayawardhana</strong>
  </p>
  <p style="margin: 10px 0;">
    <a href="mailto:harshakunkuma98@gmail.com" style="color: #D83B01; text-decoration: none;">harshakunkuma98@gmail.com</a> •
    <a href="https://www.linkedin.com/in/harsha-jayawardhana-5bbb85216/" target="_blank">LinkedIn</a> •
    <a href="https://github.com/harsha19980324" target="_blank">GitHub</a>
  </p>
  <p style="margin: 5px 0; font-size: 14px; color: #555;">
    <em>Love open-source? Let’s collaborate!</em>
  </p>
</div>
