
---

## Lab 1: Unprotected admin functionality (Apprentice)

### 🎯 Mission Description  
This lab has an unprotected admin panel.

### 🎯 Mission Objective  
Solve the lab by deleting the user `carlos`.

### 🧪 Steps (My Approach)

1. Opened `/robots.txt` and found:  

User-agent: *
Disallow: /administrator-panel

→ This reveals the admin panel path.

2. Browsed to `/administrator-panel`.  
→ It opened without authentication.

3. Deleted user `carlos`.  
→ Success.

---

### ✅ Final Step  
Access the exposed admin panel directly:  

/administrator-panel


---

### 🧠 Notes  
- Always check `/robots.txt` and `/sitemap.xml`.  
- Security by obscurity is not real security.  
- Admin panel exposed with no authentication = critical flaw.  

---

### ✅ Status: Solved 🎉  
