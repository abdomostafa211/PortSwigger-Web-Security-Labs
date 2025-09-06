
---

## Lab 2: Unprotected admin functionality with unpredictable URL (Apprentice)

### 🎯 Mission Description  
This lab has an unprotected admin panel, but its location is hidden and disclosed somewhere in the application.

### 🎯 Mission Objective  
Access the admin panel and use it to delete the user `carlos`.

### 🧪 Steps (My Approach)

1. Checked `/robots.txt` and `sitemap.xml`.  
   → Nothing useful.

2. Viewed page source and found hidden JavaScript:  

   ```javascript
   var isAdmin = false;
   if (isAdmin) {
      var topLinksTag = document.getElementsByClassName("top-links")[0];
      var adminPanelTag = document.createElement('a');
      adminPanelTag.setAttribute('href', '/admin-4yl2mj');
      adminPanelTag.innerText = 'Admin panel';
      topLinksTag.append(adminPanelTag);
      var pTag = document.createElement('p');
      pTag.innerText = '|';
      topLinksTag.appendChild(pTag);
   }

→ This discloses the real admin panel path: /admin-4yl2mj.

    Browsed to /admin-4yl2mj.
    → Access granted.

    Deleted user carlos.
    → Success.

✅ Final Step

Hidden path found in JavaScript:

/admin-4yl2mj

🧠 Notes

    Hidden admin panels often show up in JS code.

    Always search for admin keywords in source.

    Unpredictable URLs aren’t secure if they’re still referenced in code.

✅ Status: Solved 🎉
