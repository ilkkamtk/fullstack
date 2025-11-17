# MongoEngine

- ["MongoEngine – Object-Document Mapper for MongoDB"](https://docs.mongoengine.org/)
- Provides a schema-based solution to model data with MongoDB in Python.
- Built-in type casting, validation, query building, signals (hooks), etc.
- Methods return querysets or objects that can be used synchronously with Flask.

Install mongoengine dependency with command `pip install mongoengine` and the update the requirements.txt with `pip list --not-required --format freeze > requirements.txt`

---

## Connecting

For example as a separated module (`utils/db.py`):

```python
from mongoengine import connect
import os

def mongo_connect():
    try:
        connection = connect(
            db=os.getenv("DB_NAME"),
            host=os.getenv("DATABASE_URL")
        )
        print("DB connected successfully")
        return connection
    except Exception as e:
        print("Connection to db failed:", str(e))
```

Then in `.env` file (make sure to have it in `.gitignore`):

```dotenv
DATABASE_URL=mongodb+srv://<username>:<password>@<host>/
DB_NAME=some_database
PORT=3000
```

And in `app.py`:

```python
import os
from flask import Flask
from werkzeug.middleware.proxy_fix import ProxyFix
from dotenv import load_dotenv
from api.v1.cats.cats_routes import cats_bp
from api.v1.users.users_routes import users_bp
from api.utils.db import mongo_connect

load_dotenv()

app = Flask(__name__)

app.wsgi_app = ProxyFix(app.wsgi_app, x_proto=1, x_prefix=1)

app.register_blueprint(cats_bp)
app.register_blueprint(users_bp)

@app.get("/")
def index():
    return "Welcome to my REST API! Api endpoints can be found under /api/v1"


if __name__ == "__main__":
    # Establish DB connection
    mongo_connect()

    app.run(host="127.0.0.1", port=os.getenv("PORT"), debug=os.getenv("FLASK_DEBUG"), use_reloader=os.getenv("FLASK_RELOADER"))
```

- MongoEngine automatically reconnects if the connection drops.
- Format for URL is the same as in MongoDB Atlas.

---

## Model Schema

In MongoEngine, model schemas are Python classes that inherit from `Document` (or `EmbeddedDocument`).

An embedded document is a document stored inside another document, not in its own collection.

E.g. as a module (`models/blog.py`):

```python
from mongoengine import Document, StringField, DateTimeField, BooleanField, ListField, EmbeddedDocument, EmbeddedDocumentField, IntField
import datetime

class Comment(EmbeddedDocument):
    body = StringField()
    # use a callable so default is evaluated per-document
    date = DateTimeField(default=lambda: datetime.datetime.now(tz=datetime.timezone.utc))
    author = StringField()

class Blog(Document):
    title = StringField(required=True)
    author = StringField()
    body = StringField()
    comments = ListField(EmbeddedDocumentField(Comment))
    # use a callable to avoid import-time evaluation
    date = DateTimeField(default=lambda: datetime.datetime.now(tz=datetime.timezone.utc))
    hidden = BooleanField(default=False)
    votes = IntField()
    favs = IntField()
```

- Lambda callables are used here to ensure that the default value is generated at the time of document creation, not at the time of class definition. Without lambda, all documents would get the same timestamp.

Blog post is created like this:

```python
from models.blog import Blog, Comment

new_post = Blog(
    title="My first blog post",
    author="John Doe",
    body="This is the content of the blog post.",
    comments=[
        Comment(body="Great post!", author="Alice"),
        Comment(body="Thanks for sharing.", author="Bob")
    ],
    votes=10,
    favs=5
)

new_post.save()
```

---

## Field Types

- MongoEngine supports fields like:

  - `StringField`
  - `IntField`
  - `DateTimeField`
  - `BooleanField`
  - `ObjectIdField`
  - `ReferenceField`
  - `ListField`
  - `EmbeddedDocumentField`
  - `PointField` (GeoJSON)
  - etc.

- Built-in validation:

  - `required=True`
  - `min_length`, `max_length`
  - `choices` (like `enum`)
  - `regex`
  - `min_value`, `max_value`
  - Custom validation with `clean()` method.

Note: Validation and more robust serialization are covered in later lectures.

- Full list of field types and options: [MongoEngine Fields Documentation](https://docs.mongoengine.org/guide/defining-documents.html#fields)

---

### CRUD Operations

- Create: `document.save()`
- Read: `Model.objects()`, `Model.objects(field=value)`, `Model.objects.get(id=value)`
- Update: Modify fields and call `save()`, or use `update_one()`, `update_many()`
- Delete: `document.delete()`, or `Model.objects(field=value).delete()`

Example:

```python
# Create
new_blog = Blog(title="New Post", body="Content")
new_blog.save()

# Read all blogs
all_blogs = Blog.objects()

# Read by id
first_blog = Blog.objects.get(id="some_id")

# Read with filter
blogs_by_author = Blog.objects(author="John Doe")

# Read and return only specific fields
blogs_titles = Blog.objects.only("title", "date")

# Update
### Read first
first_blog = Blog.objects.get(id="some_id")
### Modify and save
first_blog.title = "Updated Title"
first_blog.save()

# Delete by id
blog_to_delete = Blog.objects.get(id="some_id")
blog_to_delete.delete()
```

---

## Collections with references

Reference fields create relationships between documents in different collections like foreign keys in SQL.

Create Python classes for **Cat**, and **User** using `mongoengine.Document`.

### User

```python
from mongoengine import Document, StringField

class User(Document):
    username = StringField(required=True, unique=True, min_length=2)
    email = StringField(required=True, unique=True)
    password = StringField(required=True)
```

### Cat

```python
from mongoengine import Document, StringField, DateTimeField, ReferenceField, PointField, ValidationError, FloatField, DictField
from api.models.users_model import User

class Cat(Document):
    cat_name = StringField(required=True, min_length=2)
    birthdate = DateTimeField(required=True)
    weight = FloatField(required=True)
    location = PointField(required=True)
    owner = ReferenceField(User, required=True)
    attributes = DictField()
```

- Note: GeoJSON coordinates are [lng, lat].

---

## Exposing models via controllers/routes

```python
# imports...

cats_bp = Blueprint("cats", __name__)
@cats_bp.route("/", methods=["GET"])
def get_all_cats():
    cats = Cat.objects()
    return cats.to_json(), 200

### Example: Create a new Cat (assuming that the JSON body matches the Cat schema):
@cats_bp.route("/", methods=["POST"])
def create_cat():
    data = request.get_json()
    new_cat = Cat(**data)
    new_cat.save()
    return {"message": "Cat created successfully"}, 201

# example JSON body:
"""
{
  "cat_name": "Whiskers",
  "birthdate": "2020-01-01",
  "weight": 4.5,
  "location": {"type": "Point", "coordinates": [24.76, 60.22]},
  "owner": "some_user_id",
  "attributes": {"color": "tabby", "indoor": true}
}
"""
```

- Note `**data` passes the dictionary’s contents as parameters to the `Cat` constructor. It is equivalent to `Cat(cat_name=data["cat_name"], birthdate=data["birthdate"], ...)`.
- Calling `to_json()` on the queryset or document converts it to JSON string but it may not be ideal for complex nested documents or references. See Marshmallow section below for better serialization. Also mime type is currently not set to `application/json`, which is also corrected when using schemas.

---

## Class Methods

In MongoEngine, you can add **class methods** to models. Class methods are methods that are bound to the class and not the instance of the class. They can be used to define custom queries or operations related to the model. They always receive the class (`cls`) as the first argument instead of the instance (`self`). `cls` is used to access class-level attributes and methods like querying the database.

### Example: Find Cats by Owner

```python
class Cat(Document):
    # fields...

    @classmethod
    def find_by_owner(cls, owner_id):
        return cls.objects(owner=owner_id)
```

Endpoint:

```python
@cat_bp.route("/owner/<owner_id>", methods=["GET"])
def get_by_owner(owner_id):
    cats = Cat.find_by_owner(owner_id)
    return cats.to_json(), 200
```

### Cats in an Area (Polygon)

```python
class Cat(Document):
    # fields...

    @classmethod
    def find_by_area(cls, geometry):
        return cls.objects(location__geo_within={"$geometry": geometry})
```

Endpoint:

```python
@cats_bp.route("/area", methods=["POST"])
def get_cats_in_area():
    geometry = request.get_json()
    cats = Cat.find_by_area(geometry)
    return cats.to_json(), 200
```

Example request body:

```json
{
  "coordinates": [
    [
      [24.73091877995617, 60.24583005429855],
      [24.73091877995617, 60.21004096366292],
      [24.791063894993783, 60.21004096366292],
      [24.791063894993783, 60.24583005429855],
      [24.73091877995617, 60.24583005429855]
    ]
  ],
  "type": "Polygon"
}
```

---

## Static Methods

Static methods do not receive an implicit first argument (neither `self` nor `cls`). They behave like regular functions but belong to the class's namespace. They are defined using the `@staticmethod` decorator. They are useful for utility functions that have some logical connection to the class but do not need to access class or instance data. For example, a static method to check if a user exists by email:

```python
class User(Document):
    # fields...
    @staticmethod
    def email_exists(email):
        return User.objects(email=email).first() is not None
```

---

## Assignment

1. Create a new branch Assignment5 from main. Make sure you have merged previous assignments.
2. Create a database connection module as shown above to `utils/db.py`.
3. Create User and Cat models as shown above in `api/models/users_model.py` and `api/v1/cats/cats_model.py`. Replace models from previous assignments with these.
   - Cat model should have fields:
     - cat_name, string, required
     - birthdate (datetime, required),
     - weight (number, required),
     - location (PointField, required),
     - owner (Reference to User, required),
     - image (string, required),
     - attributes (dict, optional).
   - User model should have fields:
     - username (string, required, unique)
     - email (string, required, unique)
     - password (string, required)
     - role (string, required, default "user", choices: "user", "admin").
     - created_at (datetime, default now).
       - created_at field should use a callable, e.g. `default=lambda: datetime.datetime.now(tz=datetime.timezone.utc)`.
4. Modify controllers and routes of Users and Cats to use the updated models if needed.
5. Test the endpoints with Postman or similar tool.
   - Create a new user.
   - Create a new cat with reference to the created user.
   - List all cats.
   - Get a cat by id.
   - Modify a cat.
   - Delete a cat.
   - Get cats by owner id.
   - Get cats in an area (polygon).
   - Do the same for users, exept for area search.
   - Create additional cats and owners as needed.
6. JSON returned from GET endpoints are not plain dicts/lists, so you need to use Marshmallow for serialization. Create Marshmallow schemas for User and Cat in `api/schemas/user_schema.py` and `api/v1/cats/cat_schema.py`.

---

## Marshmallow

- [Marshmallow](https://marshmallow.readthedocs.io/en/stable/) is a popular library for object serialization/deserialization and validation in Python.
- Can be used with MongoEngine models to serialize data for APIs.
- Results from MongoEngine queries are not plain dicts or lists, so Marshmallow helps convert them to JSON-serializable formats.
- Use `marshmallow-mongoengine` extension for easier integration.

### Example Schema

- User schema (note: without password):

```python
from marshmallow_mongoengine import ModelSchema
from api.models.users_model import User

class UserSchema(ModelSchema):
    class Meta:
        model = User
        fields = ("id", "username", "email")
```

- Cat schema (including nested owner):

```python
from marshmallow_mongoengine import ModelSchema, fields
from api.v1.users.users_schema import UserSchema
from api.v1.cats.cats_model import Cat

class CatSchema(ModelSchema):
    class Meta:
        model = Cat
        # Automatically include all model fields
        # You can also list specific fields if you want: fields = ("id", "cat_name", "weight", ...)
        # exclude = ("some_field", )  # optional
    # Custom or overridden fields
    birthdate = fields.DateTime(format="iso")
    owner = fields.Nested(UserSchema)
    location = fields.Dict()
```

### Using Schemas in Controllers

```python
# imports...

cat_bp = Blueprint("cats", __name__)

@cat_bp.route("/", methods=["GET"])
def get_all_cats():
    cats = Cat.objects.select_related()  # auto-populates ReferenceFields
    cats_json = CatSchema(many=True).dump(cats)
    return cats_json, 200

@cat_bp.route("/<cat_id>", methods=["GET"])
def get_cat_by_id(cat_id):
    cat = Cat.objects.get(id=cat_id)
    cat_json = CatSchema().dump(cat)
    return cat_json, 200
```

- Note the use of `many=True` when serializing a list of objects. When serializing a single object it is not needed.
- The `dump()` method converts the model instances to JSON-serializable dicts. This is the reason we use marshmallow here.

## Populating

Populating means fetching related documents referenced by `ReferenceField`s. It is similar to SQL joins.

- In MongoEngine:

  - `.only("field1", "field2")` to select fields.
  - `.select_related()` to populate reference fields.

Example:

```python
cats = Cat.objects().select_related().only("cat_name", "owner__username")
```

- Note the double underscore `__` to access fields of the referenced document.

## Reverse lookup

- Reverse lookup means finding documents that reference a specific document.
- To find all Cats owned by a specific User, you can use the `ReferenceField` in reverse. In `Cat`, the `owner` field references `User`. To find all Cats for a User, you can query `Cat` with the User instance.

### Find Cats by Owner

Model:

```python
class User(Document):
    # fields...

   @property
   def cats(self):
        from api.v1.cats.cats_model import Cat
        return Cat.objects(owner=self)
```

Schema:

```python
from marshmallow_mongoengine import ModelSchema, fields
from api.models.users_model import User

class UserSchema(ModelSchema):
    cats = fields.Nested('api.v1.cats.cats_schema.CatSchema', many=True)

    class Meta:
        model = User
        fields = ("id", "username", "email", "cats")
```

- The `fields.Nested` allows for the inclusion of related documents in the serialized output. This means when you serialize a User, it will also include a list of Cats owned by that User. Import cannot be in the top of the file to avoid circular import issues.
- The same needs to be applied in CatSchema for User:

```python
# remove top import of UserSchema

# in class CatSchema(ModelSchema):
    owner = fields.Nested('api.v1.users.users_schema.UserSchema', exclude=('cats',))
```

- Note the use of `exclude=('cats',)` to avoid infinite recursion when serializing because now a user has cats and each cat has an owner.

### Overfetching / Underfetching

- Overfetching: Retrieving more data than needed. E.g., fetching all fields of a document when only a few are required.
- Underfetching: Not retrieving enough data, leading to additional queries later. E.g., fetching a document without its related documents, requiring separate queries to get them.
- It can be discussed whether cats should include full owner data and vice versa. Depending on use case, you might want to limit the fields fetched or avoid nesting altogether.

## Assignment continued

1. Modify the controllers to use the schemas for serialization as shown above.
2. If owner field in Cat returns only the ObjectId. Modify the GET endpoints to populate the owner field using `select_related()` as shown above.
3. Extend the User model with a property method `cats` that returns all Cats owned by the User.
4. Extend the UserSchema to include the nested cats owned by the user.
5. Create a new endpoint in Users controller to get a user by id along with their cats.
6. Test the endpoint to ensure it returns the user data along with their cats.
7. When done, merge the changes to main or create a pull request to main branch for review.
