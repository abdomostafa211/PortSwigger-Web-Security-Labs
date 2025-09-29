# Information Disclosure on Debug Page

**Difficulty:** `APPRENTICE` | **Category:** `Information Disclosure`

---

## 🎯 Lab Goal

Find and submit the `SECRET_KEY` environment variable exposed through a debug page.

---

## 📋 Lab Description

This lab contains a debug page that discloses sensitive information about the application.

---

## 🔎 Reconnaissance

Started by examining the page source for any hidden clues.

### Source Code Analysis

Found an interesting HTML comment:

```html
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

This revealed a debug endpoint that was commented out but still accessible.

---

## 🧪 Testing & Exploitation

### Accessing the Debug Page

Navigated directly to the discovered endpoint:

```
https://[lab-id].web-security-academy.net/cgi-bin/phpinfo.php
```

### Information Leakage

The page displayed full PHP configuration information via `phpinfo()` output.

**Critical Finding:**

Scrolled through the environment variables section and found:

```
SECRET_KEY: tt7ibqee5bnpf1oc4m13logahuauax37
```

---

## 🚩 Solution

**Answer:** `tt7ibqee5bnpf1oc4m13logahuauax37`

---

## 💡 Key Takeaways

- **Debug pages** like `phpinfo()` should **never** be accessible in production
- **HTML comments** can leak sensitive paths even if links are hidden from UI
- **phpinfo()** exposes:
  - Environment variables (including secrets)
  - Server configuration
  - File paths and directory structure
  - Installed modules and versions
  - Database credentials (if in env vars)
- Always check **source code** for commented-out debugging features

---

## 🛡️ Remediation

- Remove or disable `phpinfo()` and similar debug tools in production
- Implement proper access controls for diagnostic endpoints
- Use environment-specific configurations
- Don't rely on "security through obscurity" (hiding links in comments)
- Regular security audits to identify exposed debug functionality

---

**Status:** ✅ Solved
