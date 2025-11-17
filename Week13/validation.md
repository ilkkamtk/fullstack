# Validation

---

## **Validation in General**

- Data validation ensures that information entering a system is accurate, complete, and meaningful.
- It maintains data integrity by preventing malformed, inconsistent, or unexpected input.
  - Examples include checking for required fields, data types, formats (like email), and value ranges.
- Validation reduces security risks by blocking harmful or malicious data before it reaches sensitive parts of the system.
  - Different types of injection attacks (e.g., SQL injection, XSS) can be prevented through proper validation.
- Reliable validation improves the overall stability and trustworthiness of a web application.

---

## **Server-Side Validation**

- Server-side validation occurs on the backend, where the application logic and database interactions are controlled.
- It is the final and authoritative layer of validation, ensuring that no unsafe data is stored or processed.
- Even if client-side validation exists, the server must always perform its own checks because client logic can be bypassed.
- Most backend frameworks provide built-in or easily integrated validation tools to simplify this process.

---

## **Client-Side Validation**

- Client-side validation runs in the user’s browser and provides immediate feedback before data is submitted.
- It improves user experience by catching simple mistakes early, such as missing fields or invalid formats.
- While convenient, client-side validation is not secure on its own because browser scripts can be disabled or manipulated.
- It should always complement, **never** replace—server-side validation.
- Client side validation is mainly for usability purposes.

**Pseudo-code example:**

```pseudo
onFormSubmit():
    if field.email does not match pattern /^\S+@\S+\.\S+$/:
        show "Email is invalid"
        stop submission

    if field.password.length < 8:
        show "Password must be at least 8 characters"
        stop submission

    continue submission
```

---

## Validation Strategies

- **Whitelist vs. Blacklist**: Prefer whitelisting acceptable input (defining what is allowed) over blacklisting (defining what is not allowed) to minimize risks.
- **Data Types**: Ensure that input matches expected data types (e.g., integers, strings, dates).
- **Length Checks**: Validate the length of input fields to prevent buffer overflows or excessive data.
- **Format Checks**: Prefer library's built-in functions over regular expressions to validate formats like email addresses, phone numbers, and postal codes etc.
- **Cross-Field Validation**: Some validations may depend on the relationship between multiple fields, such as ensuring a start date is before an end date.
- **Sanitization**: In addition to validation, you could sanitize inputs to remove or escape potentially harmful characters.

---

## **Best Practices**

- Always validate data on the server, even if the client has already performed validation.
- Never assume user input is safe, and expect that malicious users will attempt to bypass client checks. So basically assume that all input is malicious.
- Keep validation rules consistent across both layers to avoid contradictory behavior.
- Provide clear, actionable error messages that help users correct their input.
- Log validation failures when appropriate to support debugging and security monitoring.
- Try to maintain one source of truth for validation logic to reduce redundancy and potential errors.

### One source of truth

- One source of truth means having a single, centralized location for your validation logic. For example in a backend framework, you might define validation rules in your data models or schemas.
- This approach minimizes redundancy, ensures consistency, and simplifies maintenance.

---
