# API Security Essentials

---

## 1. CIA Triad: Confidentiality, Integrity, Availability

**Confidentiality:**

- Preventing unauthorized access to sensitive data.
- **Examples:**
  1. Encrypting data in transit using HTTPS (TLS).
  2. Storing user passwords hashed, not in plain text.
  3. Encrypting sensitive fields in a database (e.g. credit card numbers).
  4. Using JWE (JSON Web Encryption) for confidential tokens.
  5. Access control policies (e.g. data only visible to certain users).

**Integrity:**

- Ensuring data has not been tampered with or altered by unauthorized parties.
- **Examples:**
  1. Digitally signing JWTs using JWS to detect modifications.
  2. Checksums or hashes to validate data integrity during transfer or storage.
  3. Database constraints to prevent invalid or corrupted data.
  4. Audit trails/logging to detect and trace unauthorized changes.

**Availability:**

- Ensuring that systems and data are accessible when needed.
- **Examples:**
  1. Implementing rate limiting and DoS protection on APIs.
  2. Backups and redundancy (e.g. failover databases).
  3. Health checks and auto-scaling in cloud infrastructure.
  4. Disaster recovery plans and regular testing.

---

## 2. OAuth2 (Delegated Authorization)

**Definition:**

- OAuth2 is an open standard for delegated authorization—allowing one application (the client) to access resources on behalf of a user without sharing their credentials (e.g. using Google/Facebook login).

**How it works (high-level):**

1. The user wants to let an app access their data (e.g., calendar, profile) in another system.
2. The app redirects the user to the authorization server (e.g., Google), where the user logs in and grants permission.
3. The authorization server redirects back to the app with an authorization code.
4. The app exchanges the code for an access token.
5. The app uses the token to access protected APIs.

**Code Example (.NET):**

```csharp
// Example: Using OAuth2 with Microsoft.AspNetCore.Authentication.OpenIdConnect
services.AddAuthentication(options => {
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options => {
    options.Authority = "https://login.microsoftonline.com/common";
    options.ClientId = "{client-id}";
    options.ClientSecret = "{client-secret}";
    options.ResponseType = "code";
    options.SaveTokens = true;
});
```

**Code Example (Python FastAPI):**

```python
from fastapi import FastAPI, Depends
from fastapi.security import OAuth2AuthorizationCodeBearer

app = FastAPI()
oauth2_scheme = OAuth2AuthorizationCodeBearer(
    authorizationUrl="https://auth-server.com/auth",
    tokenUrl="https://auth-server.com/token"
)

@app.get("/profile")
def read_profile(token: str = Depends(oauth2_scheme)):
    # Use token to access user's data
    return {"token": token}
```

---

## 3. Identification vs Authentication vs Authorization

| Term           | What is it?                                | Example                                       |
| -------------- | ------------------------------------------ | --------------------------------------------- |
| Identification | Claiming who you are                       | Typing your username or email                 |
| Authentication | Verifying that you are who you claim to be | Entering your password or OTP                 |
| Authorization  | Granting permission to access resources    | Allowing access to /admin if you are an admin |

**Summary:**

- **Identification:** “Who am I?” (Claiming identity)
- **Authentication:** “Can you prove it?” (Validating credentials)
- **Authorization:** “What can I do?” (Granting permissions)

---

## 4. JWT, JWS, JWE Explained

### **JWT (JSON Web Token)**

**Definition:**\
JWT is just a container or format for representing claims securely between two parties.

**What does it do?**

- It can be just encoded, or signed, or encrypted—or both.

**Structure:**

- `header.payload.signature` (if signed/JWS)
- Or as a compact format if encrypted/JWE

> By itself, “JWT” doesn’t tell you if it’s just signed or also encrypted.

---

### **JWS (JSON Web Signature)**

**Definition:**\
JWS is a standard for signing data—creating integrity and authenticity but not confidentiality.

**What does it do?**

- Takes a payload, signs it (with symmetric or asymmetric keys), and outputs a JWT.
- The payload is visible to anyone with the token, but any modification will break the signature.

**Use-case:**\
Most “JWT” tokens in APIs are actually JWS—signed, not encrypted.

---

### **JWE (JSON Web Encryption)**

**Definition:**\
JWE is a standard for encrypting data—creating confidentiality (nobody can read the payload except the intended recipient).

**What does it do?**

- Takes a payload, encrypts it (with a public key or a shared key), so only the holder of the private/decryption key can read the claims.
- You can also sign and then encrypt a JWT (so you have both integrity and confidentiality).

**Use-case:**\
Used when you want to hide the contents of the token, e.g. sensitive PII, medical data, etc.

---

## **Key Differences Summarized**

|               | **JWS**                      | **JWE**                                       | **JWT**                  |
| ------------- | ---------------------------- | --------------------------------------------- | ------------------------ |
| Means         | Signed, not encrypted        | Encrypted (optionally signed)                 | The “format” (container) |
| Confidential? | ❌ Anyone can read payload    | ✅ Only receiver can read                      | Depends on use           |
| Integrity?    | ✅ Yes (signature)            | ✅ Yes (if signed before encryption)           | Depends on use           |
| API use       | Auth tokens, session, claims | Sensitive info, very rare in typical API auth | Used everywhere          |

---

## **How to Answer in an Interview**

> **"JWT stands for JSON Web Token, which is just a format—a compact way to represent claims as a string. There are two main types: JWS and JWE. JWS stands for JSON Web Signature, which means the token is signed, so you get integrity and authenticity, but the payload is still readable by anyone. Most JWTs used in authentication are JWS.
>
> JWE, on the other hand, stands for JSON Web Encryption, which means the token is encrypted, so only the intended recipient can read the payload. You can even combine them—sign, then encrypt—to get both integrity and confidentiality.
>
> In summary: JWS = signed (integrity), JWE = encrypted (confidential), JWT = the format that can be used for either or both."**

---

## **Bonus: When to Use Each?**

**JWT (Format Only):**\
When you need a standard, compact, and self-contained way to represent claims that can be signed and/or encrypted. JWT on its own is just the format—it becomes useful with JWS or JWE depending on your use case.

**JWS (Signed JWT):**\
Use for API auth, sessions, user claims—when you want anyone with the secret/public key to verify, but you don’t care if the data is visible.

**JWE (Encrypted JWT):**\
Use when you’re transmitting sensitive data, and you want only the recipient to be able to read it.

---




---

## 5. Certificates & mTLS (Mutual TLS)

**Certificate:**

- A digital file proving the identity of a server or client. Contains a public key, info about the owner, and a digital signature from a trusted authority.
- Used to establish trust in HTTPS, mTLS, etc.

**Mutual TLS (mTLS):**

- Both client and server present certificates to prove their identity (two-way authentication).
- Prevents “man-in-the-middle” attacks.

**Example Flow:**

1. Client connects to API over HTTPS, verifies server’s certificate.
2. Server requests client certificate, verifies it.
3. Only trusted clients and servers communicate.

---

## 6. Cookies vs Sessions vs Refresh Tokens

| Approach       | How it Works                                                   | Pros                                      | Cons                                   |
| -------------- | -------------------------------------------------------------- | ----------------------------------------- | -------------------------------------- |
| Cookies        | Store small pieces of data in browser, auto-sent with requests | Simple to implement, works with browsers  | Vulnerable to XSS/CSRF, size limit     |
| Sessions       | Server stores user state (session ID in cookie)                | Secure, easy to invalidate                | Not stateless, server memory           |
| Refresh Tokens | Long-lived token to obtain new access tokens                   | Stateless, scalable, works for mobile/web | Need to store securely, risk if leaked |

**Benchmark / Use Case:**

- **Cookies:** Great for classic web apps, less secure for APIs (stateless JWT preferred).
- **Sessions:** Good for small-scale, traditional apps; harder to scale.
- **Refresh Tokens:** Best for modern APIs, mobile apps, and microservices—stateless and secure if handled right.

---
