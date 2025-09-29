# Source Code Disclosure via Backup Files

**Difficulty:** `APPRENTICE` | **Category:** `Information Disclosure`

---

## 🎯 Lab Goal

Find the hard-coded database password inside leaked source code backup files and submit it.

---

## 📋 Lab Description

This lab leaks its source code via backup files stored in a hidden directory.

---

## 🔎 Reconnaissance

### Initial Enumeration

1. **Checked page source** → Nothing useful found
2. **Checked `/robots.txt`** → Discovered interesting entry

### robots.txt Analysis

Found the following configuration:

```
User-agent: *
Disallow: /backup
```

The `/backup` directory was being blocked from search engines but was still publicly accessible.

---

## 🧪 Testing & Exploitation

### Accessing the Hidden Directory

Navigated to the disclosed path:

```
https://[lab-id].web-security-academy.net/backup
```

### File Discovery

Found an exposed backup file:

```
ProductTemplate.java.bak
```

### Source Code Analysis

Opened the backup file and examined the code. Inside the `readObject` method, found hard-coded database credentials:

```java
ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
    "org.postgresql.Driver",      // Driver
    "postgresql",                  // Database type
    "localhost",                   // Host
    5432,                          // Port
    "postgres",                    // Database name
    "postgres",                    // Username
    "zxaoig986rm5v15t4s5232xsjnd1mylv"  // Password ⚠️
).withAutoCommit();
```

The last parameter contains the hard-coded database password.

---

## 🚩 Solution

**Answer:** `zxaoig986rm5v15t4s5232xsjnd1mylv`

---

## 💡 Key Takeaways

- **Backup files** (`.bak`, `.old`, `.tmp`, `~`) should never be publicly accessible
- **robots.txt** is not a security mechanism:
  - It's meant for search engine crawlers
  - Attackers can read it to find "hidden" directories
  - It provides a roadmap of sensitive areas
- **Hard-coded credentials** in source code are a critical security risk:
  - Can be exposed through various leaks
  - Difficult to rotate without code changes
  - Often reused across environments
- Common backup file extensions to watch for:
  - `.bak`, `.backup`, `.old`, `.orig`
  - `~` suffix (editor backups)
  - `.swp`, `.swo` (vim swap files)

---

## 🛡️ Remediation

- **Remove backup files** from production servers
- Configure web server to block access to backup file extensions
- Use **environment variables** or **secret management systems** for credentials
- Implement proper `.gitignore` rules to prevent committing backups
- Regular security scans for exposed sensitive files
- Use proper **access controls** on directories
- Never rely on robots.txt for security

### Example: Blocking Backup Files (Apache)

```apache
<FilesMatch "\.(bak|backup|old|orig|swp|swo|tmp)$">
    Require all denied
</FilesMatch>
```

### Example: Blocking Backup Files (Nginx)

```nginx
location ~* \.(bak|backup|old|orig|swp|swo|tmp)$ {
    deny all;
}
```

---

**Status:** ✅ Solved
