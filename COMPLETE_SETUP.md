# 📘 FileFinder Complete Setup Guide for Beginners

**Version:** 7.0  
**Last Updated:** November 28, 2025  
**Author:** FileFinder Team

---

## 🎯 What This Guide Covers

This guide will walk you through the **complete setup** of FileFinder from scratch, even if you've never used MySQL or Linux before. We'll set up:

1. **Windows MySQL Server** - The database where all file data is stored
2. **Linux Scanner Client(s)** - Machines that scan files and send data to the server

**By the end, you'll have:** A working file scanning system where Linux machines scan their filesystems and send all data to a centralized Windows MySQL database.

---

## 📋 Table of Contents

1. [System Requirements](#system-requirements)
2. [Part 1: Windows MySQL Server Setup](#part-1-windows-mysql-server-setup)
3. [Part 2: Linux Scanner Client Setup](#part-2-linux-scanner-client-setup)
4. [Part 3: Testing Connectivity](#part-3-testing-connectivity)
5. [Part 4: Troubleshooting](#part-4-troubleshooting)
6. [Part 5: Running Your First Scan](#part-5-running-your-first-scan)
7. [Part 6: Viewing Results](#part-6-viewing-results)
8. [Common Issues & Solutions](#common-issues--solutions)

---

## 📦 System Requirements

### **Windows Server Machine (Device 1)**
- Windows 10, 11, or Windows Server 2016+
- 4 GB RAM minimum (8 GB recommended)
- 20 GB free disk space
- Administrator access
- Static IP address or DHCP reservation recommended

### **Linux Client Machine (Device 2, 3, etc.)**
- Kali Linux, Ubuntu 20.04+, Debian 11+, or RHEL 8+
- Python 3.11 (exactly - not 3.10 or 3.12)
- 2 GB RAM minimum
- Network connectivity to Windows server
- Sudo/root access for installation

### **Network Requirements**
- Both machines on same network (or VPN connection)
- Port 3306 open between machines (for MySQL)
- Firewall exceptions configured

---

# Part 1: Windows MySQL Server Setup

## 🖥️ Step 1: Prepare Your Windows Machine

### 1.1 - Find Your Windows IP Address

1. Press `Win + R` on keyboard
2. Type `cmd` and press Enter
3. In the black window, type:
   ```cmd
   ipconfig
   ```
4. Look for **IPv4 Address** under your active network adapter
5. **Write this down!** Example: `192.168.0.104`

   ```
   Ethernet adapter Ethernet:
      IPv4 Address. . . . . . . . . . . : 192.168.0.104
   ```

### 1.2 - Set Static IP (Recommended)

To prevent IP from changing:

1. Press `Win + R`, type `ncpa.cpl`, press Enter
2. Right-click your network adapter → **Properties**
3. Double-click **Internet Protocol Version 4 (TCP/IPv4)**
4. Select **Use the following IP address:**
5. Enter:
   - IP address: `192.168.0.104` (the one you wrote down)
   - Subnet mask: `255.255.255.0`
   - Default gateway: `192.168.0.1` (your router IP)
   - Preferred DNS: `8.8.8.8`
6. Click **OK** twice

---

## 🗄️ Step 2: Install MySQL Server on Windows

### 2.1 - Download MySQL

1. Open web browser
2. Go to: https://dev.mysql.com/downloads/mysql/
3. Click **"Download"** for **MySQL Installer for Windows**
4. Click **"No thanks, just start my download"**
5. Save the file (around 300-400 MB)

### 2.2 - Install MySQL

1. **Run the installer** you just downloaded
2. Choose **"Server only"** installation type
3. Click **Next** → **Next** → **Execute** (wait for installation)
4. Click **Next** after installation completes

### 2.3 - Configure MySQL Server

**Type and Networking:**
1. Config Type: **Development Computer**
2. Port: `3306` (default - don't change)
3. ✅ Check **"Open Windows Firewall ports for network access"**
4. Click **Next**

**Authentication Method:**
1. Select **"Use Strong Password Encryption"**
2. Click **Next**

**Accounts and Roles:**
1. Enter a **Root Password** (write this down!)
   - Example: `MyRootPass123!`
2. Click **Add User**
3. Create application user:
   - User Name: `arungt`
   - Host: `%` (means "from anywhere")
   - Password: `fi!ef!ndgt!23`
   - Confirm password: `fi!ef!ndgt!23`
   - Role: `DB Admin`
4. Click **OK**
5. Click **Next**

**Windows Service:**
1. ✅ Check **"Configure MySQL Server as a Windows Service"**
2. ✅ Check **"Start the MySQL Server at System Startup"**
3. Click **Next**

**Apply Configuration:**
1. Click **Execute** (wait for all steps to complete - green checkmarks)
2. Click **Finish**
3. Click **Next** → **Finish** to close installer

### 2.4 - Verify MySQL is Running

1. Press `Win + R`, type `services.msc`, press Enter
2. Scroll down to find **MySQL80** (or **MySQL**)
3. Status should show: **Running**
4. If not running, right-click → **Start**

---

## 📂 Step 3: Copy Server Files to Windows

### 3.1 - Get the FileFinder Server Files

1. Navigate to where you have the FileFinder files
2. Find the folder: **`FileFinder4`**
3. Copy this entire folder to your Windows machine
4. Suggested location: `C:\FileFinder4`

### 3.2 - What's Inside

```
FileFinder4/
├── SQLScripts/
│   ├── COMPLETE_DB_EXPORT.sql          ← Complete database setup
│   ├── 1mysql_user.sql                 ← User creation script
│   ├── 2rec_files.sql                  ← Database tables
│   └── 3_rec_performance_indexes.sql   ← Performance indexes
├── setup_server.bat                    ← Automated setup script
├── README_SERVER.md                    ← Server documentation
└── WINDOWS_FIREWALL_SETUP.md           ← Firewall guide
```

---

## 🛠️ Step 4: Run Database Setup

### 4.1 - Option A: Automated Setup (Recommended for Beginners)

1. Open File Explorer
2. Navigate to `C:\FileFinder4`
3. **Right-click** `setup_server.bat`
4. Select **"Run as administrator"**
5. Follow the prompts:
   - Enter MySQL root password when asked
   - Script will:
     - Create database `rec_files`
     - Create user `arungt`
     - Import all tables
     - Create indexes
     - Configure firewall
6. Look for **"Setup complete!"** message

### 4.2 - Option B: Manual Setup

If automated setup fails, do it manually:

**Step 1: Open Command Prompt as Administrator**
1. Press `Win + X`
2. Select **"Command Prompt (Admin)"** or **"PowerShell (Admin)"**

**Step 2: Navigate to SQL Scripts**
```cmd
cd C:\FileFinder4\SQLScripts
```

**Step 3: Import Database**
```cmd
mysql -u root -p < COMPLETE_DB_EXPORT.sql
```
Enter your MySQL root password when prompted.

**Step 4: Verify Database Created**
```cmd
mysql -u root -p -e "SHOW DATABASES;"
```
You should see `rec_files` in the list.

**Step 5: Verify User Created**
```cmd
mysql -u root -p -e "SELECT user, host FROM mysql.user WHERE user='arungt';"
```
You should see:
```
+--------+------+
| user   | host |
+--------+------+
| arungt | %    |
+--------+------+
```

---

## 🔥 Step 5: Configure Windows Firewall

The MySQL port 3306 must be open for remote connections.

### 5.1 - Open PowerShell as Administrator

1. Press `Win + X`
2. Select **"Windows PowerShell (Admin)"** or **"Terminal (Admin)"**
3. Click **Yes** to allow

### 5.2 - Add Firewall Rule

Copy and paste this command:

```powershell
New-NetFirewallRule -DisplayName "MySQL Server" -Direction Inbound -Protocol TCP -LocalPort 3306 -Action Allow
```

Press Enter. You should see:
```
Name                  : {GUID}
DisplayName           : MySQL Server
Enabled               : True
```

### 5.3 - Verify Firewall Rule

```powershell
Get-NetFirewallRule -DisplayName "MySQL Server" | Select-Object DisplayName, Enabled, Action
```

Should show:
```
DisplayName  Enabled Action
-----------  ------- ------
MySQL Server    True  Allow
```

### 5.4 - Alternative: Windows Firewall GUI

If you prefer clicking instead of typing:

1. Press `Win + R`, type `wf.msc`, press Enter
2. Click **Inbound Rules** (left panel)
3. Click **New Rule...** (right panel)
4. Select **Port** → Click **Next**
5. Select **TCP**, type `3306` → Click **Next**
6. Select **Allow the connection** → Click **Next**
7. Check all boxes (Domain, Private, Public) → Click **Next**
8. Name: `MySQL Server` → Click **Finish**

---

## ⚙️ Step 6: Configure MySQL for Remote Access

### 6.1 - Find MySQL Configuration File

Open File Explorer and navigate to:
```
C:\ProgramData\MySQL\MySQL Server 8.0\my.ini
```

**Note:** ProgramData is a hidden folder. To see it:
1. Click **View** tab in File Explorer
2. Check ✅ **Hidden items**

### 6.2 - Edit my.ini

1. **Right-click Notepad** from Start Menu
2. Select **"Run as administrator"**
3. In Notepad: **File** → **Open**
4. Navigate to `C:\ProgramData\MySQL\MySQL Server 8.0\`
5. Change file filter to **"All Files (*.*)"**
6. Open `my.ini`

### 6.3 - Update bind-address

1. Press `Ctrl + F` to find
2. Search for: `bind-address`
3. You'll find a line like:
   ```ini
   bind-address = 127.0.0.1
   ```
4. Change it to:
   ```ini
   bind-address = 0.0.0.0
   ```
5. Save the file (`Ctrl + S`)
6. Close Notepad

### 6.4 - Restart MySQL Service

1. Press `Win + R`, type `services.msc`, press Enter
2. Find **MySQL80** in the list
3. Right-click → **Restart**
4. Wait for it to show **Running** status

### 6.5 - Verify MySQL is Listening

Open Command Prompt and run:
```cmd
netstat -an | findstr :3306
```

You should see:
```
TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING
```

**Important:** It should say `0.0.0.0:3306`, not `127.0.0.1:3306`

---

## ✅ Step 7: Test Windows Server is Ready

### 7.1 - Test Local MySQL Connection

Open Command Prompt:
```cmd
mysql -u arungt -p
```

Enter password: `fi!ef!ndgt!23`

You should see:
```
mysql>
```

Type:
```sql
SHOW DATABASES;
```

You should see:
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| rec_files          |
+--------------------+
```

Type `exit;` to quit.

### 7.2 - Test Database Structure

```cmd
mysql -u arungt -p rec_files -e "SHOW TABLES;"
```

You should see 9 tables including:
- `f_machine_files_summary_count`
- `d_file_details`
- `audit_info`
- etc.

---

## 🎉 Windows Server Setup Complete!

Your Windows MySQL server is now ready to receive data from Linux scanners!

**What you have:**
- ✅ MySQL Server running
- ✅ Database `rec_files` created
- ✅ User `arungt` with remote access
- ✅ Firewall configured (port 3306 open)
- ✅ MySQL listening on all network interfaces

**Write these down for Linux setup:**
- Windows IP Address: `192.168.0.104` (your actual IP)
- Database Name: `rec_files`
- Username: `arungt`
- Password: `fi!ef!ndgt!23`
- Port: `3306`

---

#Linux Scanner Client Setup

Now we'll set up the Linux machine that will scan files and send data to your Windows MySQL server.

---

## 📦 Step 1: Transfer Files to Linux

### 1.1 - Get the FileFinder User Linux Files

You need the **`FileFinder_User_Linux`** folder on your Linux machine.

**Download/Unzip**
1. If you have a zip file, transfer it to Linux
2. Unzip:
   ```bash
   unzip FileFinder_User_Linux.zip -d ~/
   ```

### 1.2 - Navigate to Folder

```bash
cd ~/FileFinder_User_Linux
```

Or if you placed it elsewhere:
```bash
cd ~/Downloads/filefinder_update/FileFinder_User_Linux
```

### 1.3 - Check Files are Present

```bash
ls -la
```

You should see:
```
file_info_version_22_linux.py
requirements-linux.txt
.env
setup_linux.sh
README_USER.md
SQLScripts/
```

---

## 🐍 Step 2: Install Python 3.11 on Linux

FileFinder requires **Python 3.11 exactly** (not 3.10, not 3.12).

### 2.1 - Update Package Lists

```bash
sudo apt update
```

Enter your password when prompted.

### 2.2 - Install Python 3.11

**For Kali Linux, Ubuntu, Debian:**
```bash
sudo apt install python3.11 python3.11-venv python3.11-dev -y
```

**For RHEL, CentOS, Fedora:**
```bash
sudo dnf install python3.11 python3.11-devel -y
```

### 2.3 - Verify Python Version

```bash
python3.11 --version
```

Should show:
```
Python 3.11.x
```

(Where x can be any number like 0, 1, 2, 9, etc.)

---

## 🛠️ Step 3: Install MySQL Client on Linux

This is needed to test connectivity to the Windows MySQL server.

```bash
sudo apt install mysql-client -y
```

**For RHEL/CentOS:**
```bash
sudo dnf install mysql -y
```

---

## 📝 Step 4: Run Linux Installation Script

### 4.1 - Make Script Executable

```bash
chmod +x setup_linux.sh
```

### 4.2 - Run Installation

```bash
./setup_linux.sh
```

**What this script does:**
1. ✅ Checks for Python 3.11
2. ✅ Creates virtual environment
3. ✅ Installs all Python dependencies
4. ✅ Creates log directories
5. ✅ Creates temp directories

**Expected output:**
```
=========================================
FileFinder Linux Installation Script
=========================================
Checking Python version...
✓ Python version: 3.11.x
Creating virtual environment...
✓ Virtual environment created
Upgrading pip...
✓ pip upgraded
Installing dependencies...
✓ Dependencies installed
✓ Log directory created: /var/log/filefinder
✓ Temp directory created: /tmp/filefinder
=========================================
Installation complete!
=========================================
```

### 4.3 - If Installation Fails

**Error: "python3.11: command not found"**
- Python 3.11 is not installed. Go back to Step 2.2

**Error: "Permission denied"**
- Run: `chmod +x setup_linux.sh` first

**Error: "pip install failed"**
- Check internet connection
- Try: `sudo apt install python3.11-pip -y`

---

## ⚙️ Step 5: Configure Database Connection

### 5.1 - Edit .env File

```bash
nano .env
```

### 5.2 - Update MySQL Connection Settings

Find these lines and update with **YOUR Windows server IP**:

**BEFORE:**
```bash
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_DATABASE=rec_files
MYSQL_USERNAME=arungt
MYSQL_PASSWORD=fi!ef!ndgt!23
```

**AFTER (replace with your Windows IP):**
```bash
MYSQL_HOST=192.168.0.104         ← CHANGE THIS to your Windows IP!
MYSQL_PORT=3306
MYSQL_DATABASE=(database name)
MYSQL_USERNAME=(sql username)
MYSQL_PASSWORD=(password)
```

### 5.3 - Optional: Customize Scan Settings

While in the `.env` file, you can adjust:

```bash
# Scan only files modified in last N days (0 = all files)
N_DAYS=365

# File extensions to scan (comma-separated, no spaces)
D_FILE_DETAILS_FILE_EXTENSIONS=.xlsx,.xls,.pdf,.doc,.docx,.txt

# Enable file extension counting (creates total_n_xls, total_n_xlsx, etc.)
ENABLE_FILE_EXT_COUNT_IN_SCAN=true

# Enable Excel content scanning (reads first few rows)
ENABLE_EXCEL_FILE_DATA_SCAN=false

# Sensitive data keywords to detect
FILE_PATH_SCAN_SENSITIVE_PATTERNS=password,creditcard,ssn,confidential

# Batch size for database inserts (higher = faster, more memory)
BATCH_INSERT_SIZE=1000
```

### 5.4 - Save and Exit

1. Press `Ctrl + X` to exit
2. Press `Y` to save
3. Press `Enter` to confirm filename

---

# Part 3: Testing Connectivity

Before running scans, let's test that Linux can connect to Windows MySQL.

---

## 🌐 Step 1: Test Network Connectivity

### 1.1 - Get Your Linux IP Address

```bash
hostname -I
```

Note the first IP address (e.g., `192.168.0.105`)

### 1.2 - Ping Windows Server

```bash
ping -c 4 192.168.0.104
```
(Replace with your Windows IP)

**Expected output:**
```
64 bytes from 192.168.0.104: icmp_seq=1 ttl=128 time=1.23 ms
64 bytes from 192.168.0.104: icmp_seq=2 ttl=128 time=0.98 ms
...
4 packets transmitted, 4 received, 0% packet loss
```

**✅ PASS:** If you see responses  
**❌ FAIL:** If you see "Destination Host Unreachable" → See [Troubleshooting](#troubleshooting-ping-fails)

---

## 🔌 Step 2: Test MySQL Port

### 2.1 - Test Port 3306 is Open

```bash
nc -zv 192.168.0.104 3306
```

**Expected output:**
```
Connection to 192.168.0.104 3306 port [tcp/mysql] succeeded!
```

**Alternative test using nmap:**
```bash
nmap -p 3306 192.168.0.104
```

Should show:
```
PORT     STATE SERVICE
3306/tcp open  mysql
```

**✅ PASS:** Port is open  
**❌ FAIL:** "Connection refused" → See [Troubleshooting](#troubleshooting-port-3306-refused)

---

## 🗄️ Step 3: Test MySQL Connection

### 3.1 - Test MySQL Login

```bash
mysql -h 192.168.0.104 -u arungt -p
```

Enter password: `fi!ef!ndgt!23`

**Expected output:**
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 123
Server version: 8.0.x MySQL Community Server

Type 'help;' or '\h' for help.

mysql>
```

**✅ PASS:** You see `mysql>` prompt  
**❌ FAIL:** See error messages below

### 3.2 - Test Database Access

If you successfully connected, type:

```sql
SHOW DATABASES;
```

Should show:
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| rec_files          |
+--------------------+
```

Type:
```sql
USE rec_files;
SHOW TABLES;
```

Should show 9 tables. Type `exit;` to quit.

---

## 🚨 Troubleshooting Connection Issues

### Troubleshooting: Error 2013 - Lost Connection at Initial Packet

**Problem:** `ERROR 2013 (HY000): Lost connection to MySQL server at 'reading initial communication packet', system error: 104`

**This means:** MySQL server is rejecting the connection before authentication even starts. This is usually a bind-address or skip-networking issue.

**Solution 1: Check MySQL bind-address Configuration**

On Windows, edit the MySQL configuration file:

1. Open Notepad as Administrator
2. Open: `C:\ProgramData\MySQL\MySQL Server 8.0\my.ini`
3. Find the `[mysqld]` section
4. Look for these lines and ensure they are set correctly:

```ini
[mysqld]
# MUST be 0.0.0.0 for remote access
bind-address = 0.0.0.0

# Make sure skip-networking is NOT present or is commented out
# skip-networking
```

5. **If you see `skip-networking` uncommented, add `#` in front:**
   ```ini
   # skip-networking
   ```

6. **If `bind-address` is missing, add it under `[mysqld]`:**
   ```ini
   bind-address = 0.0.0.0
   ```

7. Save and close the file

**Solution 2: Restart MySQL Service**

After editing `my.ini`:

```cmd
net stop MySQL80
net start MySQL80
```

Or via Services:
1. Press `Win + R`, type `services.msc`
2. Find **MySQL80** → Right-click → **Restart**

**Solution 3: Verify MySQL is Listening on All Interfaces**

On Windows CMD:
```cmd
netstat -an | findstr :3306
```

**Expected output:**
```
TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING
```

**❌ WRONG (will cause Error 2013):**
```
TCP    127.0.0.1:3306         0.0.0.0:0              LISTENING
```

If you see `127.0.0.1:3306`, MySQL is only listening locally. Go back to Solution 1.

**Solution 4: Check MySQL Error Log**

On Windows, check the error log for details:

```cmd
type "C:\ProgramData\MySQL\MySQL Server 8.0\Data\*.err"
```

Look for messages about:
- `bind-address`
- `skip-networking`
- Connection errors from your Linux IP

**Solution 5: Test from Windows MySQL Client**

On Windows, verify MySQL is working locally:

```cmd
mysql -u arungt -pfi!ef!ndgt!23 -e "SELECT 'MySQL is working' as status;"
```

Should return:
```
+------------------+
| status           |
+------------------+
| MySQL is working |
+------------------+
```

**Solution 6: Check Windows Hosts File**

Sometimes the hosts file can interfere:

1. Open: `C:\Windows\System32\drivers\etc\hosts`
2. Ensure there's no entry blocking your Linux IP
3. Should only have standard entries like `127.0.0.1 localhost`

**Solution 7: Verify Firewall Rule is Active**

```powershell
Get-NetFirewallRule -DisplayName "MySQL Server" | Format-Table DisplayName, Enabled, Direction, Action
```

Should show:
```
DisplayName  Enabled Direction Action
-----------  ------- --------- ------
MySQL Server    True   Inbound  Allow
```

If `Enabled` is `False`:
```powershell
Set-NetFirewallRule -DisplayName "MySQL Server" -Enabled True
```

**Solution 8: Try Telnet Test from Linux**

From Linux:
```bash
telnet 192.168.0.104 3306
```

If connection succeeds, you'll see garbled characters (MySQL handshake) - this is GOOD!
Press `Ctrl + ]` then type `quit` to exit.

If you get "Connection refused" or timeout, the firewall is still blocking.

**Solution 9: Temporarily Disable Windows Firewall (Testing Only)**

**⚠️ Warning: Only for testing! Re-enable after.**

On Windows PowerShell (Admin):
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

Test connection from Linux. If it works, the issue is firewall-related.

**Re-enable firewall:**
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
```

Then properly configure the MySQL firewall rule.

**Solution 10: Check MySQL User Host Permissions**

On Windows MySQL:
```cmd
mysql -u root -p
```

Run:
```sql
SELECT user, host FROM mysql.user WHERE user='arungt';
```

**Expected (allows remote access):**
```
+--------+------+
| user   | host |
+--------+------+
| arungt | %    |
+--------+------+
```

**❌ WRONG (will fail):**
```
+--------+-----------+
| user   | host      |
+--------+-----------+
| arungt | localhost |
+--------+-----------+
```

If you see `localhost`, recreate the user:
```sql
DROP USER IF EXISTS 'arungt'@'localhost';
CREATE USER 'arungt'@'%' IDENTIFIED BY 'fi!ef!ndgt!23';
GRANT ALL PRIVILEGES ON rec_files.* TO 'arungt'@'%';
FLUSH PRIVILEGES;
EXIT;
```

---

### Troubleshooting: Ping Fails

**Problem:** `ping: Destination Host Unreachable`

**Solutions:**

1. **Check Windows IP is correct:**
   - On Windows: Run `ipconfig` again
   - Verify you're using the right IP

2. **Check both machines are on same network:**
   - Linux and Windows should have IPs in same range (e.g., both 192.168.0.x)

3. **Check Windows firewall isn't blocking ICMP:**
   - On Windows PowerShell (Admin):
     ```powershell
     New-NetFirewallRule -DisplayName "Allow ICMPv4-In" -Protocol ICMPv4 -IcmpType 8 -Action Allow
     ```

4. **Try from Windows to Linux:**
   - On Windows CMD: `ping 192.168.0.105` (Linux IP)
   - If this fails too, network issue exists

---

### Troubleshooting: Port 3306 Refused

**Problem:** `Connection to 192.168.0.104 3306 port [tcp/mysql] failed: Connection refused`

**This means:** MySQL port is blocked by Windows Firewall

**Solution 1: Check Firewall Rule Exists**

On Windows PowerShell (Admin):
```powershell
Get-NetFirewallRule -DisplayName "MySQL Server"
```

If you see nothing, the rule doesn't exist. Create it:
```powershell
New-NetFirewallRule -DisplayName "MySQL Server" -Direction Inbound -Protocol TCP -LocalPort 3306 -Action Allow
```

**Solution 2: Verify MySQL is Listening**

On Windows CMD:
```cmd
netstat -an | findstr :3306
```

Should show:
```
TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING
```

If it shows `127.0.0.1:3306`, MySQL is only listening locally:
- Edit `my.ini` file (see Part 1, Step 6)
- Change `bind-address = 127.0.0.1` to `bind-address = 0.0.0.0`
- Restart MySQL service

**Solution 3: Check Antivirus**

Some antivirus software blocks MySQL:
- Temporarily disable antivirus
- Test connection again
- If it works, add MySQL to antivirus exceptions

---

### Troubleshooting: MySQL Authentication Errors

**Error 1: "Host 'rachit' is not allowed to connect to this MySQL server"**

**Problem:** MySQL user `arungt` is not configured for remote access

**Solution - On Windows CMD:**
```cmd
mysql -u root -p
```

Enter root password, then:
```sql
DROP USER IF EXISTS 'arungt'@'localhost';
CREATE USER 'arungt'@'%' IDENTIFIED BY 'fi!ef!ndgt!23';
GRANT ALL PRIVILEGES ON rec_files.* TO 'arungt'@'%';
FLUSH PRIVILEGES;
SELECT user, host FROM mysql.user WHERE user='arungt';
EXIT;
```

Should show:
```
+--------+------+
| user   | host |
+--------+------+
| arungt | %    |
+--------+------+
```

**Error 2: "Access denied for user 'arungt'@'...'**

**Problem:** Wrong password or user doesn't exist

**Solution 1: Check .env file:**
```bash
cat .env | grep MYSQL_PASSWORD
```

Should show: `MYSQL_PASSWORD=fi!ef!ndgt!23`

**Solution 2: Reset user on Windows:**
```sql
mysql -u root -p
DROP USER IF EXISTS 'arungt'@'localhost';
CREATE USER 'arungt'@'%' IDENTIFIED BY 'fi!ef!ndgt!23';
GRANT ALL PRIVILEGES ON rec_files.* TO 'arungt'@'%';
FLUSH PRIVILEGES;
```

---

# Part 4: Running Your First Scan

Once all connectivity tests pass, you're ready to scan!

---

## 🚀 Step 1: Activate Virtual Environment

```bash
cd ~/FileFinder_User_Linux
source venv/bin/activate
```

Your prompt will change to show `(venv)`:
```
(venv) user@linux:~/FileFinder_User_Linux$
```

---

## ▶️ Step 2: Run FileFinder Scanner

```bash
python file_info_version_22_linux.py
```

---

## 📊 Step 3: Follow Interactive Prompts

### Prompt 1: Enter Username
```
Enter your username: kali
```
Type your name or username and press Enter.

### Prompt 2: Select Scan Type

Use arrow keys ↑↓ to navigate, press Enter to select:

```
? Select scan type:
  File Count        ← Quick count only
❯ File Data Scan    ← Full detailed scan (RECOMMENDED)
```

Select: **File Data Scan**

### Prompt 3: Select Scan Option

```
? Select the type of scan:
  Full System Scan      ← Scans entire filesystem (SLOW - several hours)
❯ Specific Path Scan    ← Scan chosen directory (RECOMMENDED for testing)
```

Select: **Specific Path Scan**

### Prompt 4: Enter Path to Scan

**For first test, scan a small directory:**
```
Enter the path to scan (e.g., /home/username): /home/kali/Documents
```

**Other good test paths:**
- `/home/kali/Downloads`
- `/home/kali/Desktop`
- `/tmp` (quick test)

Press Enter.

---

## 📈 Step 4: Monitor Scan Progress

You'll see output like:

```
=========================================
FileFinder Linux v7.0
=========================================
System: Linux 6.x.x-kali
Machine_Name: kali
IP Address: 192.168.0.105
Database: rec_files
=========================================

[yellow]Scanning /home/kali/Documents...[/yellow]
[bright_green]Scan complete! Found 347 files[/bright_green]

[yellow]Collecting file metadata...[/yellow]
Metadata collection progress: 347/347 files

[yellow]Inserting 347 files into database...[/yellow]
Batch inserted 347/347 files

[bright_green]✓ Inserted 347 files successfully[/bright_green]

[bright_green]============================================================[/bright_green]
[bright_green]Scan completed in 23.45s (0.39 minutes)[/bright_green]
[bright_green]============================================================[/bright_green]
```

**✅ SUCCESS:** If you see "Scan completed" message  
**❌ ERROR:** If you see database connection errors, see [Troubleshooting](#part-4-troubleshooting)

---

## 🔍 Step 5: Check Logs

```bash
tail -50 /var/log/filefinder/*.log
```

Look for:
```
2025-11-28 14:30:15 | INFO     | Successfully connected to MySQL database: rec_files
2025-11-28 14:30:45 | SUCCESS  | Machine summary count inserted successfully
2025-11-28 14:30:45 | SUCCESS  | FileFinder Linux completed successfully
```

---

# Part 5: Viewing Results on Windows

Now let's check that the data was stored in Windows MySQL!

---

## 🔍 Step 1: Open MySQL on Windows

### Option A: MySQL Command Line

```cmd
mysql -u arungt -pfi!ef!ndgt!23 rec_files
```

(Note: No space between `-p` and password)

### Option B: MySQL Workbench (GUI)

1. Open MySQL Workbench
2. Create connection:
   - Connection Name: FileFinder
   - Hostname: localhost
   - Port: 3306
   - Username: arungt
   - Password: fi!ef!ndgt!23
3. Click **Test Connection**
4. Click **OK**
5. Double-click connection to open

---

## 📊 Step 2: View Scanned Data

### Query 1: Check Machine Summary

```sql
USE rec_files;

SELECT 
    Machine_Name,
    ip_address,
    total_n_files,
    CONCAT(ROUND(total_diskspace, 2), ' GB') as total_disk,
    CONCAT(ROUND(used_diskspace, 2), ' GB') as used_disk,
    CONCAT(ROUND(free_diskspace, 2), ' GB') as free_disk,
    row_creation_date_time
FROM f_machine_files_summary_count
ORDER BY row_creation_date_time DESC;
```

**Expected result:**
```
+---------------+----------------+---------------+------------+-----------+-----------+---------------------+
| Machine_Name  | ip_address     | total_n_files | total_disk | used_disk | free_disk | row_creation_date...
+---------------+----------------+---------------+------------+-----------+-----------+---------------------+
| kali          | 192.168.0.105  |           347 | 465.76 GB  | 234.12 GB | 231.64 GB | 2025-11-28 14:30:45 |
+---------------+----------------+---------------+------------+-----------+-----------+---------------------+
```

You should see:
- ✅ **Machine_Name:** Your Linux hostname (e.g., `kali`)
- ✅ **ip_address:** Your Linux IP
- ✅ **total_n_files:** Count of files scanned
- ✅ **Disk space:** Total, used, free in GB

---

### Query 2: View File Details

```sql
SELECT 
    Machine_Name,
    file_path,
    file_name,
    file_extension,
    ROUND(file_size_bytes/1024/1024, 2) as size_mb,
    file_owner,
    file_modification_time
FROM d_file_details
WHERE Machine_Name = 'kali'
ORDER BY row_creation_date_time DESC
LIMIT 10;
```

**Expected result:**
```
+--------------+----------------------------------+-----------------+----------------+---------+-------------+---------------------+
| Machine_Name | file_path                        | file_name       | file_extension | size_mb | file_owner  | file_modification...|
+--------------+----------------------------------+-----------------+----------------+---------+-------------+---------------------+
| kali         | /home/kali/Documents/report.pdf  | report.pdf      | .pdf           |    2.34 | kali:kali   | 2025-11-27 10:23:45 |
| kali         | /home/kali/Documents/data.xlsx   | data.xlsx       | .xlsx          |    0.45 | kali:kali   | 2025-11-26 15:30:12 |
...
+--------------+----------------------------------+-----------------+----------------+---------+-------------+---------------------+
```

You should see:
- ✅ Linux file paths (e.g., `/home/kali/...`)
- ✅ File sizes in megabytes
- ✅ Linux file ownership (e.g., `kali:kali`)

---

### Query 3: File Extension Summary

```sql
SELECT 
    file_extension,
    COUNT(*) as file_count,
    ROUND(SUM(file_size_bytes)/1024/1024/1024, 2) as total_size_gb
FROM d_file_details
WHERE Machine_Name = 'kali'
GROUP BY file_extension
ORDER BY file_count DESC
LIMIT 10;
```

**Expected result:**
```
+----------------+------------+---------------+
| file_extension | file_count | total_size_gb |
+----------------+------------+---------------+
| .pdf           |        234 |          1.23 |
| .docx          |        123 |          0.45 |
| .jpg           |         89 |          0.12 |
...
+----------------+------------+---------------+
```

---

### Query 4: Check Scan History (Audit Log)

```sql
SELECT 
    pc_ip_address,
    employee_username,
    start_time,
    end_time,
    duration,
    activity_status
FROM audit_info
ORDER BY start_time DESC
LIMIT 5;
```

---

### Query 5: Largest Files

```sql
SELECT 
    file_path,
    file_name,
    ROUND(file_size_bytes/1024/1024, 2) as size_mb
FROM d_file_details
WHERE Machine_Name = 'kali'
ORDER BY file_size_bytes DESC
LIMIT 20;
```

---

## 🎉 Success! You Now Have:

- ✅ Windows MySQL server storing all file data
- ✅ Linux scanner successfully sending data
- ✅ Complete file inventory in database
- ✅ Disk space tracking
- ✅ File extension counts
- ✅ Audit trail of all scans

---

# Part 6: Running Regular Scans

## 🔄 Daily Usage Workflow

### Quick Scan (Same Directory)

```bash
cd ~/FileFinder_User_Linux
source venv/bin/activate
python file_info_version_22_linux.py

# Select: File Data Scan → Specific Path Scan
# Enter path: /home/kali/Documents
```

### Full System Scan (Comprehensive)

**⚠️ Warning:** This will take several hours!

```bash
cd ~/FileFinder_User_Linux
source venv/bin/activate
python file_info_version_22_linux.py

# Select: File Data Scan → Full System Scan
```

**What gets scanned:**
- `/` - Root filesystem
- `/home` - All user directories
- `/opt` - Optional software
- `/usr` - Unix system resources
- `/var` - Variable data

**What's automatically skipped:**
- `/proc` - Process information (virtual)
- `/sys` - System kernel data (virtual)
- `/dev` - Device files
- `/run` - Runtime data
- `/snap` - Snap packages

---

## 📅 Schedule Automatic Scans (Optional)

### Using Systemd Timer (Recommended)

**Step 1: Edit Service File**
```bash
nano filefinder.service
```

Update these lines with your paths:
```ini
[Service]
User=kali
WorkingDirectory=/home/kali/FileFinder_User_Linux
ExecStart=/home/kali/FileFinder_User_Linux/venv/bin/python3 /home/kali/FileFinder_User_Linux/file_info_version_22_linux.py
```

Save and exit.

**Step 2: Install Service**
```bash
sudo cp filefinder.service /etc/systemd/system/
sudo cp filefinder.timer /etc/systemd/system/
sudo systemctl daemon-reload
```

**Step 3: Enable Daily Scans**
```bash
sudo systemctl enable filefinder.timer
sudo systemctl start filefinder.timer
```

**Step 4: Check Status**
```bash
sudo systemctl status filefinder.timer
systemctl list-timers filefinder.timer
```

**See `SYSTEMD_SETUP.md` for advanced scheduling options.**

---

## 🖥️ Monitor Active Scan (Real-time)

While a scan is running, open a second terminal:

```bash
# Watch logs in real-time
tail -f /var/log/filefinder/*.log

# Monitor database growth
watch -n 10 'mysql -h 192.168.0.104 -u arungt -pfi\!ef\!ndgt\!23 rec_files -e "SELECT COUNT(*) as total FROM d_file_details WHERE Machine_Name=\"kali\";"'
```

---

## 📊 Generate Reports on Windows

### Report 1: Files by Machine

```sql
SELECT 
    Machine_Name,
    COUNT(*) as total_files,
    ROUND(SUM(file_size_bytes)/1024/1024/1024, 2) as total_size_gb,
    MAX(file_modification_time) as latest_file_date
FROM d_file_details
GROUP BY Machine_Name
ORDER BY total_files DESC;
```

### Report 2: Recently Modified Files (Last 7 Days)

```sql
SELECT 
    Machine_Name,
    file_path,
    file_name,
    file_modification_time
FROM d_file_details
WHERE file_modification_time >= DATE_SUB(NOW(), INTERVAL 7 DAY)
ORDER BY file_modification_time DESC
LIMIT 50;
```

### Report 3: Disk Space by Machine

```sql
SELECT 
    Machine_Name,
    ip_address,
    CONCAT(ROUND(total_diskspace, 2), ' GB') as total,
    CONCAT(ROUND(used_diskspace, 2), ' GB') as used,
    CONCAT(ROUND(free_diskspace, 2), ' GB') as free,
    CONCAT(ROUND((used_diskspace/total_diskspace)*100, 1), '%') as usage_percent
FROM f_machine_files_summary_count
ORDER BY Machine_Name;
```

### Report 4: Sensitive Files Detected

```sql
SELECT 
    Machine_Name,
    file_path,
    file_name,
    file_size_bytes
FROM d_file_details
WHERE file_is_sensitive_data = '1'
ORDER BY Machine_Name, file_path;
```

---

# Common Issues & Solutions

## 🔧 Issue 1: "Can't connect to MySQL server"

**Symptoms:**
```
ERROR 2003 (HY000): Can't connect to MySQL server on '192.168.0.104:3306' (111)
```

**Checklist:**
1. ✅ Windows MySQL service is running
2. ✅ Port 3306 is open in Windows Firewall
3. ✅ MySQL configured with `bind-address = 0.0.0.0`
4. ✅ `.env` file has correct Windows IP
5. ✅ Network connectivity (ping works)

**Solutions:**
- Restart MySQL service on Windows
- Verify firewall rule: `Get-NetFirewallRule -DisplayName "MySQL Server"`
- Check `netstat -an | findstr :3306` shows `0.0.0.0:3306`

---

## 🔧 Issue 2: "Permission denied" when scanning

**Symptoms:**
```
PermissionError: [Errno 13] Permission denied: '/root/file.txt'
```

**Solution 1: Run with sudo (for system-wide scans)**
```bash
sudo -E env PATH=$PATH python file_info_version_22_linux.py
```

**Solution 2: Scan only user-accessible directories**
```
Enter path: /home/kali
```

---

## 🔧 Issue 3: "Log directory not writable"

**Symptoms:**
```
PermissionError: [Errno 13] Permission denied: '/var/log/filefinder/'
```

**Solution:**
```bash
sudo mkdir -p /var/log/filefinder
sudo chown $(whoami):$(whoami) /var/log/filefinder
sudo chmod 755 /var/log/filefinder
```

---

## 🔧 Issue 4: Virtual environment activation failed

**Symptoms:**
```
bash: venv/bin/activate: No such file or directory
```

**Solution:**
```bash
# Recreate virtual environment
python3.11 -m venv venv

# Activate it
source venv/bin/activate

# Reinstall dependencies
pip install -r requirements-linux.txt
```

---

## 🔧 Issue 5: "Module not found" errors

**Symptoms:**
```
ModuleNotFoundError: No module named 'mysql.connector'
```

**Solution:**
```bash
# Make sure virtual environment is activated
source venv/bin/activate

# Reinstall dependencies
pip install --upgrade pip
pip install -r requirements-linux.txt
```

---

## 🔧 Issue 6: Scan is very slow

**Causes & Solutions:**

1. **File extension counting enabled:**
   - Edit `.env`: `ENABLE_FILE_EXT_COUNT_IN_SCAN=false`

2. **Excel scanning enabled:**
   - Edit `.env`: `ENABLE_EXCEL_FILE_DATA_SCAN=false`

3. **Scanning system directories:**
   - Avoid scanning `/` on first run
   - Start with specific directories: `/home/user`

4. **Network latency:**
   - Check network speed between Linux and Windows
   - Consider increasing batch size: `BATCH_INSERT_SIZE=2000`

---

## 🔧 Issue 7: No data appearing in database

**Checklist:**

1. **Check scan completed successfully:**
   ```bash
   tail -100 /var/log/filefinder/*.log
   ```
   Look for "SUCCESS" messages

2. **Verify database name:**
   ```bash
   cat .env | grep MYSQL_DATABASE
   ```
   Should be: `MYSQL_DATABASE=rec_files`

3. **Check Windows MySQL logs:**
   ```cmd
   # On Windows
   type "C:\ProgramData\MySQL\MySQL Server 8.0\Data\*.err"
   ```

4. **Test query on Windows:**
   ```sql
   SELECT COUNT(*) FROM d_file_details WHERE Machine_Name='kali';
   ```

---

# 📚 Additional Resources

## Documentation Files

- **README_USER.md** - Linux client detailed guide
- **README_SERVER.md** - Windows server detailed guide
- **DEPLOYMENT_ARCHITECTURE.md** - System architecture overview
- **SCHEMA_UPDATE_SUMMARY.md** - Database schema details
- **SYSTEMD_SETUP.md** - Automated scheduling guide
- **WINDOWS_FIREWALL_SETUP.md** - Advanced firewall configuration

## File Locations

**Windows:**
```
C:\FileFinder4\
├── SQLScripts\
│   └── COMPLETE_DB_EXPORT.sql
├── setup_server.bat
└── README_SERVER.md
```

**Linux:**
```
~/FileFinder_User_Linux/
├── file_info_version_22_linux.py
├── requirements-linux.txt
├── .env
├── setup_linux.sh
└── README_USER.md
```

## MySQL Workbench (Free GUI Tool)

Download: https://dev.mysql.com/downloads/workbench/

**Benefits:**
- Visual query builder
- Database diagram view
- Easy data export to Excel
- Syntax highlighting
- Query history

---

# 🎓 Understanding the Architecture

```
┌─────────────────────────────────────────────────────────┐
│           WINDOWS MySQL SERVER (192.168.0.104)          │
│                                                         │
│  ┌───────────────────────────────────────────────────┐ │
│  │   MySQL Database: rec_files                       │ │
│  │                                                   │ │
│  │   Tables:                                         │ │
│  │   • f_machine_files_summary_count (1 row/machine)│ │
│  │   • d_file_details (millions of file records)    │ │
│  │   • audit_info (scan history)                    │ │
│  │   • xls_file_sheet (Excel metadata)              │ │
│  │   • env_info (configuration)                     │ │
│  └───────────────────────────────────────────────────┘ │
│                         ▲                               │
└─────────────────────────┼───────────────────────────────┘
                          │
                          │ MySQL Protocol
                          │ Port 3306
                          │ TCP/IP
         ┌────────────────┼────────────────┐
         │                │                │
         │                │                │
┌────────▼────────┐ ┌─────▼──────┐ ┌──────▼─────┐
│  Linux Client 1 │ │Linux Client│ │Linux Client│
│  (Kali)         │ │ (Ubuntu)   │ │ (Debian)   │
│                 │ │            │ │            │
│ FileFinder_     │ │FileFinder_ │ │FileFinder_ │
│ User_Linux/     │ │User_Linux/ │ │User_Linux/ │
│                 │ │            │ │            │
│ • Scans /home   │ │• Scans /var│ │• Scans /opt│
│ • Sends to DB   │ │• Sends data│ │• Sends data│
│ • Python 3.11   │ │• Python3.11│ │• Python3.11│
└─────────────────┘ └────────────┘ └────────────┘
```

**Data Flow:**
1. Linux scanner reads local filesystem
2. Collects file metadata (path, size, owner, dates)
3. Calculates disk space usage
4. Counts files by extension
5. Sends INSERT queries to Windows MySQL
6. Windows MySQL stores data permanently
7. Admin queries data from Windows (any time)

---

# 🚀 Next Steps

## For Production Use:

1. **Deploy to Multiple Linux Machines**
   - Copy `FileFinder_User_Linux` to each machine
   - Configure same Windows MySQL IP in `.env`
   - All data centralizes in Windows database

2. **Schedule Regular Scans**
   - Use systemd timers for daily/weekly scans
   - Monitor logs for failures
   - Set up email alerts (optional)

3. **Backup Database**
   - Regular MySQL backups on Windows
   - Test restore procedures
   ```cmd
   mysqldump -u arungt -p rec_files > backup.sql
   ```

4. **Optimize Performance**
   - Increase batch sizes for large scans
   - Disable extension counting if not needed
   - Add more MySQL indexes for custom queries

5. **Security Hardening**
   - Use VPN for scanning over internet
   - Restrict MySQL user to specific IPs
   - Encrypt database connections (SSL)
   - Regular password rotation

---

# ✅ Quick Reference Card

## Windows Server Quick Commands

```cmd
REM Check MySQL is running
sc query MySQL80

REM Test local connection
mysql -u arungt -pfi!ef!ndgt!23 rec_files

REM View recent scans
mysql -u arungt -pfi!ef!ndgt!23 rec_files -e "SELECT Machine_Name, ip_address, total_n_files, row_creation_date_time FROM f_machine_files_summary_count ORDER BY row_creation_date_time DESC LIMIT 5;"

REM Backup database
mysqldump -u arungt -pfi!ef!ndgt!23 rec_files > C:\backup_%DATE%.sql
```

## Linux Client Quick Commands

```bash
# Activate environment
cd ~/FileFinder_User_Linux && source venv/bin/activate

# Run scan
python file_info_version_22_linux.py

# Check logs
tail -50 /var/log/filefinder/*.log

# Test connection
mysql -h 192.168.0.104 -u arungt -pfi\!ef\!ndgt\!23 -e "SHOW DATABASES;"

# Test port
nc -zv 192.168.0.104 3306
```

---

# 📞 Support & Contact

**Email:** arunkg99@gmail.com

**Documentation:** Check all README files in each folder

**Common Questions:**
- Check logs first: `/var/log/filefinder/*.log` (Linux)
- Verify connectivity: `ping` then `nc -zv` then `mysql` test
- Check Windows firewall if connection fails
- Ensure Python 3.11 exactly (not 3.10 or 3.12)

---

# 🎉 Congratulations!

You now have a fully functional FileFinder deployment!

**You've successfully:**
- ✅ Set up Windows MySQL server from scratch
- ✅ Configured network and firewall for remote access
- ✅ Installed Linux scanner with Python 3.11
- ✅ Tested all connectivity (ping, port, MySQL)
- ✅ Ran your first file scan
- ✅ Viewed results in Windows database
- ✅ Understand the complete architecture

**Your system can now:**
- Track millions of files across multiple machines
- Monitor disk space usage in real-time
- Detect sensitive data patterns
- Generate comprehensive reports
- Scale to unlimited Linux clients

**Happy scanning! 🔍✨**

---

**Document Version:** 1.0  
**Last Updated:** November 28, 2025  
**FileFinder Version:** 7.0 (Complete Schema)
