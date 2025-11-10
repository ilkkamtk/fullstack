# Minimal JS example: login and store JWT in localStorage

This very small example shows only the basics: POST credentials, store the returned JWT in `localStorage`, make an authenticated request, and logout. No helpers or try/catch are used here for clarity.

```javascript
// 1) Login and store token
async function login(username, password) {
  const res = await fetch('/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username, password }),
  });

  const data = await res.json();
  // token property may be named `token` or `accessToken` depending on your API
  const token = data.token || data.accessToken;
  localStorage.setItem('jwt', token);
  return data;
}

// 2) Use token for a single authenticated request
async function getProfile() {
  const token = localStorage.getItem('jwt');
  const res = await fetch('/api/auth/me', {
    headers: { Authorization: 'Bearer ' + token },
  });
  return res.json();
}

// 3) Logout
function logout() {
  localStorage.removeItem('jwt');
}

// Example usage:
// await login('alice', 's3cr3t');
// const profile = await getProfile();
// logout();
```

Note: In RESTful APIs authentication is implemented with tokens (e.g. JWT) sent in the `Authorization` header. Cookies are uncommon for pure REST APIs.

Common token storage patterns and trade-offs:

- localStorage — persists across page reloads and browser sessions, but is accessible to JavaScript and thus vulnerable to XSS.
- In-memory (app variable) — not persisted across reloads, so it's safer from XSS but the token is lost on refresh or new tabs.

Always use HTTPS. Consider token lifetime, refresh strategies, and your app's threat model when choosing where to store tokens in production.

Output Encoding and Sanitization: Never render unsanitized user input or third-party data directly into the DOM using methods like element.innerHTML. Always use safe DOM manipulation methods, text content properties, or dedicated sanitization libraries (like DOMPurify) to ensure any data is treated as text, not executable HTML.
