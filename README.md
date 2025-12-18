# CyberPanel Installation & Usage Guide
## Complete Documentation for Raspberry Pi 5 + Ubuntu Server 24.04

**Author:** Md Arif Billah  
**Date:** December 2024  
**Hardware:** Raspberry Pi 5 (4GB/8GB)  
**OS:** Ubuntu Server 24.04 LTS  
**Power:** Respberi pi official (5V/5A)

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [System Preparation](#system-preparation)
3. [CyberPanel Installation](#cyberpanel-installation)
4. [Accessing CyberPanel](#accessing-cyberpanel)
5. [SSH Connection Setup](#ssh-connection-setup)
6. [FTP Configuration](#ftp-configuration)
7. [File Management](#file-management)
8. [Creating Your First Website](#creating-your-first-website)
9. [Performance Optimization](#performance-optimization)
10. [Troubleshooting](#troubleshooting)
11. [Maintenance & Monitoring](#maintenance--monitoring)

---

## Prerequisites

### Hardware Requirements
- **Raspberry Pi 5** (4GB or 8GB RAM recommended)
- **Power Supply:** 27W USB-C (recommended) or 3A minimum
- **Storage:** 32GB+ microSD card (Class 10/U1)
- **Network:** WiFi or Ethernet connection
- **Cooling:** Active cooling (fan) recommended

### Software Requirements
- **OS:** Ubuntu Server 24.04 LTS (64-bit)
- **Internet:** Active internet connection
- **SSH Client:** PuTTY (Windows) or Terminal (Linux/Mac)

### Power Considerations
⚠️ **Important:** If using power bank (3A), system will work for light loads but official 27W PSU recommended for production.

**Current Power Status:**
- Power Bank: 15W (5V/3A) ⚠️ Marginal
- Recommended: 27W (5V/5A) ✓ Optimal

---

## System Preparation

### Step 1: Connect to Your Pi

#### Finding Your Pi's IP Address

**Method 1: Using Hostname (Easiest)**
```bash
# From your laptop/desktop:
ping gub-server.local

# Or directly SSH:
ssh gub@gub-server.local
```

**Method 2: Router Admin Panel**
1. Open browser: `http://192.168.1.1` or `http://192.168.0.1`
2. Login with router credentials
3. Find "Connected Devices" or "DHCP Client List"
4. Look for device named "gub-server" or "ubuntu"
5. Note the IP address (e.g., `192.168.1.105`)

**Method 3: Using nmap (Advanced)**
```bash
# Install nmap:
sudo apt install nmap

# Scan your network:
nmap -sn 192.168.1.0/24

# Look for "gub-server" in results
```

### Step 2: Initial System Update

```bash
# SSH to your Pi:
ssh gub@gub-server.local
# Enter password when prompted

# Update system:
sudo apt update
sudo apt upgrade -y

# Install essential tools:
sudo apt install -y \
  curl \
  wget \
  git \
  htop \
  net-tools \
  ufw

# Reboot if kernel updated:
sudo reboot

# Wait 2 minutes, then SSH again
```

### Step 3: Configure Firewall

```bash
# Enable firewall with essential ports:
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 80/tcp      # HTTP
sudo ufw allow 443/tcp     # HTTPS
sudo ufw allow 8090/tcp    # CyberPanel
sudo ufw allow 21/tcp      # FTP
sudo ufw allow 40110:40210/tcp  # FTP Passive Ports

# Enable firewall:
sudo ufw enable

# Check status:
sudo ufw status
```

### Step 4: Set Correct Timezone

```bash
# Set timezone to Bangladesh:
sudo timedatectl set-timezone Asia/Dhaka

# Verify:
timedatectl
```

### Step 5: Monitor Power During Installation

**Open two SSH sessions:**

**Terminal 1:** For installation
```bash
ssh gub@gub-server.local
```

**Terminal 2:** For monitoring
```bash
ssh gub@gub-server.local

# Run power monitor:
watch -n 2 'echo "=== System Monitor ===" && date && echo "" && vcgencmd measure_temp && vcgencmd get_throttled && vcgencmd measure_volts core && echo "" && free -h | grep Mem && echo "" && uptime'
```

**Understanding Monitor Output:**
- `temp=65.0'C` - Normal during installation (up to 75°C OK)
- `throttled=0x0` - Perfect! No issues
- `throttled=0x50005` - Under-voltage (acceptable during install)
- `volt=0.85V` - Good voltage
- `volt=0.80V` - Low voltage warning

---

## CyberPanel Installation

### Installation Process (60-90 minutes)

#### Step 1: Download and Run Installer

```bash
# Make sure you're root or use sudo for entire installation:
sudo su -

# Download and run CyberPanel installer:
sh <(curl https://cyberpanel.net/install.sh || wget -O - https://cyberpanel.net/install.sh)
```

#### Step 2: Installation Options

**Select these options when prompted:**

```
1. Install CyberPanel
   Choose: 1

2. Install Type
   Choose: 1 (Full Service)

3. Web Server
   Choose: 1 (OpenLiteSpeed)
   
   Reason: OpenLiteSpeed is lighter and faster than LiteSpeed Enterprise
   Perfect for Pi 5 limited resources

4. CyberPanel Version
   Choose: 1 (Latest Stable)

5. Default Admin Password
   Enter: [Your secure password]
   
   ⚠️ IMPORTANT: Write this down! You'll need it to login

6. Memcached
   Choose: Y (Yes)
   
   Reason: Improves performance significantly

7. Redis
   Choose: Y (Yes)
   
   Reason: Essential for caching

8. Watchdog
   Choose: Y (Yes)
   
   Reason: Monitors and restarts services if they crash

9. Email Server (Optional)
   Choose: N (No) - Unless you need email hosting
   
   Reason: Email server is resource-intensive

10. Remote MySQL (Optional)
    Choose: N (No)
    
    Reason: Use local MySQL for better performance

11. DNS Server
    Choose: Y (Yes)
    
    Reason: Useful for managing domains locally

12. FTP Server
    Choose: Y (Yes) - IMPORTANT!
    
    Reason: You'll need FTP to upload files
```

#### Step 3: Monitor Installation Progress

**What happens during installation:**

```
Phase 1: Package Installation (15 min)
├─ Downloading packages
├─ Installing dependencies
├─ Power draw: ~11-13W
└─ Status: Should be stable

Phase 2: Compiling Modules (20-30 min)
├─ Building OpenLiteSpeed
├─ Compiling PHP
├─ Power draw: ~13-14W ⚠️
├─ CPU usage: High
├─ Temperature: 65-75°C
└─ Status: May see throttling (normal)

Phase 3: Configuration (10 min)
├─ Setting up databases
├─ Configuring services
├─ Power draw: ~9-10W
└─ Status: Stable

Phase 4: Service Startup (5 min)
├─ Starting all services
├─ Final configurations
└─ Status: Completing

Total Time:
- With 27W PSU: ~50-60 minutes
- With Power Bank (3A): ~70-90 minutes (slower due to throttling)
```

#### Step 4: Installation Complete

**When installation finishes, you'll see:**

```
########################################################################
#                                                                      #
#                    CyberPanel Successfully Installed                 #
#                                                                      #
########################################################################

Visit: https://YOUR_PI_IP:8090
Username: admin
Password: [Your password]

Web Panel: https://YOUR_PI_IP:8090
SSH/SFTP Terminal: https://YOUR_PI_IP:8090/terminal
Webmail: https://YOUR_PI_IP:8090/rainloop

########################################################################
```

**Save this information!**

#### Step 5: Post-Installation Setup

```bash
# Exit root shell:
exit

# Reboot system:
sudo reboot

# Wait 2-3 minutes, then SSH back:
ssh gub@gub-server.local

# Verify services running:
sudo systemctl status lsws        # OpenLiteSpeed
sudo systemctl status mariadb     # Database
sudo systemctl status cyberpanel  # CyberPanel

# All should show "active (running)" ✓
```

---

## Accessing CyberPanel

### Web Panel Access

#### Step 1: Open CyberPanel in Browser

```
URL: https://YOUR_PI_IP:8090

Examples:
- https://192.168.1.105:8090
- https://gub-server.local:8090
```

#### Step 2: SSL Certificate Warning

**You'll see a security warning - THIS IS NORMAL!**

**Chrome:**
1. Click "Advanced"
2. Click "Proceed to gub-server.local (unsafe)"

**Firefox:**
1. Click "Advanced"
2. Click "Accept the Risk and Continue"

**Edge:**
1. Click "Advanced"
2. Click "Continue to gub-server.local"

**Why this happens:** CyberPanel uses self-signed SSL certificate. This is safe for local network access.

#### Step 3: Login

```
Username: admin
Password: [Your password from installation]
```

#### Step 4: First Login Dashboard

**You'll see:**
- Server statistics (CPU, RAM, Disk)
- Quick actions menu
- Service status
- Recent activities

**Initial Setup Wizard (if appears):**
1. Skip or complete basic setup
2. Ignore SSL certificate setup for now
3. Continue to dashboard

---

## SSH Connection Setup

### Basic SSH Connection

#### From Linux/Mac:

```bash
# Standard connection:
ssh gub@gub-server.local

# Or with IP:
ssh gub@192.168.1.105

# Specify port (default 22):
ssh -p 22 gub@gub-server.local
```

#### From Windows:

**Method 1: Windows PowerShell (Built-in)**
```powershell
# Open PowerShell
# Windows Key + X, select "Windows PowerShell"

# Connect:
ssh gub@gub-server.local
```

**Method 2: PuTTY**
1. Download PuTTY: https://www.putty.org/
2. Install and open PuTTY
3. Configuration:
   - Host Name: `gub-server.local` or `192.168.1.105`
   - Port: `22`
   - Connection type: `SSH`
4. Click "Open"
5. Accept security alert (first time only)
6. Login:
   - Username: `gub`
   - Password: [your password]

### SSH Key Authentication (Advanced - Recommended)

**Benefits:**
- No password needed
- More secure
- Faster login
- Required for automation

#### Step 1: Generate SSH Key (On Your Laptop)

**Linux/Mac:**
```bash
# Generate key:
ssh-keygen -t ed25519 -C "gub-laptop"

# Output:
# Generating public/private ed25519 key pair.
# Enter file in which to save the key (/home/user/.ssh/id_ed25519):
# [Press Enter]

# Enter passphrase (optional but recommended):
# [Enter passphrase or press Enter for no passphrase]
```

**Windows (PowerShell):**
```powershell
# Generate key:
ssh-keygen -t ed25519 -C "gub-laptop"

# Follow same prompts as Linux/Mac
```

#### Step 2: Copy Key to Pi

**Linux/Mac:**
```bash
# Copy key:
ssh-copy-id gub@gub-server.local

# Enter your Pi password one last time
```

**Windows:**
```powershell
# Copy key manually:
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh gub@gub-server.local "cat >> ~/.ssh/authorized_keys"

# Enter password
```

#### Step 3: Test Key Authentication

```bash
# Try connecting (should not ask for password):
ssh gub@gub-server.local

# Should login directly! ✓
```

#### Step 4: Disable Password Authentication (Optional - More Secure)

```bash
# SSH to Pi:
ssh gub@gub-server.local

# Edit SSH config:
sudo nano /etc/ssh/sshd_config

# Find and change these lines:
PasswordAuthentication no
PubkeyAuthentication yes

# Save and exit (Ctrl+X, Y, Enter)

# Restart SSH:
sudo systemctl restart ssh

# ⚠️ Make sure key authentication works before doing this!
```

### Creating SSH Config (Shortcut)

**On your laptop, create config file:**

**Linux/Mac:**
```bash
# Edit config:
nano ~/.ssh/config

# Add:
Host pi
    HostName gub-server.local
    User gub
    Port 22

Host pi-office
    HostName gub-server.local
    User gub
    Port 22

# Save and exit

# Now you can connect with:
ssh pi

# Or:
ssh pi-office
```

**Windows:**
```powershell
# Edit config:
notepad $env:USERPROFILE\.ssh\config

# Add same content as Linux/Mac
# Save file

# Connect with:
ssh pi
```

---

## FTP Configuration

### Understanding FTP Access

CyberPanel uses **Pure-FTPd** server. There are **two types of FTP access:**

1. **Admin FTP** - Full server access (root level)
2. **Website FTP** - Per-website access (restricted)

### Option 1: Admin FTP Access (Full Access)

#### Step 1: Create FTP Admin User

```bash
# SSH to Pi:
ssh gub@gub-server.local

# Create FTP user with admin privileges:
sudo pure-pw useradd gubftp -u root -d /home

# You'll be prompted for password:
# Password: [enter secure password]
# Enter it again: [confirm password]

# Update FTP database:
sudo pure-pw mkdb

# Restart FTP:
sudo systemctl restart pure-ftpd
```

#### Step 2: Connect with FTP Client

**Install FileZilla (Recommended):**
- Download: https://filezilla-project.org/
- Available for Windows, Mac, Linux

**FileZilla Configuration:**
```
Host: sftp://gub-server.local   (or sftp://192.168.1.105)
Username: gub
Password: [your Pi password]
Port: 22
```

**Or using FTP:**
```
Host: ftp://gub-server.local    (or ftp://192.168.1.105)
Username: gubftp
Password: [FTP password you set]
Port: 21
```

**Quick Connect:**
1. Open FileZilla
2. File → Site Manager → New Site
3. Fill details:
   - Protocol: `SFTP - SSH File Transfer Protocol` (recommended)
   - Host: `gub-server.local`
   - Logon Type: `Normal`
   - User: `gub`
   - Password: [your password]
4. Click "Connect"

### Option 2: Website-Specific FTP (Recommended for Clients)

#### Step 1: Create FTP Account via CyberPanel

1. **Login to CyberPanel:** `https://gub-server.local:8090`

2. **Navigate:**
   - FTP → Create FTP Account

3. **Fill Details:**
   ```
   Select Website: [your-domain.com]
   Username: mysite_ftp
   Password: [secure password]
   Path: /home/your-domain.com/public_html
   ```

4. **Click:** "Create FTP Account"

#### Step 2: Connect to Website FTP

**FileZilla Settings:**
```
Host: ftp://gub-server.local
Username: mysite_ftp@your-domain.com
Password: [password from CyberPanel]
Port: 21
```

**Access Restriction:**
- Can only access `/home/your-domain.com/public_html`
- Cannot see other websites
- Safe for giving to clients

### FTP Troubleshooting

#### Problem: Connection Refused

```bash
# Check FTP service:
sudo systemctl status pure-ftpd

# If not running:
sudo systemctl start pure-ftpd
sudo systemctl enable pure-ftpd

# Check firewall:
sudo ufw allow 21/tcp
sudo ufw allow 40110:40210/tcp
```

#### Problem: Can't Login

```bash
# List FTP users:
sudo pure-pw list

# Reset password:
sudo pure-pw passwd gubftp
# Enter new password

# Update database:
sudo pure-pw mkdb

# Restart service:
sudo systemctl restart pure-ftpd
```

#### Problem: Passive Mode Issues

```bash
# Edit FTP config:
sudo nano /etc/pure-ftpd/pure-ftpd.conf

# Find and set:
PassivePortRange    40110 40210

# Save and restart:
sudo systemctl restart pure-ftpd
```

---

## File Management

### Method 1: CyberPanel File Manager (Easiest)

#### Access File Manager:

1. **Login to CyberPanel**
2. **Navigate:** File Manager → Choose Website
3. **Select Website:** your-domain.com
4. **Click:** "Launch File Manager"

**Features:**
- Upload files (drag & drop)
- Download files
- Edit files (text editor)
- Create folders
- Change permissions
- Compress/Extract files
- View file properties

**Upload Limits:**
- Single file: Up to 100MB
- Multiple files: Drag and drop folder

#### Editing Files via File Manager:

1. Navigate to file
2. Right-click → Edit
3. Edit in browser
4. Save

### Method 2: SFTP (Secure FTP)

**Why SFTP over FTP:**
- Encrypted connection (secure)
- Uses SSH (port 22, always open)
- No need for separate FTP server
- Better for single-user access

#### FileZilla with SFTP:

```
Protocol: SFTP - SSH File Transfer Protocol
Host: gub-server.local
Port: 22
Username: gub
Password: [your Pi password]
```

**Navigate to:**
- Website files: `/home/your-domain.com/public_html/`
- All websites: `/home/`
- System config: `/etc/`
- Logs: `/usr/local/lsws/logs/`

### Method 3: Command Line (SSH)

#### Basic File Operations:

```bash
# SSH to Pi:
ssh gub@gub-server.local

# Navigate to website:
cd /home/your-domain.com/public_html/

# List files:
ls -lah

# Create folder:
mkdir assets

# Create file:
nano index.php

# Copy file:
cp index.php index.php.backup

# Move/Rename file:
mv old-name.php new-name.php

# Delete file:
rm file.php

# Delete folder:
rm -rf folder-name/

# Change ownership (important!):
sudo chown -R nobody:nogroup /home/your-domain.com/public_html/

# Change permissions:
chmod 644 index.php      # Files: 644
chmod 755 folder/        # Folders: 755
```

#### Upload File via SSH (SCP):

**From your laptop:**

```bash
# Upload single file:
scp /local/path/file.php gub@gub-server.local:/home/your-domain.com/public_html/

# Upload entire folder:
scp -r /local/path/folder/ gub@gub-server.local:/home/your-domain.com/public_html/

# Download file:
scp gub@gub-server.local:/home/your-domain.com/public_html/config.php /local/path/

# Download folder:
scp -r gub@gub-server.local:/home/your-domain.com/public_html/ /local/path/
```

### Method 4: WinSCP (Windows)

**Download:** https://winscp.net/

**Configuration:**
```
File Protocol: SFTP
Host name: gub-server.local
Port: 22
Username: gub
Password: [your password]
```

**Features:**
- Drag and drop interface
- Dual-pane (local & remote)
- Built-in text editor
- Synchronized browsing
- Compare directories

---

## Creating Your First Website

### Step 1: Create Website via CyberPanel

1. **Login to CyberPanel:** `https://gub-server.local:8090`

2. **Navigate:**
   - Websites → Create Website

3. **Fill Details:**
   ```
   Select Package: Default
   Select Owner: admin
   Domain Name: test.local (or your-domain.com)
   Email: admin@test.local
   PHP: 8.1 (recommended)
   ```

4. **Advanced Options (Optional):**
   ```
   Open Basedir Protection: OFF (for testing)
   ```

5. **Click:** "Create Website"

**Wait 10-20 seconds for creation to complete.**

### Step 2: Access Website Files

**Location:** `/home/test.local/public_html/`

**Default structure:**
```
/home/test.local/
├── public_html/          # Your website files here
│   └── index.html        # Default homepage
├── logs/                 # Access & error logs
├── ssl/                  # SSL certificates
└── backup/               # Website backups
```

### Step 3: Create Simple PHP Website

#### Option A: Via File Manager

1. CyberPanel → File Manager → test.local
2. Navigate to `public_html/`
3. Delete default `index.html`
4. Create → New File → `index.php`
5. Edit file, paste code (see below)
6. Save

#### Option B: Via SSH

```bash
# SSH to Pi:
ssh gub@gub-server.local

# Navigate to website:
cd /home/test.local/public_html/

# Remove default file:
sudo rm index.html

# Create new index.php:
sudo nano index.php
```

**Paste this code:**

```php
<?php
// Simple PHP Website Example
// Perfect for testing your CyberPanel setup

// Database connection (optional)
$host = 'localhost';
$dbname = 'test_local';
$username = 'test_local';
$password = '[your-db-password]';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$dbname", $username, $password);
    $dbStatus = "✓ Database Connected";
} catch(PDOException $e) {
    $dbStatus = "✗ Database Connection Failed";
}

// Server info
$serverInfo = [
    'PHP Version' => phpversion(),
    'Server Software' => $_SERVER['SERVER_SOFTWARE'],
    'Document Root' => $_SERVER['DOCUMENT_ROOT'],
    'Server IP' => $_SERVER['SERVER_ADDR'],
    'Client IP' => $_SERVER['REMOTE_ADDR'],
];
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My CyberPanel Website</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        .container {
            background: white;
            border-radius: 20px;
            padding: 40px;
            max-width: 800px;
            width: 100%;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
        }
        h1 {
            color: #667eea;
            margin-bottom: 10px;
            font-size: 2.5em;
        }
        .subtitle {
            color: #666;
            margin-bottom: 30px;
            font-size: 1.1em;
        }
        .status {
            background: #f0f0f0;
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 20px;
        }
        .status.success {
            background: #d4edda;
            color: #155724;
        }
        .info-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 15px;
            margin-top: 20px;
        }
        .info-card {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 10px;
            border-left: 4px solid #667eea;
        }
        .info-card h3 {
            color: #667eea;
            font-size: 0.9em;
            margin-bottom: 10px;
            text-transform: uppercase;
        }
        .info-card p {
            color: #333;
            font-size: 1.1em;
            word-break: break-all;
        }
        .footer {
            margin-top: 30px;
            padding-top: 20px;
            border-top: 2px solid #eee;
            text-align: center;
            color: #666;
        }
        .badge {
            display: inline-block;
            background: #667eea;
            color: white;
            padding: 5px 15px;
            border-radius: 20px;
            font-size: 0.9em;
            margin: 5px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 Welcome to CyberPanel!</h1>
        <p class="subtitle">Your website is running on Raspberry Pi 5</p>
        
        <div class="status <?php echo strpos($dbStatus, '✓') !== false ? 'success' : ''; ?>">
            <strong>Database Status:</strong> <?php echo $dbStatus; ?>
        </div>
        
        <h2 style="color: #667eea; margin-top: 30px; margin-bottom: 15px;">Server Information</h2>
        
        <div class="info-grid">
            <?php foreach($serverInfo as $label => $value): ?>
                <div class="info-card">
                    <h3><?php echo $label; ?></h3>
                    <p><?php echo htmlspecialchars($value); ?></p>
                </div>
            <?php endforeach; ?>
        </div>
        
        <div style="margin-top: 30px;">
            <h2 style="color: #667eea; margin-bottom: 15px;">Features</h2>
            <span class="badge">PHP <?php echo phpversion(); ?></span>
            <span class="badge">OpenLiteSpeed</span>
            <span class="badge">MariaDB</span>
            <span class="badge">Redis</span>
            <span class="badge">Memcached</span>
        </div>
        
        <div class="footer">
            <p>Powered by CyberPanel on Raspberry Pi 5</p>
            <p style="margin-top: 5px; font-size: 0.9em;">
                Hosted by gub • <?php echo date('Y'); ?>
            </p>
        </div>
    </div>
</body>
</html>
```

**Save and exit:**
- Nano: `Ctrl+X`, `Y`, `Enter`
- FileZilla: Right-click → Edit → Save

#### Fix Permissions:

```bash
# Set correct ownership:
sudo chown -R nobody:nogroup /home/test.local/public_html/

# Set permissions:
sudo chmod -R 755 /home/test.local/public_html/
sudo find /home/test.local/public_html/ -type f -exec chmod 644 {} \;
```

### Step 4: Create Database (Optional)

1. **CyberPanel → Database → Create Database**

2. **Fill Details:**
   ```
   Select Website: test.local
   Database Name: test_local
   Database Username: test_local
   Password: [secure password]
   ```

3. **Click:** "Create Database"

4. **Update PHP file with database credentials**

### Step 5: Access Your Website

**In browser, visit:**
```
http://gub-server.local
http://192.168.1.105
http://test.local (if configured in hosts file)
```

**You should see your website! 🎉**

### Step 6: Configure Local Domain (Optional)

**On your laptop (for development):**

**Windows:**
```powershell
# Run as Administrator:
notepad C:\Windows\System32\drivers\etc\hosts

# Add line:
192.168.1.105 test.local

# Save and close
```

**Linux/Mac:**
```bash
# Edit hosts file:
sudo nano /etc/hosts

# Add line:
192.168.1.105 test.local

# Save and exit
```

**Now visit:** `http://test.local` - Should work! ✓

---

## Performance Optimization

### For Power Bank (3A) Users

Since you're using 3A power supply, optimize for lower power consumption:

#### Step 1: Reduce OpenLiteSpeed Connections

```bash
# Edit LiteSpeed config:
sudo nano /usr/local/lsws/conf/httpd_config.conf

# Find and modify these values:
maxConnections              150     # Down from 500
maxSSLConnections          100     # Down from 300
connTimeout                 60      # Up from 30
keepAliveTimeout           5       # Down from 10

# Save and restart:
sudo systemctl restart lsws
```

#### Step 2: Optimize PHP-FPM

```bash
# Edit PHP-FPM config (adjust PHP version as needed):
sudo nano /usr/local/lsws/lsphp81/etc/php-fpm.d/www.conf

# Modify:
pm = dynamic
pm.max_children = 5          # Down from 20
pm.start_servers = 2         # Down from 5
pm.min_spare_servers = 1     # Down from 5
pm.max_spare_servers = 3     # Down from 10
pm.max_requests = 500        # Same

# Restart:
sudo killall lsphp
sudo systemctl restart lsws
```

#### Step 3: Optimize MySQL/MariaDB

```bash
# Edit MySQL config:
sudo nano /etc/mysql/my.cnf

# Add under [mysqld]:
[mysqld]
performance_schema = OFF
innodb_buffer_pool_size = 128M
max_connections = 50
table_open_cache = 64
query_cache_type = 1
query_cache_size = 16M

# Save and restart:
sudo systemctl restart mariadb
```

#### Step 4: Configure Redis for Low Memory

```bash
# Edit Redis config:
sudo nano /etc/redis/redis.conf

# Find and modify:
maxmemory 128mb
maxmemory-policy allkeys-lru
save ""  # Disable persistence (optional)

# Restart:
sudo systemctl restart redis
```

#### Step 5: Enable OPcache

```bash
# Edit PHP config:
sudo nano /usr/local/lsws/lsphp81/etc/php.ini

# Find [opcache] section and enable:
opcache.enable=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=60

# Restart:
sudo systemctl restart lsws
```

### Monitoring Performance

#### Create Performance Monitor Script:

```bash
cat > ~/performance_monitor.sh << 'EOF'
#!/bin/bash

while true; do
    clear
    echo "=========================================="
    echo "  CyberPanel Performance Monitor"
    echo "=========================================="
    echo ""
    
    # Power Status
    echo "=== Power Status ==="
    TEMP=$(vcgencmd measure_temp | cut -d= -f2)
    THROTTLED=$(vcgencmd get_throttled)
    VOLT=$(vcgencmd measure_volts core | cut -d= -f2)
    echo "Temperature: $TEMP"
    echo "Voltage: $VOLT"
    echo "Throttled: $THROTTLED"
    echo ""
    
    # System Resources
    echo "=== System Resources ==="
    echo "CPU Load:"
    uptime | awk -F'load average:' '{print $2}'
    echo ""
    echo "Memory:"
    free -h | grep Mem | awk '{print "Used: "$3" / "$2" ("$3/$2*100"%)"}'
    echo ""
    echo "Disk:"
    df -h / | tail -1 | awk '{print "Used: "$3" / "$2" ("$5")"}'
    echo ""
    
    # Service Status
    echo "=== Services ==="
    systemctl is-active lsws && echo "OpenLiteSpeed: ✓ Running" || echo "OpenLiteSpeed: ✗ Stopped"
    systemctl is-active mariadb && echo "MariaDB: ✓ Running" || echo "MariaDB: ✗ Stopped"
    systemctl is-active redis && echo "Redis: ✓ Running" || echo "Redis: ✗ Stopped"
    echo ""
    
    # Web Server Stats
    echo "=== Web Server ==="
    CONNECTIONS=$(netstat -an | grep :80 | grep ESTABLISHED | wc -l)
    echo "Active connections: $CONNECTIONS"
    echo ""
    
    echo "Press Ctrl+C to exit"
    echo "Updating in 5 seconds..."
    
    sleep 5
done
EOF

chmod +x ~/performance_monitor.sh

# Run it:
~/performance_monitor.sh
```

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Can't Access CyberPanel (Port 8090)

**Symptoms:**
- Browser shows "Connection refused" or timeout
- `https://gub-server.local:8090` doesn't load

**Solutions:**

```bash
# Check if CyberPanel service running:
sudo systemctl status cyberpanel

# If not running, start it:
sudo systemctl start cyberpanel
sudo systemctl enable cyberpanel

# Check firewall:
sudo ufw status
sudo ufw allow 8090/tcp

# Check if port listening:
sudo netstat -tulpn | grep 8090

# Restart CyberPanel:
sudo systemctl restart cyberpanel

# Check logs:
sudo tail -f /usr/local/CyberCP/log/CyberCP.log
```

#### Issue 2: Website Shows 503 Error

**Symptoms:**
- Website not loading
- Shows "503 Service Unavailable"

**Solutions:**

```bash
# Check OpenLiteSpeed status:
sudo systemctl status lsws

# Restart OpenLiteSpeed:
sudo systemctl restart lsws

# Check if website exists:
ls -la /home/test.local/public_html/

# Check permissions:
sudo chown -R nobody:nogroup /home/test.local/public_html/
sudo chmod -R 755 /home/test.local/public_html/

# Check error logs:
sudo tail -f /usr/local/lsws/logs/error.log
```

#### Issue 3: FTP Connection Failed

**Solutions:**

```bash
# Check FTP service:
sudo systemctl status pure-ftpd

# Start if stopped:
sudo systemctl start pure-ftpd
sudo systemctl enable pure-ftpd

# Check firewall:
sudo ufw allow 21/tcp
sudo ufw allow 40110:40210/tcp

# Test FTP locally:
ftp localhost

# Check FTP users:
sudo pure-pw list
```

#### Issue 4: Database Connection Error

**Solutions:**

```bash
# Check MariaDB status:
sudo systemctl status mariadb

# Restart MariaDB:
sudo systemctl restart mariadb

# Login to MySQL:
sudo mysql

# Check databases:
SHOW DATABASES;

# Check users:
SELECT user, host FROM mysql.user;

# Create database manually if needed:
CREATE DATABASE test_local;
CREATE USER 'test_local'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON test_local.* TO 'test_local'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

#### Issue 5: High CPU/Memory Usage

**Solutions:**

```bash
# Check processes:
htop

# Or:
top

# Identify resource hogs:
ps aux --sort=-%cpu | head -10
ps aux --sort=-%mem | head -10

# Restart services:
sudo systemctl restart lsws
sudo systemctl restart mariadb

# Clear cache:
sudo rm -rf /tmp/lshttpd/*
redis-cli FLUSHALL
```

#### Issue 6: Under-Voltage Throttling

**Symptoms:**
- System slow
- Random reboots
- `vcgencmd get_throttled` shows `0x50005`

**Solutions:**

```bash
# Check current status:
vcgencmd get_throttled

# Reduce load:
# - Limit concurrent connections (see optimization section)
# - Stop unnecessary services
# - Reduce PHP workers

# Long-term solution:
# Get Official 27W PSU (৳1,500)
```

#### Issue 7: Website File Upload Fails

**Solutions:**

```bash
# Increase upload limits in OpenLiteSpeed:
sudo nano /usr/local/lsws/conf/httpd_config.conf

# Find and increase:
maxReqBodySize              2047M

# Increase PHP limits:
sudo nano /usr/local/lsws/lsphp81/etc/php.ini

# Modify:
upload_max_filesize = 100M
post_max_size = 100M
max_execution_time = 300

# Restart:
sudo systemctl restart lsws
```

### Getting Help

**Log Files Locations:**

```bash
# CyberPanel logs:
/usr/local/CyberCP/log/CyberCP.log

# OpenLiteSpeed logs:
/usr/local/lsws/logs/error.log
/usr/local/lsws/logs/access.log

# Website-specific logs:
/home/test.local/logs/

# System logs:
/var/log/syslog

# View logs in real-time:
sudo tail -f /path/to/log/file

# Search for errors:
sudo grep -i error /usr/local/lsws/logs/error.log
```

**Useful Commands:**

```bash
# Check all services:
sudo systemctl list-units --type=service --state=running

# Restart all CyberPanel services:
sudo systemctl restart lsws mariadb redis pure-ftpd cyberpanel

# Check system resources:
htop
df -h
free -h
```

---

## Maintenance & Monitoring

### Daily Tasks

```bash
# Quick health check:
sudo systemctl status lsws mariadb cyberpanel

# Check disk space:
df -h

# Check memory:
free -h

# Check power status:
vcgencmd get_throttled
vcgencmd measure_temp
```

### Weekly Tasks

#### Backup Websites

```bash
# Manual backup:
sudo mkdir -p /backup/$(date +%Y%m%d)
sudo cp -r /home/test.local /backup/$(date +%Y%m%d)/

# Or use CyberPanel:
# Backup → Create Backup → Select Website
```

#### Update System

```bash
# Update packages:
sudo apt update
sudo apt upgrade -y

# Clean old packages:
sudo apt autoremove -y
sudo apt autoclean
```

#### Clean Logs

```bash
# Remove old logs (older than 7 days):
sudo find /usr/local/lsws/logs/ -name "*.log.*" -mtime +7 -delete
sudo find /home/*/logs/ -name "*.log.*" -mtime +7 -delete

# Compress current logs:
sudo gzip /usr/local/lsws/logs/access.log
sudo gzip /usr/local/lsws/logs/error.log
```

### Monthly Tasks

#### Optimize Databases

```bash
# Optimize all databases:
sudo mysqlcheck -Aos -u root -p

# Or per database:
sudo mysqlcheck -o test_local -u root -p
```

#### Check Security

```bash
# Update CyberPanel:
# Login to CyberPanel → Server Status → Upgrade CyberPanel

# Check for failed login attempts:
sudo grep "Failed password" /var/log/auth.log | tail -20

# Check firewall rules:
sudo ufw status verbose
```

#### Review Performance

```bash
# Generate performance report:
cat > ~/monthly_report.sh << 'EOF'
#!/bin/bash

REPORT="/tmp/cyberpanel_report_$(date +%Y%m%d).txt"

echo "CyberPanel Monthly Report" > $REPORT
echo "Generated: $(date)" >> $REPORT
echo "================================" >> $REPORT
echo "" >> $REPORT

echo "=== System Info ===" >> $REPORT
uname -a >> $REPORT
uptime >> $REPORT
echo "" >> $REPORT

echo "=== Disk Usage ===" >> $REPORT
df -h >> $REPORT
echo "" >> $REPORT

echo "=== Memory Usage ===" >> $REPORT
free -h >> $REPORT
echo "" >> $REPORT

echo "=== Top Processes ===" >> $REPORT
ps aux --sort=-%cpu | head -10 >> $REPORT
echo "" >> $REPORT

echo "=== Service Status ===" >> $REPORT
systemctl status lsws --no-pager >> $REPORT
systemctl status mariadb --no-pager >> $REPORT
echo "" >> $REPORT

echo "=== Website List ===" >> $REPORT
ls -la /home/ | grep -v "^total" >> $REPORT
echo "" >> $REPORT

echo "=== Database List ===" >> $REPORT
sudo mysql -e "SHOW DATABASES;" >> $REPORT
echo "" >> $REPORT

echo "Report saved to: $REPORT"
cat $REPORT
EOF

chmod +x ~/monthly_report.sh
~/monthly_report.sh
```

### Automated Monitoring

#### Setup Monitoring Service

```bash
# Create systemd service:
sudo nano /etc/systemd/system/cyberpanel-monitor.service

# Paste:
[Unit]
Description=CyberPanel Monitoring Service
After=network.target

[Service]
Type=simple
User=gub
ExecStart=/home/gub/performance_monitor_daemon.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

# Create daemon script:
cat > ~/performance_monitor_daemon.sh << 'EOF'
#!/bin/bash

LOG="/var/log/cyberpanel_monitor.log"

while true; do
    TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
    
    # Check services
    LSWS=$(systemctl is-active lsws)
    MYSQL=$(systemctl is-active mariadb)
    
    # Power status
    THROTTLED=$(vcgencmd get_throttled)
    TEMP=$(vcgencmd measure_temp | cut -d= -f2)
    
    # Log
    echo "$TIMESTAMP | LSWS:$LSWS | MySQL:$MYSQL | Throttle:$THROTTLED | Temp:$TEMP" >> $LOG
    
    # Alert if service down
    if [ "$LSWS" != "active" ]; then
        echo "$TIMESTAMP | ALERT: OpenLiteSpeed is down!" >> $LOG
        sudo systemctl restart lsws
    fi
    
    if [ "$MYSQL" != "active" ]; then
        echo "$TIMESTAMP | ALERT: MariaDB is down!" >> $LOG
        sudo systemctl restart mariadb
    fi
    
    sleep 60
done
EOF

chmod +x ~/performance_monitor_daemon.sh

# Enable and start:
sudo systemctl daemon-reload
sudo systemctl enable cyberpanel-monitor
sudo systemctl start cyberpanel-monitor

# Check status:
sudo systemctl status cyberpanel-monitor

# View logs:
sudo tail -f /var/log/cyberpanel_monitor.log
```

---

## Quick Reference

### Essential Commands

```bash
# Service Management
sudo systemctl start lsws          # Start web server
sudo systemctl stop lsws           # Stop web server
sudo systemctl restart lsws        # Restart web server
sudo systemctl status lsws         # Check status

# Same for other services:
sudo systemctl restart mariadb     # Database
sudo systemctl restart pure-ftpd   # FTP
sudo systemctl restart cyberpanel  # CyberPanel

# File Permissions
sudo chown -R nobody:nogroup /path/to/folder
sudo chmod -R 755 /path/to/folder
sudo find /path -type f -exec chmod 644 {} \;

# Database
sudo mysql                         # Login to MySQL
SHOW DATABASES;                    # List databases
USE database_name;                 # Select database
SHOW TABLES;                       # List tables

# Logs
sudo tail -f /usr/local/lsws/logs/error.log
sudo tail -f /home/test.local/logs/access_log

# System Info
vcgencmd measure_temp              # Temperature
vcgencmd get_throttled             # Throttling status
htop                               # Resource monitor
df -h                              # Disk usage
free -h                            # Memory usage
```

### Important Paths

```bash
# Website Files
/home/your-domain.com/public_html/

# CyberPanel
/usr/local/CyberCP/

# OpenLiteSpeed
/usr/local/lsws/

# Logs
/usr/local/lsws/logs/
/home/your-domain.com/logs/

# Configuration
/usr/local/lsws/conf/httpd_config.conf
/etc/mysql/my.cnf
/usr/local/lsws/lsphp*/etc/php.ini
```

### Access URLs

```
CyberPanel:    https://SERVER_IP:8090
Website:       http://SERVER_IP or http://your-domain.com
PHPMyAdmin:    http://SERVER_IP:8090/dataBases/phpMyAdmin
Webmail:       http://SERVER_IP:8090/rainloop
Terminal:      https://SERVER_IP:8090/terminal
```

### Default Credentials

```
CyberPanel Admin:
Username: admin
Password: [Set during installation]

Database Root:
Username: root
Password: [Set during installation]

FTP:
Username: [created-user]@[domain.com]
Password: [Set in CyberPanel]
```

---

## Power Considerations

### Current Setup Analysis

**Your Hardware:**
- Power Source: Baseus PPADM20S (5V/3A = 15W)
- Recommendation: Upgrade to 27W PSU

**Power Usage Estimates:**

```
Idle State:
├─ System base: 7-8W
├─ CyberPanel: 1-2W
└─ Total: ~9-10W ✓ OK

Light Load (1-5 users):
├─ System: 8W
├─ Web server: 2W
├─ Database: 1W
└─ Total: ~11W ✓ OK

Medium Load (10-20 users):
├─ System: 9W
├─ Web server: 3W
├─ Database: 2W
└─ Total: ~14W ⚠️ Borderline

Heavy Load (50+ users):
├─ System: 10W
├─ Web server: 5W
├─ Database: 3W
└─ Total: ~18W ❌ Exceeds capacity
```

**Recommendations:**

```
Current Setup (Power Bank 3A):
✓ Good for: Development, testing, learning
✓ Good for: 1-3 small websites
✓ Good for: Low traffic (<20 concurrent)
⚠️ Avoid: Production hosting
⚠️ Avoid: High traffic sites
⚠️ Avoid: Multiple WordPress installations

Upgrade Path:
→ Official 27W PSU (৳1,500)
→ Full production capability
→ No throttling concerns
→ Reliable 24/7 operation
```

---

## Conclusion

You now have a fully functional CyberPanel installation on your Raspberry Pi 5!

**What you can do:**
- ✅ Host multiple websites
- ✅ Manage via web interface
- ✅ Upload files via FTP/SFTP
- ✅ Create databases
- ✅ Run PHP applications
- ✅ Monitor system health
- ✅ Remote SSH access

**Next Steps:**
1. Create your Softorio.com website
2. Test with client projects
3. Learn advanced CyberPanel features
4. Consider upgrading power supply for production

**Remember:**
- Backup regularly
- Monitor system resources
- Keep software updated
- Use strong passwords
- Consider upgrading to 27W PSU for production use

---

## Support & Resources

### Official Documentation
- CyberPanel Docs: https://cyberpanel.net/docs/
- OpenLiteSpeed: https://openlitespeed.org/kb/
- Ubuntu Server: https://ubuntu.com/server/docs

### Community
- CyberPanel Forum: https://forums.cyberpanel.net/
- Discord: https://discord.gg/cyberpanel
- GitHub: https://github.com/usmannasir/cyberpanel

### Local Contacts
**RoboticsBD Uttara:**
- Address: House#8, Road#15, Sector#7, Uttara, Dhaka
- Phone: 017 83 007 004
- Products: Raspberry Pi supplies, accessories

---

**Document Version:** 1.0  
**Last Updated:** December 2024  
**Author:** Md gub Billah  
**Contact:** [Your contact info]

**Happy Hosting! 🚀**
