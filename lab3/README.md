# Server-Side Vulnerabilities: Path Traversal Lab

Welcome to my cybersecurity research repository. This project documents my hands-on learning experience identifying and exploiting **Server-Side Vulnerabilities**, specifically focusing on **Path Traversal (Directory Traversal)**. 

The practical portion of this research was successfully completed using the **PortSwigger Web Security Academy**.

---

## 🧠 Core Concepts Learned

### 1. What are Server-Side Vulnerabilities?
Server-side vulnerabilities are security flaws that exist entirely on the backend architecture (the web server, database, or internal logic) rather than the user's local device (client-side). They occur when a backend application mishandles incoming user instructions, allowing attackers to access unauthorized database rows, manipulate logic, or execute administrative system commands.

### 2. Understanding Path Traversal
Path Traversal (also known as Directory Traversal) is a critical vulnerability that allows an attacker to trick a web application into reading or writing files outside the intended public web root directory. 
* **The Mechanism (`../`):** Operating systems use `../` as a relative shortcut meaning "go up one directory level."
* **The Root Cause:** If input parameters representing file names are not strictly sanitized, an attacker can string multiple `../` shortcuts together to exit the application folder and wander anywhere across the server's file system.

### 3. Target Vectors: What are "Arbitrary Files"?
An **arbitrary file vulnerability** means an attacker is not restricted to pre-defined paths—they can read or overwrite *any* file of their choosing on the server, provided the web service account has system permissions to touch it. Common high-value targets include:
* **System Files:** Linux `/etc/passwd` (user listings) or `/etc/shadow` (password hashes).
* **Application Source Code:** Core application logic (e.g., `app.js`, `index.php`) to find deeper hidden flaws.
* **Environment Configuration:** `.env` or `config.json` files storing database passwords, API tokens, and encryption keys.

---

## 🛠️ Tooling & Hands-on Methodology (Burp Suite)

During this lab, I mastered advanced usage of **Burp Suite Professional/Community Edition** to intercept and manipulate traffic that is normally invisible inside standard web browser interfaces.

### Step-by-Step Lab Execution:
1. **Background Request Discovery:** While visiting a product page (`/product?productId=1`), I analyzed the HTML structure and realized the page loads images via a separate background engine request: `/image?filename=38.jpg`.
2. **HTTP Traffic Analysis:** Because image requests are frequently masked by proxy view filters, I configured the Burp Proxy **HTTP History MIME-Type Filter** to explicitly show image files.
3. **Request Isolation:** I captured the specific background endpoint `/image?filename=38.jpg` from the history log.
4. **Targeted Manipulation via Repeater:** I transferred the isolated packet into **Burp Repeater** to alter the live request state.
5. **Payload Injection:** I stripped out the legitimate image filename and injected a path traversal payload designed to step backward out of the public images directory into the root environment directory:
   ```http
   GET /image?filename=../../../etc/passwd HTTP/2
   Host: vulnerable-academy-instance.net
   ```
6. **Data Extraction:** Upon hitting **Send**, the vulnerable backend application processed the shortcuts, opened the Linux user directory, and returned the raw contents of `/etc/passwd` directly into the Burp Response pane, solving the challenge.

---

## 📸 Lab Proof & Screenshots

Below is the verified screenshot demonstrating the successful exploitation of the path traversal flaw and extraction of the `/etc/passwd` document.

![Portswigger/lab3](./images/pathtraversalresponse.png)
![Portswigger/lab3](./images/pathtraverse.png)

---

## 🛡️ Mitigation Strategies (How to Fix It)

To secure code repositories against path traversal attacks, I learned that developers should implement the following defenses:
1. **Input Validation & Whitelisting:** Strictly permit only alphanumeric characters or safe filenames. Explicitly block `../`, `/`, and `\`.
2. **Use Built-in Path Resolution Engines:** Use platform-native tools like Python's `os.path.basename()` or Node.js's `path.basename()` to strip out structural folder paths and evaluate *only* the flat filename.
3. **Enforce Realpath Verification:** Implement functions like `realpath()` in PHP to translate paths into their absolute locations and confirm they remain bound strictly inside the designated public directory map.
