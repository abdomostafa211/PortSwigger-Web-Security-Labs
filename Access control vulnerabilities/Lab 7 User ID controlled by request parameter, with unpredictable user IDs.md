### Lab 7 User ID controlled by request parameter, with unpredictable user IDs (Apprentice)

**Mission Description**  
The lab has a horizontal privilege escalation vulnerability on the user account page, but instead of normal usernames, it identifies users with GUIDs.  

**Mission Objective**  
Find the GUID for `carlos` and submit his API key as the solution.  

**Steps**  
- Log in with `wiener:peter`.  
- Capture the login request in Burp.  
- Notice the request looks like:  

GET /my-account?id=wiener

- Change `id=wiener` to `id=carlos`:  

GET /my-account?id=carlos

- You’ll now get Carlos’s account details along with his API key.  
- Submit the API key to solve the lab.  

**Final Payload**  

GET /my-account?id=carlos


**Notes**  
- Even though the lab says GUIDs are used, here it still accepts predictable IDs (`wiener`, `carlos`).  
- This is a classic **IDOR / Insecure Direct Object Reference** issue.  

**Status: Solved ✅**
