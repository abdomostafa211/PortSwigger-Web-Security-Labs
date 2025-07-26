## Lab: Information disclosure on debug page — APPRENTICE

### 🧠 Mission Description  
This lab contains a debug page that discloses sensitive information about the application.

### 🎯 Mission Objective  
Find and submit the `SECRET_KEY` environment variable.

### 🪜 Steps  
- Opened the page source → spotted a juicy comment:  
  `<!--  <a href=/cgi-bin/phpinfo.php>Debug</a> -->`  
- Opened the debug page directly:  
  `https://0ac50042038128a38067f3b8009c0048.web-security-academy.net/cgi-bin/phpinfo.php`  
- The page shows full PHP info output.  
- Scrolled through and found this line:  
  `SECRET_KEY	tt7ibqee5bnpf1oc4m13logahuauax37`

### 🧨 Final Payload  
```
tt7ibqee5bnpf1oc4m13logahuauax37
```

### 📝 Notes  
- Leaving debug tools like `phpinfo()` accessible in production environments is a big no-no.  
- These pages can leak environment variables, file paths, config details — basically everything an attacker dreams of.

**Status: Solved ✅**
