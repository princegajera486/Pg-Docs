# Authentication Module

## 1. Overview

**Purpose of this module:**  
The Authentication module is designed to provide a secure and reliable gateway for the PG Management System. It ensures that only authorized personnel can access sensitive property, tenant, and financial data.

**Business objective:**  
To restrict unauthorized access to the application, safeguard confidential business information, and maintain a seamless, centralized entry point for system administration.

**User who can access this module:**  
Only one authorized user (Owner/Admin) is permitted to access this system. Self-registration is strictly disabled to prevent unauthorized account creation.

**Expected system behaviour:**  
The system will prompt the user to provide their valid credentials (Email Address or Mobile Number, along with a Password). Upon successful validation, the system will establish a secure session and redirect the user to the central dashboard. In the event of a forgotten password, the system provides a secure recovery mechanism.

---

## 2. Screen Preview

```text
+--------------------------------------+
|              Login                   |
|--------------------------------------|
| Email / Mobile Number                |
| [________________________]           |
|                                      |
| Password                             |
| [________________________]           |
|                                      |
| ( Login )                            |
|                                      |
| Forgot Password?                     |
+--------------------------------------+
```

---

## 3. Screen Fields

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Email / Mobile Number** | Alphanumeric | Yes | Must be a valid Email format OR a valid 10-digit Mobile Number. Max length: 50. | `admin@pgsystem.com` or `9876543210` | Accepts either email or mobile to enhance usability. Leading/trailing spaces should be trimmed. |
| **Password** | Password | Yes | Minimum 8 characters. Must contain at least one uppercase letter, one number, and one special character. | `**********` | Input must be masked. Trailing/leading spaces should be trimmed. |
| **Login** | Button | N/A | Trigger authentication sequence upon click. | N/A | Button should be disabled during active API calls to prevent double submission. |
| **Forgot Password?** | Link | N/A | Redirects user to the password recovery screen. | N/A | Placed below the login button for easy discovery. |

---

## 4. Validations

**Frontend Validations:**
*   **Empty Fields:** If either the identifier or password field is left empty upon submission, display an inline error: "Please enter your Email/Mobile Number" or "Password is required".
*   **Input Trimming:** Automatically strip leading and trailing whitespace from the Email/Mobile field before submission.
*   **Format Check:** Determine if the input is an email or mobile number using Regex. If it contains an `@`, validate against standard email format. If numerical, ensure it matches the 10-digit mobile number format.
*   **Password Rules:** Ensure the password field is not empty before transmitting (complex rules are deferred to the backend to prevent leaking security policies).

**Backend Validations:**
*   **Invalid Email / Mobile Number:** If the identifier does not exist in the database, return a generic error message to prevent enumeration: "Invalid credentials."
*   **Incorrect Credentials:** Validate password against stored hash. Reject unauthorized access and log the failed attempt. Return generic "Invalid credentials."
*   **SQL Injection Prevention:** Use parameterized queries or ORM (Object-Relational Mapping) for all database interactions.
*   **XSS Prevention:** Sanitize all inputs to strip any potentially malicious scripts.
*   **Session Timeout:** Ensure active session expires after 30 minutes of inactivity.
*   **Maximum Login Attempts:** Restrict login attempts to 5 consecutive failures. 
*   **Account Lock Policy:** Lock the account for 15 minutes after 5 consecutive failed attempts and send a warning notification to the registered email.
*   **Rate Limiting:** Implement rate limiting (e.g., max 10 requests per minute per IP) on the authentication endpoint to thwart brute-force attacks.
*   **Case Sensitivity:** The Email address should be treated as case-insensitive, whereas the Password MUST be strictly case-sensitive.
*   **HTTPS Requirement:** Reject any authentication requests that do not originate over a secure HTTPS connection.

---

## 5. Business Rules

*   **Authorized Access Only:** Only predefined, authorized users (Owner/Admin) can log in.
*   **No Self-Registration:** The system shall not expose any public registration endpoints. Account provisioning is handled manually via database seeding or an internal tool.
*   **Protected Routes:** A user cannot access the dashboard or any internal module without presenting a valid, active session/token.
*   **Session Expiry:** A user session should automatically expire after a predefined period of inactivity (e.g., 30 minutes) to maintain security.
*   **Secure Storage:** Passwords should never be stored in plain text. They must be salted and hashed using strong cryptographic algorithms (e.g., bcrypt or Argon2).
*   **Secure Tokens:** Authentication tokens (such as JWTs) must be securely generated, signed, and transmitted.
*   **Active Status:** Only accounts marked with an 'Active' status in the database can successfully log in. Disabled or suspended accounts must be rejected with a "Your account has been deactivated" message.

---

## 6. Dependencies

*   **User Database:** Required to store and retrieve the admin credentials, account status, and login history.
*   **Authentication Service:** The core backend logic handling credential verification and token/session generation.
*   **Session Management:** Required to maintain user state across requests, utilizing either secure HTTP-only cookies or server-side sessions.
*   **Password Hashing Service:** Required for securely hashing user input to compare against the stored database hash (e.g., bcrypt library).
*   **Email Service:** Required to dispatch secure OTPs or password reset links for the "Forgot Password" flow.
*   **SMS Service:** Intended as a future dependency for mobile-based OTP recovery (if implemented later).
*   **Logging Service:** Required to track both successful logins and failed attempts for audit trails and anomaly detection.

---

## 7. Success Flow

1.  **Open Login Page:** The user navigates to the PG Management System login URL.
2.  **Enter Email/Mobile:** User inputs their registered Email Address or Mobile Number.
3.  **Enter Password:** User inputs their secure password.
4.  **Click Login:** User triggers the submit action.
5.  **Validate Credentials:** The frontend performs basic validation and sends an encrypted HTTPS request. The backend verifies the credentials against the database.
6.  **Create Session:** Upon successful validation, the backend generates a secure session or JWT token.
7.  **Redirect to Dashboard:** The system issues the token to the client and automatically redirects the user to the main Dashboard screen.

---

## 8. Exception Flow

*   **Empty Fields:** The user clicks login without entering credentials. The system blocks the request and highlights the required fields in red.
*   **Invalid Email/Mobile Format:** The user inputs a malformed email. The frontend displays an inline error: "Please enter a valid format."
*   **Wrong Password / Incorrect Credentials:** The user inputs valid formatting but an incorrect password. The backend rejects the request, and the frontend displays "Invalid credentials."
*   **Account Disabled:** The admin account is marked inactive in the database. The system rejects the login and displays "Your account is temporarily disabled. Please contact support."
*   **Too Many Failed Attempts (Account Locked):** The user fails 5 consecutive login attempts. The system locks the account for 15 minutes and displays "Account locked due to multiple failed attempts. Try again later."
*   **Server Unavailable:** The backend authentication service is down. The system displays a user-friendly fallback error: "System is temporarily unavailable. Please try again later."
*   **Session Expired:** An already logged-in user tries to navigate the system after 30 minutes of inactivity. The system terminates their access and redirects them back to the Login Page with a prompt: "Your session has expired. Please log in again."

---

## 9. Security Considerations

*   **Password Hashing:** Use `bcrypt` with a high work factor (e.g., 10+) for hashing passwords.
*   **Secure Cookies:** If using cookies for session management, they must be flagged as `HttpOnly`, `Secure`, and `SameSite=Strict`.
*   **JWT/Session Handling:** If using JWT, ensure tokens have a short expiration time and utilize refresh tokens stored securely.
*   **HTTPS:** Enforce strict Transport Layer Security (TLS/SSL) for all data in transit.
*   **CSRF Protection:** Implement Anti-CSRF tokens for session-based architectures to prevent cross-site request forgery.
*   **XSS Prevention:** Encode data before rendering on the frontend to prevent Cross-Site Scripting.
*   **SQL Injection Prevention:** Enforce the use of parameterized queries and prepared statements across all database transactions.
*   **Brute-Force Protection:** Utilize Rate Limiting and Account Lockout mechanisms to mitigate brute-force and dictionary attacks.
*   **Audit Logging:** Log all authentication events (IP address, user agent, timestamp, success/failure status) for security auditing.
*   **Secure Logout:** Ensure the logout action actively invalidates the token/session on both the client and server sides.

---

## 10. Future Enhancements

*   **Two-Factor Authentication (2FA):** Integrate TOTP (Time-Based One-Time Password) apps like Google Authenticator for an added layer of security.
*   **Biometric Login:** Support WebAuthn to allow login via fingerprint or FaceID on supported devices.
*   **Google / Microsoft Login:** Integrate OAuth2 to allow Single Sign-On (SSO) using enterprise accounts.
*   **Remember Me:** Implement a secure, long-lived persistent cookie to retain sessions for frequently used devices.
*   **Device Management:** Display a list of actively logged-in devices and allow the admin to remotely log out unrecognized sessions.
*   **Login History:** Provide a visible log of recent login locations and timestamps on the user profile page.
*   **Single Sign-On (SSO):** Enable seamless access across multiple internal PG management tools without re-authenticating.
