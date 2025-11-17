# Client-Side Validation

---

## HTML5 Built-in Validation

Modern browsers support HTML5 form validation attributes that can be used to enforce basic validation rules without writing custom JavaScript. Some common attributes include:

- `required`: Ensures the field is not left empty.
- `type`: Specifies the type of input (e.g., `email`, `number`, `url`), which enforces format validation.
- `min` and `max`: Define numeric limits for input fields.
- `pattern`: Allows you to specify a regular expression that the input must match.
  **Example:**

```html
<form id="myForm">
  <label for="email">Email:</label>
  <input type="email" id="email" name="email" required />
  <label for="age">Age:</label>
  <input type="number" id="age" name="age" min="0" max="120" />
</form>
```

## Custom Client-Side Validation

While HTML5 provides basic validation, you may need to implement custom validation logic for more complex requirements. This can be done using JavaScript by listening to form events and checking input values. Common libraries for client-side validation include:

- [**Yup**: A JavaScript schema builder](https://yup-docs.vercel.app/docs/intro) for value parsing and validation.
  - Best Used For: Validating form objects (e.g., an entire user registration payload) where you need to check data type, format, and dependencies.
- [**Validator.js**: A library](https://github.com/validatorjs/validator.js) for string validation and sanitization.
  - Best Used For: Validating individual string inputs (e.g., email addresses, URLs) with a wide range of built-in validators.
  - Simpler and more lightweight compared to Yup for single field validations but for more complex form validations, Yup is often preferred.

---

**Example using Yup:**

```javascript
import * as Yup from 'yup';
const schema = Yup.object().shape({
  email: Yup.string()
    .email('Invalid email format')
    .required('Email is required'),
  password: Yup.string()
    .min(8, 'Password must be at least 8 characters')
    .required('Password is required'),
});

document
  .getElementById('myForm')
  .addEventListener('submit', async function (event) {
    event.preventDefault();
    const formData = {
      email: document.getElementById('email').value,
      password: document.getElementById('password').value,
    };

    try {
      await schema.validate(formData, { abortEarly: false });
      // Proceed with form submission
    } catch (err) {
      // Handle validation errors
      err.inner.forEach((error) => {
        console.log(error.path + ': ' + error.message);
      });
    }
  });
```

---

**Example using Validator.js:**

```html
<script type="text/javascript" src="validator.min.js"></script>
<form id="myForm">
  <label for="email">Email:</label>
  <input type="text" id="email" name="email" />
  <label for="url">Website:</label>
  <input type="text" id="url" name="url" />
  <button type="submit">Submit</button>
</form>

<script type="text/javascript">
  document
    .getElementById('myForm')
    .addEventListener('submit', function (event) {
      event.preventDefault();
      const email = document.getElementById('email').value;
      const url = document.getElementById('url').value;

      if (!validator.isEmail(email)) {
        alert('Invalid email format');
        return;
      }

      if (!validator.isURL(url)) {
        alert('Invalid URL format');
        return;
      }

      // Proceed with form submission
    });
</script>
```

---

## Displaying Validation Errors

When validation fails, it's important to provide clear feedback to users. You can display error messages next to the relevant input fields or in a summary section at the top of the form:

```html
<input type="text" id="email" name="email" />
<div id="emailError" class="hidden error-message"></div>

<script type="text/javascript">
  // Inside your validation logic
  const emailErrorDiv = document.getElementById('emailError');
  emailErrorDiv.innerHTML = ''; // Clear previous errors

  if (!validator.isEmail(email)) {
    emailErrorDiv.textContent = 'Invalid email format';
    emailErrorDiv.classList.remove('hidden');
  } else {
    emailErrorDiv.classList.add('hidden');
  }
</script>
```

```css
.error-message {
  color: red;
  font-size: 0.7rem;
  margin-top: 5px;
}
.hidden {
  display: none;
}
```
