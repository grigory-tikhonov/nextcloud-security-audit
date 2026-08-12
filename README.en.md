# Security Audit of yhe Nextcloud Cloud Service

## Project Description
As part of my final qualifying project, I conducted a practical security analysis of the self-hosted Nextcloud cloud storage service.
A lab environment was set up, attacks were performed (session hijacking, brute force, IDOR, malicious file upload, CSRF),
HTTP headers were analyzed, HTTPS and 2FA were configured.

## Objectives
- Set up a lab environment (VirtualBox + Ubuntu Server).
- Install and configure Nextcloud.
- Conduct attacks using Burp Suite.
- Analyze HTTP security headers.
- Harden the system (HTTPS, 2FA).
- Provide security recommendations.

## Tools & Technologies
- **Virtualization** - VirtualBox (Ubuntu Server).
- **Cloud Platform** - Nextcloud (Snap installation).
- **Security Testing** - Burp Suite Community Edition (Proxy, Inruder, Repeater).
- **Cookie spoofing** - DevTools (browser).
- **2FA** - Google Authenticator (TOTP).

## Research Steps

### 1. Lab Setup
- Installed Ubuntu Server in VirtualBox.
- Configured network (Host-Only adapter).
- Installed Nextcloud via Snap.
- Created test users `user1` and `user2` and uploaded files.

### 2. Attacks Conducted

#### Session Hijacking
- Intercepted HTTP login request via Burp Suite.
- Username and password transmitted in cleartext.
- Copied session cookies and pasted them into another browser - access was gained.
- **Vulnerability:** No HTTPS.
- **Solution:** Enable HTTPS with a self-signed certificate.

#### Brute Force Attack
- Used Burp Intruder.
- Used a dictionary of the most common passwords.
- System throttled the attack (`X-Nextcloud-Bruteforce-Throttled`) and applied rate limiting after a series of failed attempts; however, with a weak password, a brute-force attack is still possible.
- **Solution:** Enable 2FA (TOTP) + strong password policy.

#### IDOR (Insecure Direct Object Reference)
- Attempted to access another user’s file via direct WebDAV URL.
- Server correctly checked permissions and rejected the request (404 Not Found).
- **Vulnerability not found.**

#### Malicious File Upload (MIME Spoofing)
- Uploaded `test.php` and `shell.php.jpg` containing PHP code.
- Modified MIME type to `image/jpeg` (MIME spoofing).
- Files were stored but not executed — they stored outside web-root.
- **Vulnerability not found.**

#### CSRF (Cross-Site Request Forgery)
- Removed `requesttoken` header from file upload request.
- Server returned `401 Unauthorized` with message *"CSRF check not passed"*.
- **Protection works correctly.**

### 3. HTTP Security Headers Analysis
Headers checked:
- `Content-Security-Policy` (CSP)
- `Feature-Policy`
- `X-Content-Type-Options` (nosniff)
- `X-Frame-Options` (SAMEORIGIN)
- `Referrer-Policy` (same-origin)
- Cookies with `HttpOnly` and `SameSite=Lax`

**Conclusion:** Headers are configured correctly, but the `Secure` attribute was missing on cookies (due to HTTP).

### 4. Security Hardening
- **HTTPS:** Enabled via `sudo nextcloud.enable-https self-signed`.
- After HTTPS, cookies received the `Secure` attribute.
- **2FA:** Activated `Two-Factor TOTP Provider` app, configured Google Authenticator for user `user1`.

### 5. Final Assessment
After made changes, Nextcloud can be considered safe for use in private environment, provided that:
- HTTPS is enabled,
- 2FA is enabled,
- logs are monitored regularly,
- password policies are followed.

## Results
| Vulnerability | Found | How Fixed |
|---------------|-------|-----------|
| Session Hijacking | Yes | Enabled HTTPS |
| Brute Force | Yes (with weak passwords) | 2FA + Password Policy |
| IDOR | No | — |
| Malicious File Upload | No | Isolation from web-root |
| CSRF | No | Built-in CSRF token |

## Recommendations
- Always enable HTTPS (preferably with a valid CA-signed certificate).
- Enforce 2FA for all users.
- Use strong passwords and limit the number of login attempts.
- Regularly update Nextcloud and conduct penetration tests.
- Check HTTP security headers (CSP, HSTS, etc.).

## Links
- [Nextcloud Documentation](https://docs.nextcloud.com/)
- [Burp Suite](https://portswigger.net/burp)
- [OWASP Top 10](https://owasp.org/Top10/)

## Contact
- [Telegram](https://t.me/dreamlesslyyy)
- Email: grigoriytikhonov4@gmail.com

---

- **[Full thesis](docs/nextcloud-security-audit-thesis.pdf)**
- **Screenshots are available in the [images/](images/) folder**
