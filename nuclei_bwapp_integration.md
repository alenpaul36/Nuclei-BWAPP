# Vulnerability Testing with Nuclei on BWAPP

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Setting Up BWAPP](#setting-up-bwapp)
- [Installing Nuclei](#installing-nuclei)
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

- Docker (for running BWAPP)
- Go 1.19+ (for Nuclei installation)
- Basic understanding of web vulnerabilities
- Terminal/Command Line interface familiarity
- Git

## Setting Up BWAPP

BWAPP can be set up using Docker for convenience:

```bash
# Pull the BWAPP Docker image
docker pull raesene/bwapp

# Run the BWAPP container
docker run -d -p 80:80 raesene/bwapp

# The application will be available at http://localhost/
```

After deployment, complete the initial setup:

1. Navigate to http://localhost/
2. Click on "Install" to set up the database
3. Log in with default credentials (bee:bug)
4. You now have access to a vulnerable application with various security issues

## Configuring Nuclei for BWAPP

To effectively scan BWAPP, you'll need to:

1. Update Nuclei templates:
```bash
nuclei -ut
```

2. Create a target file for BWAPP:
```bash
echo "http://localhost/" > bwapp_target.txt
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

## Understanding Scan Results

Nuclei produces results in various formats. By default, the output includes:

- Vulnerability name
- Severity level
- Affected URL
- Details about the vulnerability
- References for further information

Example output:
```
[2023-03-26 18:30:45] [sql-injection] [high] SQL Injection detected at http://localhost/sqli.php
[2023-03-26 18:30:47] [xss-reflected] [medium] Reflected XSS detected at http://localhost/xss-reflected.php
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

## Troubleshooting

Common issues and solutions:

| Issue | Solution |
|-------|----------|
| BWAPP container not starting | Check Docker service status and port conflicts |
| Templates not found | Run `nuclei -ut` to update templates |
| Slow scan performance | Use concurrent flag: `-c 10` to increase speed |
| False negatives | Try different parameter variations or create custom templates |
| Connection refused | Ensure BWAPP is running and accessible |

## References

- [Nuclei GitHub Repository](https://github.com/projectdiscovery/nuclei)
- [Nuclei Documentation](https://nuclei.projectdiscovery.io/)
- [BWAPP Official Website](http://www.itsecgames.com/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Common Vulnerability Scoring System](https://www.first.org/cvss/)
