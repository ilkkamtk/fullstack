# Web Application Security

## Data Security

Data security refers to the protective measures and techniques used to safeguard data from unauthorized access, alteration, or destruction. Its main goal is to ensure data integrity, availability, and confidentiality. Techniques for ensuring data security include:

- **Access Control**: Ensuring that only authorized individuals can access specific data sets.
- **Encryption**: Transforming data into a secure format that unauthorized users cannot easily interpret.
- **Backups**: Creating copies of data to prevent loss in case of hardware failure, natural disasters, or other disruptions.
- **Network Security**: Protecting data in transit across networks from interception or intrusion.
- **Physical Security**: Safeguarding the physical devices and infrastructure where data is stored.
- **Vulnerability Management**: Regularly updating software and systems to protect against known security vulnerabilities.

Data security techniques are applied across different layers, including the physical hardware, the software, the network, and the users who interact with the data. The user is often the weakest link in the security chain, so it’s important to educate users about security best practices and to implement proper access controls to prevent unauthorized access.

## Data Privacy

Data privacy means proper handling, processing, and storage of personal information, ensuring that it’s used in a fair, legal, and transparent manner. Privacy is more about the rights of individuals regarding their personal data:

- **Consent**: Ensuring that individuals have consented to the collection and use of their personal data.
- **Minimal Collection**: Collecting only the data that is necessary for the defined purpose.
- **Data Anonymization**: Removing or modifying personal information so that individuals cannot be readily identified.
- **Transparency**: Being clear with individuals about how their data is being used.
- **Legal Compliance**: Adhering to data protection laws and regulations, such as GDPR (General Data Protection Regulation) in the European Union

Data privacy is often ensured through policies, legal agreements (like privacy policies), and user controls that allow individuals to understand and manage how their data is used.

While data security and data privacy are distinct, they are deeply interconnected. Effective data privacy cannot exist without robust data security measures, as securing data is fundamental to maintaining its privacy. However, you can have high data security without necessarily ensuring data privacy. For instance, a database might be well-secured against unauthorized access but could still be used in ways that violate privacy if proper data handling policies are not in place.

## Practical Security in REST APIs

1. **[HTTPS](https://en.wikipedia.org/wiki/HTTPS) (SSL/TLS) for Communication**
   - HTTP over [TLS/SSL](https://en.wikipedia.org/wiki/Transport_Layer_Security) (TLS is the new progression of SSL but the _term_ SSL is still generally used)
   - [SSL certificate](https://www.kaspersky.com/resource-center/definitions/what-is-a-ssl-certificate) _authenticates_ a website's identity and enables an encrypted connection
   - Protection of the _privacy_ and _integrity_ of the exchanged data
   - Always use HTTPS to encrypt data in transit and protect against eavesdropping, man-in-the-middle attacks, and data tampering.
   - When using a Python + Flask application, it is generally better to handle TLS termination in a reverse proxy such as Apache or Nginx, rather than inside the Flask app itself.
   - All unsecure HTTP connections (port 80) should be automatically redirected to HTTPS (port 443) by using [HTTP status codes](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html), the 3XX codes are redirect, 301 means Moved Permanently.
   - (Typically, for localhost development environments secure connections are not needed)
1. **Authentication**
   - Implement strong authentication mechanisms to ensure that only authorized users or systems can access the API.
   - Common methods include API keys, OAuth, JWT (JSON Web Tokens), or other token-based systems.
   - 2FA (Two-Factor Authentication) or MFA (Multi-Factor Authentication) can be used to add an extra layer of security.
1. **Authorization**
   - Define and enforce proper access controls to restrict users or systems to only the resources they are allowed to access.
   - Implement role-based access control (RBAC) or attribute-based access control (ABAC) as needed. (Models can be combined.)
   - RBAC is a widely used access control model based on predefined roles within an organization. In this model, access permissions are assigned to roles rather than to individual users. When a user is assigned a role, they inherit the permissions associated with that role.
     - For example, a user might be assigned the role of "admin," which comes with permissions to create, read, update, or delete data. Another user might be assigned the role of "guest," which allows them to only read data.
   - ABAC is a more flexible and fine-grained access control model that bases access decisions on attributes. These attributes can be related to the user, the resource being accessed, the action being performed, and the context (like time of day or IP address).
     - For example, a user might be allowed to read data only during business hours or only from a specific IP address.
1. **Token Management**
   - If using tokens (such as JWT), ensure proper token validation, expiration checks, and secure storage.
   - Check that the user stored in the token is still valid and has the required permissions.
     - For example, if a user is deleted from the database, their token should no longer be valid.
   - Implement token revocation mechanisms in case of compromised tokens.
     - In practice, this means that the server should keep track of issued tokens and their validity. If a token is compromised, it can be added to a blacklist and rejected by the server.
1. **Secure Data Transmission**
   - Avoid exposing sensitive information in the URL; use request headers or the request body for transmitting sensitive data.
   - Be cautious with query parameters and ensure they do not expose sensitive information.
1. **Data Protection**
   - Protect persistent data
   - Implement strict access control. This includes defining database users and privileges ensuring that users/applications have access only to the data necessary for their role.
   - Encrypt sensitive data (like passwords) in the database and ensure that only authorized individuals can access it.
1. **Input Validation**
   - Validate and sanitize all incoming data to prevent injection attacks, such as SQL injection, NoSQL injection, or script injection.
   - Enforce strong validation for data types, lengths, and formats.
1. **Output Encoding**
   - Encode output data to protect against Cross-Site Scripting (XSS) attacks. HTML, XML, or JSON encoding should be applied depending on the output format.
1. **Error Handling**
   - Provide meaningful error messages to clients without revealing sensitive information.
   - Log errors securely on the server side for monitoring and debugging.
1. **[Same-origin policy (SOP)](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) and [Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)**
   - SOP is a security mechanism that restricts how a document or script loaded by one [origin](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#definition_of_an_origin) can interact with a resource from another origin.
   - CORS is an HTTP-header based mechanism that allows a server to indicate any origins other than its own from which a browser should permit loading resources.
   - Implement proper CORS headers to control which domains are allowed to access the API if needed.
   - Be restrictive with CORS configurations to prevent unauthorized cross-origin requests.
1. **Rate Limiting**
   - Implement rate limiting to prevent abuse, brute-force attacks, or denial-of-service attacks.
   - Configure sensible rate limits based on the nature of the API.
   - Can be implemented in the API itself or in a reverse proxy.
1. **Logging and Monitoring**
   - Log security-relevant events and regularly monitor logs for suspicious activity.
   - Set up alerts for unusual patterns or potential security incidents.
1. **API Versioning**
   - Consider versioning your API to avoid breaking changes and to allow clients to migrate at their own pace.
1. **Security Headers**
   - Utilize security headers, such as [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) and [Strict-Transport-Security (HSTS)](https://developer.mozilla.org/en-US/docs/Glossary/HSTS), to enhance overall security.
1. **Security Reviews and Audits**
   - Regularly conduct security reviews and audits of your API design, code, and infrastructure.
   - Stay informed about security best practices and vulnerabilities relevant to your technology stack.

## OWASP Top 10

> The OWASP® Foundation works to improve the security of software through its community-led open source software projects, hundreds of chapters worldwide, tens of thousands of members, and by hosting local and global conferences.

- [OWASP Top 10](https://owasp.org/www-project-top-ten/) lists the top 10 most critical web application security risks.

## Application Security in Express

Reading:

- [Handling CORS in Flask](https://flask-cors.readthedocs.io/en/latest/)
- [Flask Security Considerations](https://flask.palletsprojects.com/en/stable/web-security/)
- [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)
- [All OWASP Cheat Sheets](https://cheatsheetseries.owasp.org/)

### Password Security

Reading:

- [Traficom National Cyber Security Centre Finland: Strong Password Guidelines](https://www.kyberturvallisuuskeskus.fi/en/ncsc-news/instructions-and-guides/longer-better-how-create-strong-password)
- [NIST Password Guidelines 2024](https://www.auditboard.com/blog/nist-password-guidelines/)
- [Salted Password Hashing - Doing it Right](https://crackstation.net/hashing-security.htm)
- xkcd comic classics: [Password Strength](https://xkcd.com/936/), [Password Reuse](https://xkcd.com/792/)
- [How long it takes to crack a password?](https://www.hivesystems.com/password)
- [About AI password cracking](https://www.rd.com/article/ai-password-cracking/)

### Enabling CORS for development

Needed for cross-origin requests, e.g., when the front-end is served from a server which is different domain from the Flask server running the back-end. For example, front-end running on `http://localhost:3000` and back-end on `http://localhost:5000`.

1. Install [Flask cors package](https://flask-cors.readthedocs.io/en/latest/) :

   ```bash
   pip install flask-cors
   ```

1. Enable CORS for all origins and routes in app.py:

   ```python
   from flask_cors import CORS

   app = Flask(__name__)
   CORS(app)  # This will enable CORS for all routes and origins
   ```

In production, you should consider more restrictive CORS settings. If you serve the front-end from the same domain (server and port) as the back-end (e.g. use Express static file server for the front-end files), you might not need CORS at all.
