# User ID Controlled by Request Parameter with Unpredictable User IDs

**Difficulty:** `APPRENTICE` | **Category:** `Access Control`

---

## 🎯 Lab Goal

Find Carlos's GUID and use it to retrieve his API key.

---

## 📋 Lab Description

The lab has a horizontal privilege escalation vulnerability on the account page. Users are identified by GUIDs instead of simple usernames.

---

## 🔎 Reconnaissance

### Initial Access

Logged in with provided credentials:

| Field | Value |
|-------|-------|
| Username | `wiener` |
| Password | `peter` |

### Account Page Analysis

Navigated to **My Account** and observed the request:

```http
GET /my-account?id=cee1597a-9c83-4a1c-ac33-eae19ca390d9 HTTP/2
```

**Response:** Shows wiener's username and API key

🔍 **Challenge:** Need to find Carlos's GUID to access his account.

---

## 🧪 Testing & Exploitation

### Step 1: GUID Enumeration

Explored the **Blogs** section looking for information leakage.

### Step 2: Author Profile Discovery

Clicked on a blog author's profile and noticed the URL structure:

```
/blogs?userId=391f068a-1c52-47a1-bc1e-98f479bd9dd0
```

**Finding:** The blog posts expose author GUIDs publicly!

### Step 3: Identify Carlos's GUID

Found a blog post authored by **Carlos** with exposed GUID:

```
391f068a-1c52-47a1-bc1e-98f479bd9dd0
```

### Step 4: Access Carlos's Account

Modified the account request with Carlos's GUID:

```http
GET /my-account?id=391f068a-1c52-47a1-bc1e-98f479bd9dd0 HTTP/2
```

**Response:**
```
Your username is: carlos
Your API Key is: 4IA8tQ6B6zMFPbOVGUeVW3bv5aJzePMt
```

---

## 🚩 Solution

**Carlos's GUID:** `391f068a-1c52-47a1-bc1e-98f479bd9dd0`

**API Key:** `4IA8tQ6B6zMFPbOVGUeVW3bv5aJzePMt`

---

## 💡 Key Takeaways

### GUIDs Are Not Security

- **GUIDs provide obscurity, not security**
- If exposed elsewhere in the application, they can be harvested
- Authorization must still be enforced regardless of ID type

### Common GUID Leakage Points

Where attackers find "hidden" GUIDs:
- **Public profiles** (blogs, forums, comments)
- **API responses** (user lists, search results)
- **Email headers** (notification emails)
- **Referrer headers**
- **Analytics data**
- **JavaScript code**
- **Error messages**

### IDOR with Unpredictable IDs

Even with GUIDs, IDOR vulnerabilities exist when:
1. IDs are exposed in other contexts
2. No authorization check validates ownership
3. The application trusts the ID parameter

---

## 🛡️ Remediation

### Secure Implementation

**❌ Bad (No Authorization):**
```python
@app.route('/my-account')
def account():
    guid = request.args.get('id')
    user = User.query.filter_by(guid=guid).first()
    return jsonify(user.to_dict())
```

**✅ Good (With Authorization):**
```python
@app.route('/my-account')
@login_required
def account():
    requested_guid = request.args.get('id')
    current_user_guid = session['user_guid']
    
    # Verify ownership
    if requested_guid != current_user_guid:
        abort(403)
    
    user = User.query.filter_by(guid=requested_guid).first()
    return jsonify(user.to_dict())
```

**✅ Best (Implicit Reference):**
```python
@app.route('/my-account')
@login_required
def account():
    # Don't accept user input for own account
    user = User.query.get(session['user_id'])
    return jsonify(user.to_dict())
```

### Best Practices

- ✅ **Always enforce authorization** regardless of ID type
- ✅ **Minimize GUID exposure** in public contexts
- ✅ **Use indirect references** (session tokens) when possible
- ✅ **Separate public and private identifiers**
- ✅ **Implement rate limiting** to prevent enumeration
- ✅ **Log suspicious access patterns**

### Defense in Depth

```python
# Example: Separate public and private IDs
class User:
    id = Column(Integer)           # Internal database ID
    guid = Column(String)          # Private account identifier
    public_id = Column(String)     # Public profile identifier
    
# Blog author endpoint (public)
@app.route('/author/<public_id>')
def author_profile(public_id):
    user = User.query.filter_by(public_id=public_id).first()
    return render_template('public_profile.html', user=user)

# Account endpoint (private)
@app.route('/my-account')
@login_required
def my_account():
    user = User.query.get(session['user_id'])
    return render_template('private_account.html', user=user)
```

---

**Status:** ✅ Solved
