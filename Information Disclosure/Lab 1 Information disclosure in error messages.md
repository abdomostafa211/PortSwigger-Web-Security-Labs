## Lab: Information disclosure in error messages — APPRENTICE

### 🧠 Mission Description  
This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework.

### 🎯 Mission Objective  
Obtain and submit the version number of the exposed framework.

### 🪜 Steps  
- First, I viewed the page source → nothing interesting.  
- Then I opened `/robots.txt` → also nothing found.  
- I noticed the product page URL looked like this:  
  `https://0a18006004ee1ffc81180c3100b7009f.web-security-academy.net/product?productId=2`  
- So I started testing with the `productId` parameter:  
  - Tried `productId=0` → Not found  
  - Tried `productId=999` → Also not found  
  - Tried `productId=A` → Boom 💥 got an error:

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

- That stack trace leaked the framework and its version.  
- So the flag is: `2 2.3.31`

### 🧨 flag is  
```
2 2.3.31
```

### 📝 Notes  
- Triggering a type mismatch in the parameter (using a letter instead of a number) revealed a detailed Java error.  
- These stack traces often leak sensitive backend info like framework names and versions.  
- Apache Struts is a known framework with critical vulnerabilities, so exposing the version is a big deal.

**Status: Solved ✅**

