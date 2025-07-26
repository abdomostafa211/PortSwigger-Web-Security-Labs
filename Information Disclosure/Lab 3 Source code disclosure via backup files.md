## Lab: Source code disclosure via backup files — APPRENTICE

### 🧠 Mission Description  
This lab leaks its source code via backup files stored in a hidden directory.

### 🎯 Mission Objective  
Find the hard-coded database password inside the leaked source code and submit it.

### 🪜 Steps  
- Opened the page source → nothing useful.  
- Checked `/robots.txt` → found this entry:
  ```
  User-agent: *
  Disallow: /backup
  ```
- Visited the hidden directory:  
  `https://0a7400d70373279681d29330001a00c0.web-security-academy.net/backup`  
- Found an exposed file: `ProductTemplate.java.bak`  
- Opened it and read the source code. Inside the `readObject` method, the DB credentials were hard-coded:  
  ```java
  ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
      "org.postgresql.Driver",
      "postgresql",
      "localhost",
      5432,
      "postgres",
      "postgres",
      "zxaoig986rm5v15t4s5232xsjnd1mylv"
  ).withAutoCommit();
  ```
- The last argument is the database password. That’s our flag!

### 🧨 Final Payload  
```
zxaoig986rm5v15t4s5232xsjnd1mylv
```

### 📝 Notes  
- Backup files (`.bak`, `.old`, etc.) should never be publicly accessible — always restrict or remove them.  
- `robots.txt` is not a security feature — it just gives attackers a roadmap if misused.  
- Hard-coding secrets in source code is risky, especially if the code leaks.

**Status: Solved ✅**
