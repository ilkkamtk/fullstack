# Validation with MongoEngine

When using MongoEngine, you can leverage its [built-in validation features](https://docs.mongoengine.org/guide/validation.html) to ensure that your data meets specific criteria before being saved to the database. This helps maintain data integrity and prevents invalid data from being stored.

## Defining Validation Rules

You can define validation rules directly in your document schema by using field options and custom validation methods. For example:

```python
from mongoengine import Document, StringField, IntField, ValidationError
class User(Document):
    username = StringField(required=True, min_length=3, max_length=50)
    email = StringField(required=True, unique=True)
    age = IntField(min_value=0)

    def clean(self):
        # Custom validation logic
        if "@" not in self.email:
            raise ValidationError("Invalid email address")
```

In this example:

- The `username` field is required and must be between 3 and 50 characters long.
- The `email` field is required and must be unique.
- The `age` field must be a non-negative integer.
- The `clean` method provides custom validation to ensure the email contains an "@" symbol.

## Handling Validation Errors

When you attempt to save a document that violates the defined validation rules, MongoEngine raises a `ValidationError`. You can catch this exception to handle validation errors gracefully:

```python
try:
    user = User(username="jsmith", email="jsmithexample.com", age=-5)
    user.save()
except ValidationError as e:
    print("Validation Error:", e)
```

In this example, attempting to save a user with an invalid email and negative age will raise a `ValidationError`, which you can catch and process accordingly.

## Advanced Validation

You can also create more complex validation logic by defining custom validation functions for specific fields:

```python
import bcrypt
from mongoengine import Document, StringField, ValidationError, signals

class User(Document):
    email = StringField(required=True, unique=True)
    password = StringField(required=True, validation=validate_password)

    @staticmethod
    def validate_password(password):
        if len(password) < 8:
            raise ValidationError("Password must be at least 8 characters long.")
        if not any(char.isupper() for char in password):
            raise ValidationError("Password must contain at least one uppercase letter.")
        if not any(char.islower() for char in password):
            raise ValidationError("Password must contain at least one lowercase letter.")
        if not any(char.isdigit() for char in password):
            raise ValidationError("Password must contain at least one digit.")

# -----------------------------
# Signal handler for pre-save
# -----------------------------
def hash_password(sender, document, **kwargs):
    """
    Automatically validates and hashes the password before saving.
    Only hashes if the password is not already hashed. Checks if it starts with '$2b$' because save() runs also on updates.
    """
    if not document.password.startswith('$2b$'):
        # Validate plain-text password
        User.validate_password(document.password)
        # Hash the password
        salt = bcrypt.gensalt()
        document.password = bcrypt.hashpw(document.password.encode('utf-8'), salt).decode('utf-8')


# Connect the signal to the User model
signals.pre_save.connect(hash_password, sender=User)
```

**Controller Example:**

```python
def create_user():
    data = request.get_json()
    try:
        user = User(**data)  # password is plain-text here
        user.save()           # signal automatically validates and hashes password
    except ValidationError as e:
        return jsonify({"message": str(e)}), 400
    except Exception as e:
        return jsonify({"message": "Failed to create user"}), 500

    return jsonify(user.to_json()), 201
```

In this example, the `validate_password` function checks that the password meets specific criteria. The `hash_password` function is connected to the `pre_save` signal to ensure that passwords are validated and hashed before being saved to the database.

[Signals](https://docs.mongoengine.org/guide/signals.html) are a feature in MongoEngine that allow you to hook into the lifecycle of documents, enabling you to perform actions such as validation and transformation automatically. To use signals, install the `blinker` library (no import needed in the code):

```bash
pip install blinker
```

## Assignment

1. Continue your existing Flask app and create a branch `validation`.
2. Implement server-side validation for your MongoEngine models using built-in field options and custom validation methods.
   - Define appropriate validation rules for each field in your models:
     - Users:
       - username: StringField, required, unique, min length 3, max length 50
       - email: StringField, required, unique, valid email format
       - password: StringField, required, min length 8, must contain at least one uppercase letter, one lowercase letter, and one digit
       - role: StringField, required, must be either 'user' or 'admin'
       - created_at: DateTimeField, automatically set to the current date and time when the user is created
     - Cats:
       - cat_name: StringField, required, min length 2, max length 30
       - birthdate: DateField, required, must be a valid date in the past
       - weight: FloatField, required
       - location: PointField, required
       - owner: ReferenceField to User, required
       - attributes: DictField, optional, keys and values must be strings
3. Ensure that your Flask routes handle `ValidationError` exceptions appropriately, returning meaningful error messages to the client.
4. Test your validation logic by attempting to create and update documents with both valid and invalid data.
5. Add endpoints `/api/v1/users/check/:email` and `/api/v1/users/check/:username` to check if an email or username is already in use. Enpoints should return JSON like `{ "exists": true }` or `{ "exists": false }`.
6. Commit and push your changes to the `validation` branch.
7. Create a pull request to merge the `validation` branch into `main` and request feedback from Copilot in GitHub.
