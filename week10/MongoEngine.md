# MongoEngine

- ["MongoEngine – Object-Document Mapper for MongoDB"](https://docs.mongoengine.org/)
- Provides a schema-based solution to model data with MongoDB in Python.
- Built-in type casting, validation, query building, signals (hooks), etc.
- Methods return querysets or objects that can be used synchronously with Flask.

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
from flask import Flask
from dotenv import load_dotenv
import os
from utils.db import mongo_connect

load_dotenv()

app = Flask(__name__)
port = int(os.getenv("PORT", 3000))

mongo_connect()

@app.route("/")
def home():
    return {"message": "Flask + MongoEngine API running"}

if __name__ == "__main__":
    app.run(port=port, debug=True)
```

- MongoEngine automatically reconnects if the connection drops.
- Format for URL is the same as in MongoDB Atlas.

---

## Schema

In MongoEngine, schemas are Python classes that inherit from `Document` (or `EmbeddedDocument`).

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

## Schema Types

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
import datetime

class Cat(Document):
    cat_name = StringField(required=True, min_length=2)
    birthdate = DateTimeField(required=True)
    weight = FloatField(required=True)
    location = PointField(required=True)
    owner = ReferenceField("User", required=True)
    attributes = DictField()
```

- Note: GeoJSON coordinates are [lng, lat].

---

## Exposing models via controllers/routes

### Example: Get all Cats

```python
# imports...

cats_bp = Blueprint("cats", __name__)
@cats_bp.route("/", methods=["GET"])
def get_all_cats():
    cats = Cat.objects()
    return jsonify(cats), 200

### Example: Create a new Cat (assuming that the JSON body matches the Cat schema):
@cats_bp.route("/", methods=["POST"])
def create_cat():
    data = request.get_json()
    new_cat = Cat(**data)
    new_cat.save()
    return jsonify(new_cat), 201

```

- Note `**data` passes the dictionary’s contents as parameters to the `Cat` constructor. It is equivalent to `Cat(cat_name=data["cat_name"], birthdate=data["birthdate"], ...)`.

---

## Class Methods

In MongoEngine, you can add **class methods** to models.

### Example: Find Cats by Owner

```python
class Cat(Document):
    # fields...

    @classmethod
    def find_by_owner(cls, owner_id: str):
        return cls.objects(owner=owner_id)
```

Endpoint:

```python
@cat_bp.route("/owner/<owner_id>", methods=["GET"])
def get_by_owner(owner_id):
    cats = Cat.find_by_owner(owner_id)
    return jsonify(cats)
```

### Cats in an Area (Polygon)

```python
class Cat(Document):
    # fields...

    @classmethod
    def find_by_area(cls, polygon):
        return cls.objects(location__geo_within={"$polygon": polygon})
```

Endpoint:

```python
@cats_bp.route("/area", methods=["POST"])
def cats_in_area():
    polygon = request.get_json()["coordinates"][0]
    cats = Cat.find_by_area(polygon)
    return jsonify(cats)
```

Example request body:

```json
{
  "type": "Polygon",
  "coordinates": [
    [
      [12.0, 34.0],
      [56.0, 78.0],
      [90.0, 12.0],
      [12.0, 34.0]
    ]
  ]
}
```

---

## Selecting & Populating

Populating means fetching related documents referenced by `ReferenceField`s. It is similar to SQL joins.

- In MongoEngine:

  - `.only("field1", "field2")` to select fields.
  - `.select_related()` to populate reference fields.

Example:

```python
cats = Cat.objects().select_related().only("cat_name", "owner__username")
```

- Note the double underscore `__` to access fields of the referenced document.

## Aggregations

Aggregations are supported with `.aggregate()` for complex joins.

### Aggregation example

You can run a MongoDB aggregation pipeline via the underlying pymongo collection returned by MongoEngine:

```python
#import models...

pipeline = [
    {
      "$lookup": {
        # ensure this 'from' matches the actual collection name
        "from": User._get_collection().name,
        "localField": "owner",
        "foreignField": "_id",
        "as": "owner_info",
      },
    },
    {"$unwind": "$owner_info"},
    {
      "$project": {
        "id": 1,
        "cat_name": 1,
        "owner_name": "$owner_info.username",
      },
    },
]

# Run the pipeline and get results as a list of dicts
cursor = Cat._get_collection().aggregate(pipeline)
results = list(cursor)
# results -> [{'id': ..., 'cat_name': ..., 'owner_name': ...}, ...]
```

## Marshmallow

- [Marshmallow](https://marshmallow.readthedocs.io/en/stable/) is a popular library for object serialization/deserialization and validation in Python.
- Can be used with MongoEngine models to serialize data for APIs.
- Results from MongoEngine queries are not plain dicts or lists, so Marshmallow helps convert them to JSON-serializable formats.

### Example Schema

- User schema (note: without password):

```python
from marshmallow_mongoengine import ModelSchema, fields

class UserSchema(ModelSchema):
    class Meta:
        model = User
        fields = ("id", "username", "email")
```

- Cat schema (including nested owner):

```python
from marshmallow_mongoengine import ModelSchema, fields
from api.v1.users.user_schema import UserSchema

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

## Assignment

1. Create a new branch Assignment5 from main. Make sure you have merged previous assignments.
2. Create a database connection module as shown above to `utils/db.py`.
3. Create User and Cat models as shown above in `api/models/users_model.py` and `api/v1/cats/cats_model.py`. Replace models from previous assignments with these.
   - Cat model should have fields: cat_name (string), birthdate (datetime), weight (number), location (PointField), owner (Reference to User), attributes (dict).
   - User model should have fields: username (string), email (string), password (string), role (string, default "user"), created_at (datetime, default now).
     - created_at field should use a callable, e.g. `default=lambda: datetime.datetime.now(tz=datetime.timezone.utc)`.
4. Modify controllers of Users and Cats to use the updated models if needed.
5. Test the endpoints with Postman or similar tool.
   - Create a new user.
   - Create a new cat with reference to the created user.
   - List all cats.
   - Get a cat by id.
   - Modify a cat.
   - Delete a cat.
   - Create additional cats and owners and test querying by area.
   - Do the same for users
6. If owner field in Cat returns only the ObjectId. Modify the GET endpoints to populate the owner field using `select_related()` as shown above.
7. Create Marshmallow schemas for User and Cat in `api/schemas/user_schema.py` and `api/v1/cats/cat_schema.py`.
8. Modify the controllers to use the schemas for serialization as shown above.

---

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
        from ..api.v1.cats.cats_model import Cat
        return Cat.objects(owner=self)
```

Schema:

```python
from marshmallow import ModelSchema, fields

class UserSchema(ModelSchema):
    # use a lazy callable to avoid importing CatSchema at module import time
    cats = fields.Nested(lambda: CatSchema(), many=True)

    class Meta:
        model = User
        fields = ("id", "username", "email", "cats")
```

- Lazy callable is a technique to avoid circular imports by delaying the import of `CatSchema` until it is actually needed. How does python find CatSchema even its not imported at the top of the file? Because the lambda function is called only when the schema is being used, not when the module is imported. Avoiding importing CatSchema at the top prevents circular import issues.

Controller:

```python
def get_user_cats(user_id):
    user = User.objects.get(id=user_id)
    cats = user.cats
    return jsonify(cats)
```

## Assignment continued

1. Extend the User model with a property method `cats` that returns all Cats owned by the User.
2. Extend the UserSchema to include the nested cats owned by the user.
3. Create a new endpoint in Users controller to get a user by id along with their cats.
4. Test the endpoint to ensure it returns the user data along with their cats.
5. When done, merge the changes to main or create a pull request to main branch for review.
