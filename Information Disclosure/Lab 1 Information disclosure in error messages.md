# Information Disclosure in Error Messages

**Difficulty:** `APPRENTICE` | **Category:** `Information Disclosure`

---

## 🎯 Lab Goal

Obtain and submit the version number of the framework exposed through verbose error messages.

---

## 📋 Lab Description

This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework.

---

## 🔎 Reconnaissance

Started with basic enumeration:

1. **Checked page source** → Nothing interesting
2. **Checked `/robots.txt`** → No useful info
3. **Analyzed URL structure:**
   ```
   https://[lab-id].web-security-academy.net/product?productId=2
   ```

The `productId` parameter looked like a potential attack vector.

---

## 🧪 Testing & Exploitation

### Parameter Testing

Tested different values for `productId`:

| Input | Result |
|-------|--------|
| `productId=0` | Not found |
| `productId=999` | Not found |
| `productId=A` | 💥 **Error triggered!** |

### Error Analysis

Sending `productId=A` caused a type mismatch and revealed:

```
Internal Server Error: java.lang.NumberFormatException: For input string: "A"
[Stack trace truncated...]

Apache Struts 2 2.3.31
```

<details>
<summary>📄 Full Stack Trace (click to expand)</summary>

```
Internal Server Error: java.lang.NumberFormatException: For input string: "A"
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Integer.parseInt(Integer.java:661)
	at java.base/java.lang.Integer.parseInt(Integer.java:777)
	at lab.b.d.m.c.y(Unknown Source)
	at lab.n.q.y.j.o(Unknown Source)
	at lab.n.q.p.m.q.O(Unknown Source)
	at lab.n.q.p.n.lambda$handleSubRequest$0(Unknown Source)
	at j.h.r.i.lambda$null$3(Unknown Source)
	at j.h.r.i.o(Unknown Source)
	at j.h.r.i.lambda$uncheckedFunction$4(Unknown Source)
	at java.base/java.util.Optional.map(Optional.java:260)
	at lab.n.q.p.n.L(Unknown Source)
	at lab.server.p.b.w.l(Unknown Source)
	at lab.n.q.j.P(Unknown Source)
	at lab.n.q.j.l(Unknown Source)
	at lab.server.p.b.o.c.g(Unknown Source)
	at lab.server.p.b.o.q.lambda$handle$0(Unknown Source)
	at lab.b.z.x.a.r(Unknown Source)
	at lab.server.p.b.o.q.Z(Unknown Source)
	at lab.server.p.b.h.W(Unknown Source)
	at j.h.r.i.lambda$null$3(Unknown Source)
	at j.h.r.i.o(Unknown Source)
	at j.h.r.i.lambda$uncheckedFunction$4(Unknown Source)
	at lab.server.am.W(Unknown Source)
	at lab.server.p.b.h.I(Unknown Source)
	at lab.server.p.w.y.c(Unknown Source)
	at lab.server.p.z.K(Unknown Source)
	at lab.server.p.k.K(Unknown Source)
	at lab.server.a4.R(Unknown Source)
	at lab.server.a4.I(Unknown Source)
	at lab.m.j.lambda$consume$0(Unknown Source)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1144)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642)
	at java.base/java.lang.Thread.run(Thread.java:1583)
Apache Struts 2 2.3.31
```
</details>

---

## 🚩 Solution

**Answer:** `2 2.3.31`

---

## 💡 Key Takeaways

- **Type confusion attacks** can trigger verbose errors that leak sensitive info
- **Stack traces** often expose:
  - Framework names and versions
  - Internal code structure
  - File paths and dependencies
- **Apache Struts 2.3.31** has known critical vulnerabilities (e.g., CVE-2017-5638)
- Production systems should use **custom error pages** instead of detailed stack traces

---

## 🛡️ Remediation

- Implement generic error messages for users
- Log detailed errors server-side only
- Disable debug mode in production
- Keep frameworks updated to latest versions

---

**Status:** ✅ Solved
