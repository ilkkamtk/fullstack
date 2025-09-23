# Flask (Python)

<https://flask.palletsprojects.com/>

> A lightweight WSGI web framework that's easy to start and great for building REST APIs.

Flask provides routing, request/response handling, templating (Jinja2), and static file serving while staying minimal and unopinionated.

## Installation and setup

-[Flask installation with VScode and tutorial](https://code.visualstudio.com/docs/python/tutorial-flask)

Minimal Flask app (`app.py`):

```python
from flask import Flask

app = Flask(__name__)

@app.get("/")
def index():
    return "Welcome to my REST API!"

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=3000, debug=True)
```

Run the server:

```bash
python app.py
# or with Flask CLI (auto-reload):
export FLASK_APP=app.py
export FLASK_ENV=development
flask run --host 127.0.0.1 --port 3000
```

## Main features

- Static file server (built-in)
- Routing with decorators (@app.get/@app.post/@app.put/@app.delete)
- Jinja2 template engine (optional)
- Easy JSON responses (return dict or use jsonify)

### Serving static files

By default Flask serves files from a folder named `static` at `/static/...`.

If you want to serve from a folder named `public` mounted at `/public`, configure the app:

```python
from flask import Flask

app = Flask(__name__, static_folder="public", static_url_path="/public")
```

Then place files in `public` and access them at `http://localhost:3000/public/...`.

### Serving JSON (for client-side rendering, CSR)

Flask will JSON-encode a Python dict automatically (Flask 2+):

```python
from flask import Flask

app = Flask(__name__)

@app.get("/api/resource")
def get_resource():
    my_data = {
        "title": "This is an item",
        "description": "Just some dummy data here",
    }
    return my_data
```

Or explicitly use `jsonify`:

```python
from flask import jsonify

@app.get("/api/other")
def other():
    return jsonify(success=True)
```

### Reading request parameters

Using route parameters (path variables, preferred for REST):

```python
from flask import abort

@app.get("/api/resource/<int:item_id>")
def get_item(item_id: int):
    if item_id == 99:
        return {
            "title": f"This is a specific item, id: {item_id}",
            "description": "Just some dummy data here",
        }
    abort(404)
```

Using query parameters:

```python
from flask import request, abort

@app.get("/api/resource")
def get_with_query():
    item_id = request.args.get("id")  # e.g. /api/resource?id=99&name=foo
    if item_id == "99":
        return {
            "title": f"This is a specific item, id: {item_id}",
            "description": "Just some dummy data here",
        }
    abort(404)
```

### Reading JSON request body

```python
from flask import request

@app.post("/api/resource")
def create_resource():
    body = request.get_json(silent=True) or {}
    return {"your_request": body}, 201
```

### Templates (optional)

Place templates in a `templates` folder and render with Jinja2:

```python
from flask import render_template

@app.get("/hello")
def hello_page():
    return render_template("hello.html", name="Flask")
```

## Environment variables

Environment variables keep secrets and config out of code. With `python-dotenv`, variables in a `.env` file are loaded automatically when using the Flask CLI, or you can load them manually:

```python
from dotenv import load_dotenv
load_dotenv()  # reads .env into environment
```

Example `.env`:

```dotenv
FLASK_APP=app.py
FLASK_ENV=development
SECRET_KEY=change-me
```

Access in code via `os.getenv("SECRET_KEY")`.

---

## Assignment

In the labs we are going to build a REST API with Flask. The API will serve JSON data and static files. [Example API documentation here](https://media.edu.metropolia.fi/cats/docs/) (Metropolia network / VPN only).

1. Create a new project folder for this week's assignments and initialize a new Git repository.
2. Add a `.gitignore` file. Exclude `.venv/`, `__pycache__/`, and `.env`. You can use [gitignore.io](https://www.toptal.com/developers/gitignore) to generate Python ignores.
3. Create a new branch `Assignment1` and switch to it: `git checkout -b Assignment1`.
4. Commit your changes regularly and push to the remote.
5. Optional but recommended: set up Python code style tools (e.g., Black, Ruff) and EditorConfig.
6. Create a new Flask project based on the "Hello world" example above.
   - create and activate a virtual environment
   - install dependencies: `python -m pip install Flask python-dotenv`
   - create `app.py` and add the minimal Flask code
   - run with auto-reload using the Flask CLI: `flask run --host 127.0.0.1 --port 3000`
7. Create an endpoint that returns a cat object in JSON at `GET /api/v1/cat` with properties:
   - `cat_id`: number
   - `name`: string
   - `birthdate`: string
   - `weight`: number
   - `owner`: number
   - `image`: string, URL to an image (e.g., `https://place-hold.it/320x240&text=Cat`)
     Assign any values you like.
8. Test the application:
   - run the server
   - open a browser and navigate to `http://localhost:3000/api/v1/cat`
9. Create a folder `public` and add an image file. Serve it by configuring Flask with `static_folder="public"` and `static_url_path="/public"` and verify at `http://localhost:3000/public/your-image.jpg`.
10. Freeze dependencies: `pip freeze > requirements.txt`.
11. Commit and push branch `Assignment1` to the remote repository.
12. Merge `Assignment1` into `main` and push `main`.
13. Deploy the project to a server (e.g., [Azure](https://www.youtube.com/playlist?list=PLKenVLUxjmH_1obN-sz7KvOcBHbRuTdiO), [Metropolia ecloud](./ecloud-installation.md) and test that it works. For production, consider running with a WSGI server like Gunicorn behind a reverse proxy.
