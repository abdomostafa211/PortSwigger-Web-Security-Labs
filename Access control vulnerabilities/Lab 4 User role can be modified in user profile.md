## Lab 4 User role can be modified in user profile (Apprentice)

### Mission Description  
The app has an admin panel at `/admin`, but it’s restricted to users with `roleid=2`. Normal users only have `roleid=1`.  

### Mission Objective  
Gain access to the admin panel and delete the user **carlos**.  

### Steps  
1. Login with the given creds → `wiener:peter`.  
2. Try to access `/admin` → blocked.  
3. Open **Storage/Cookies** in browser.  
4. Spot the value `roleid=1`.  
5. Change it to `roleid=2`.  
6. Reload page → now `/admin` is accessible.  
7. Inside the panel, delete **carlos**.  

### Notes  
- No payload here, just a **role flip**.  
- This is a perfect example of **Broken Access Control**.  

**Status: Solved ✅**
