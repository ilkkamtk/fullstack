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
DB_NAME=animals
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

E.g. as a module (`models/blog.py`):

```python
from mongoengine import Document, StringField, DateTimeField, BooleanField, ListField, EmbeddedDocument, EmbeddedDocumentField, IntField
import datetime

class Comment(EmbeddedDocument):
    body = StringField()
    date = DateTimeField(default=datetime.datetime.utcnow)
    author = StringField()

class Blog(Document):
    title = StringField(required=True)
    author = StringField()
    body = StringField()
    comments = ListField(EmbeddedDocumentField(Comment))
    date = DateTimeField(default=datetime.datetime.utcnow)
    hidden = BooleanField(default=False)
    votes = IntField()
    favs = IntField()
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
  - `GeoPointField`
  - etc.

- Built-in validation:

  - `required=True`
  - `min_length`, `max_length`
  - `choices` (like `enum`)
  - `regex`
  - `min_value`, `max_value`
  - Custom validation with `clean()` method.

---

## Example collections with references

Create Python classes for **Category**, **Species**, and **Animal** using `mongoengine.Document`.

### Category

```python
from mongoengine import Document, StringField

class Category(Document):
    category_name = StringField(required=True, unique=True, min_length=2)
```

### Species

```python
from mongoengine import Document, StringField, ReferenceField, PointField

class Species(Document):
    species_name = StringField(required=True, unique=True, min_length=2)
    image = StringField(required=True)  # can validate URL in clean()
    category = ReferenceField("Category", required=True)
    location = PointField(required=True)  # GeoJSON point
```

### Animal

```python
from mongoengine import Document, StringField, DateTimeField, ReferenceField, PointField, ValidationError
import datetime

class Animal(Document):
    animal_name = StringField(required=True, min_length=2)
    birthdate = DateTimeField(required=True)
    species = ReferenceField("Species", required=True)
    location = PointField(required=True)

    def clean(self):
        if self.birthdate > datetime.datetime.utcnow():
            raise ValidationError("Birthdate cannot be in the future")
```

---

## Exposing models via controllers/routes

Example: `routes/category_routes.py`

```python
from flask import Blueprint, request
from models.category import Category

category_bp = Blueprint("category", __name__)

@category_bp.route("/", methods=["GET"])
def get_all_categories():
    return {"categories": Category.objects()}

@category_bp.route("/<id>", methods=["GET"])
def get_category(id):
    return {"category": Category.objects.get_or_404(id=id)}

@category_bp.route("/", methods=["POST"])
def create_category():
    data = request.get_json()
    category = Category(**data).save()
    return {"category": category}, 201

@category_bp.route("/<id>", methods=["PUT"])
def update_category(id):
    data = request.get_json()
    Category.objects(id=id).update_one(**data)
    return {"category": Category.objects.get(id=id)}

@category_bp.route("/<id>", methods=["DELETE"])
def delete_category(id):
    Category.objects(id=id).delete()
    return {"deleted": id}
```

(similar for species and animals)

Special endpoint for animals in a bounding box:

```python
@animal_bp.route("/location", methods=["GET"])
def get_animals_in_box():
    top_right = request.args.get("topRight").split(",")
    bottom_left = request.args.get("bottomLeft").split(",")
    box = [ [float(bottom_left[1]), float(bottom_left[0])],
            [float(top_right[1]), float(top_right[0])] ]
    animals = Animal.objects(location__within_box=box)
    return jsonify(animals)
```

---

## Class Methods

In MongoEngine, you can add **class methods** to models.

### Example: Find Animals by Species

```python
class Animal(Document):
    # fields...

    @classmethod
    def find_by_species(cls, species_name: str):
        return cls.objects(species__species_name=species_name)
```

Endpoint:

```python
@animal_bp.route("/species/<species>", methods=["GET"])
def get_by_species(species):
    animals = Animal.find_by_species(species)
    return jsonify(animals)
```

### Species in an Area (Polygon)

```python
class Species(Document):
    # fields...

    @classmethod
    def find_by_area(cls, polygon):
        return cls.objects(location__geo_within={"$polygon": polygon})
```

Endpoint:

```python
@species_bp.route("/area", methods=["POST"])
def species_in_area():
    polygon = request.get_json()["coordinates"][0]
    species = Species.find_by_area(polygon)
    return jsonify(species)
```

---

## Selecting & Populating

- In MongoEngine:

  - `.only("field1", "field2")` to select fields.
  - `.select_related()` to populate reference fields.

Example:

```python
animals = Animal.objects().select_related().only("animal_name", "species__species_name", "species__category__category_name")
```

- Aggregations are supported with `.aggregate()` for complex joins.

---
