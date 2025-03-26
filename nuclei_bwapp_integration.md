# Vulnerability Testing with Nuclei on BWAPP

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Setting Up BWAPP](#setting-up-bwapp)
- [Configuring Nuclei for BWAPP](#configuring-nuclei-for-bwapp)
- [Running Vulnerability Scans](#running-vulnerability-scans)
- [Understanding Scan Results](#understanding-scan-results)
- [Creating Custom Templates](#creating-custom-templates)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [References](#references)

## Introduction

This guide demonstrates how to use [Nuclei](https://github.com/projectdiscovery/nuclei), a powerful vulnerability scanner, to test for security issues in [BWAPP](http://www.itsecgames.com/) (Buggy Web Application Project). BWAPP is a deliberately vulnerable web application used for security training, and Nuclei is a fast, template-based vulnerability scanner that allows for easy customization and extension.

By integrating these tools, security professionals can:
- Practice vulnerability detection in a safe environment
- Learn how to create custom vulnerability templates
- Understand common web application security flaws
- Test detection methodologies before applying them to production environments

## Prerequisites

Before starting, ensure you have the following:

- Kali Linux (which comes with Nuclei pre-installed)
- LAMP/XAMPP stack (for BWAPP installation)
- Basic understanding of web vulnerabilities
- Terminal/Command Line interface familiarity
- Git

## Setting Up BWAPP

BWAPP requires a proper setup environment to run correctly. The following guide is based on [ahmedhamdy0x's BWAPP Installation Guide](https://github.com/ahmedhamdy0x/bwapp-Installation).

### Installing BWAPP on Kali Linux & Ubuntu

#### 1. Download BWAPP
1. Go to [bWAPP download page](http://www.itsecgames.com/download.htm)
2. Click on "You can download bWAPP from here" to go to the download server
3. Click on "Download Latest Version"

#### 2. Prepare the Environment
```bash
# Update system
sudo apt-get update -y

# Create and prepare directories
cd ~/Downloads
mkdir bwapp
mv bWAPPv2.2.zip bwapp
cd bwapp

# Extract BWAPP
unzip bWAPPv2.2.zip
rm bWAPPv2.2.zip
```

#### 3. Install and Configure MySQL
```bash
# Install MySQL if not already installed
sudo apt install mysql-server

# Start and enable MySQL
sudo systemctl start mysql
sudo systemctl enable mysql

# Configure MySQL
sudo mysql
CREATE USER 'bwapp'@'localhost' IDENTIFIED BY 'bug';
GRANT ALL PRIVILEGES ON bWAPP.* TO 'bwapp'@'localhost';
FLUSH PRIVILEGES;
exit
```

#### 4. Install and Configure Apache2
```bash
# Install Apache2 if not already installed
sudo apt install apache2

# Start and enable Apache2
sudo systemctl start apache2
sudo systemctl enable apache2
```

#### 5. Prepare BWAPP Files
```bash
# Navigate to BWAPP directory
cd bWAPP

# Set correct permissions
chmod 777 passwords/
chmod 777 images/
chmod 777 documents/

# Edit database settings
mousepad admin/settings.php
# Change database credentials:
# $db_username = "bwapp";
# $db_password = "bug";
```

#### 6. Critical Step: Fix install.php File
This step is crucial for successful installation:

1. Go to [this link](https://drive.google.com/file/d/1LaABJUfjHgMZgYTfZvIltbvDfX5wYFJJ/view) and copy the content
2. Edit install.php:
   ```bash
   mousepad install.php
   ```
3. Select all (Ctrl+A), paste the copied content (Ctrl+V), and save (Ctrl+S)

#### 7. Move BWAPP to Web Server and Install
```bash
# Move BWAPP files to web server directory
cd ../
sudo mv * /var/www/html/

# Open browser and install the database
# Navigate to http://localhost/bWAPP/install.php and click "Install"
```

#### 8. Verify and Login
1. Verify database installation:
   ```bash
   sudo mysql
   SHOW DATABASES;
   exit
   ```
2. Login to BWAPP at http://localhost/bWAPP/ with:
   - Username: `bee`
   - Password: `bug`

## Configuring Nuclei for BWAPP

Nuclei comes pre-installed in Kali Linux. If you're using a different distribution, refer to the official [Nuclei installation guide](https://github.com/projectdiscovery/nuclei).

To effectively scan BWAPP, you'll need to:

1. **Update Nuclei templates**:
   ```bash
   nuclei -ut
   ```

2. **Create a target file for BWAPP**:
   ```bash
   echo "http://localhost/bWAPP/" > bwapp_target.txt
   ```

3. **Configure session handling** (if needed):
   - Create a session file for authenticated scanning:
     ```bash
     # First log in to BWAPP using a browser and capture the cookie
     # Create a cookies.txt file with the captured session cookie
     echo "http://localhost/bWAPP/ cookie_name=cookie_value" > cookies.txt
     ```

## Running Vulnerability Scans

### Basic Scan
Run a basic scan against BWAPP:

```bash
nuclei -l bwapp_target.txt -o bwapp_scan_results.txt
```

### Targeted Vulnerability Scanning
To scan for specific vulnerabilities like SQL injection that are known to exist in BWAPP:

```bash
nuclei -l bwapp_target.txt -t nuclei-templates/vulnerabilities/sql-injection/ -o sql_injection_results.txt
```

### Common Web Vulnerabilities
Scan for common web vulnerabilities that BWAPP intentionally contains:

```bash
nuclei -l bwapp_target.txt -t nuclei-templates/vulnerabilities/common/ -o common_vulns_results.txt
```

### Authenticated Scanning
If you need to scan authenticated pages:

```bash
nuclei -l bwapp_target.txt -H "Cookie: PHPSESSID=your_session_id" -o authenticated_scan_results.txt
```

### Advanced Scan Options
For more comprehensive scanning:

```bash
nuclei -l bwapp_target.txt -severity critical,high,medium -c 10 -rate-limit 100 -screenshot -o detailed_scan.txt
```

## Understanding Scan Results

Nuclei produces results in various formats. By default, the output includes:

- Vulnerability name
- Severity level
- Affected URL
- Details about the vulnerability
- References for further information

Example output:
```
[2023-03-26 18:30:45] [sql-injection] [high] SQL Injection detected at http://localhost/bWAPP/sqli.php
[2023-03-26 18:30:47] [xss-reflected] [medium] Reflected XSS detected at http://localhost/bWAPP/xss-reflected.php
```

## Creating Custom Templates

To create custom templates specifically for BWAPP vulnerabilities:

1. Create a new YAML file:
```bash
touch bwapp-template.yaml
```

2. Define a template for a specific BWAPP vulnerability:

```yaml
id: bwapp-sql-injection-login
info:
  name: BWAPP SQL Injection in Login Form
  author: YourName
  severity: high
  description: Tests for SQL injection in BWAPP login form
  tags: bwapp,sqli,injection

requests:
  - method: POST
    path:
      - "{{BaseURL}}/login.php"
    body: "login=admin%27%20or%20%271%27%3D%271&password=password&form=submit"
    matchers:
      - type: word
        words:
          - "Welcome to the password protected area"
        part: body
```

3. Run the custom template:
```bash
nuclei -l bwapp_target.txt -t bwapp-template.yaml
```

## Best Practices

When using Nuclei with BWAPP:

1. **Start with targeted scans**: Begin with specific vulnerability categories before running full scans
2. **Use rate limiting**: Even though it's a local application, practice good habits with `-rate-limit` flag
3. **Categorize findings**: Use tags and severity filters to organize results
4. **Review false positives**: BWAPP may trigger certain detections that require manual verification
5. **Create custom templates**: Practice writing templates for vulnerabilities specific to BWAPP
6. **Document findings**: Create comprehensive reports of vulnerabilities found
7. **Test remediation**: Practice fixing vulnerabilities and verify with follow-up scans

## Troubleshooting

Common issues and solutions:

| Issue | Solution |
|-------|----------|
| BWAPP not accessible | Check Apache/MySQL services status with `sudo systemctl status apache2` and `sudo systemctl status mysql` |
| Docker container not starting | Check Docker service status with `sudo systemctl status docker` and port conflicts with `sudo netstat -tuln` |
| Templates not found | Run `nuclei -ut` to update templates |
| Slow scan performance | Use concurrent flag: `-c 10` to increase speed |
| False negatives | Try different parameter variations or create custom templates |
| Connection refused | Ensure BWAPP is running and accessible |
| Authentication issues | Verify session cookies or try re-logging in |

### Specific Database Connection Errors

#### "Connection failed: Unknown database 'bWAPP'"
1. **Check database creation**:
   ```bash
   # For Docker
   sudo docker exec -it bwapp-container bash
   mysql -u root
   SHOW DATABASES;
   ```
   
2. **Case sensitivity issue**:
   - MySQL on Linux is case-sensitive for database names
   - Try creating both `bWAPP` and `bwapp` databases
   - Edit the settings.php file to match the correct case

3. **Permissions issue**:
   ```bash
   # Verify user permissions
   mysql -u root
   SELECT user,host FROM mysql.user;
   SHOW GRANTS FOR 'bee'@'localhost';
   ```

4. **Manual database setup**:
   ```bash
   # Import the SQL schema manually
   mysql -u root
   CREATE DATABASE bWAPP;
   USE bWAPP;
   SOURCE /var/www/html/bWAPP/bWAPP.sql;
   ```

5. **Reset database settings**:
   - Ensure settings.php has correct credentials
   - Try both "bee" and "bwapp" as the username
   - Verify password is "bug"

## References

- [Nuclei GitHub Repository](https://github.com/projectdiscovery/nuclei)
- [Nuclei Documentation](https://nuclei.projectdiscovery.io/)
- [BWAPP Official Website](http://www.itsecgames.com/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Common Vulnerability Scoring System](https://www.first.org/cvss/)
