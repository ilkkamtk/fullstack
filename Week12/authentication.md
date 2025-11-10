# Application State and User Authentication

## Maintaining State in Web Applications

In any realistic interactive Web application you will come across the need of retaining information between individual pages. This is referred to as "maintaining state" or as the "persistence of data". HTTP protocol does not support this — it is a _stateless_ protocol: every page request starts in a blank state with no knowledge of data that was available on the previous page. Since HTTP is stateless by nature, web applications need to implement their own methods for maintaining state. There are several common strategies to implement this on the server-side.
Choosing the right method for maintaining state depends on the specific requirements of your application, such as the type of data being stored, security considerations, scalability, and whether the state is user-specific or shared across users. It"s common to use a combination of these methods to address different aspects of application state.

### Cookies

- Small pieces of data stored on the client"s browser. Cookies are sent back to the server with every HTTP request.
- Cookies store variables as name and value pairs, plus additional information such as expiration time and the origin (domain) where the cookie came from.
- By default a website can modify only its own cookies
- Use Cases: Remembering user preferences, authentication tokens, tracking sessions.
- Limitations: Size is limited (typically 4KB), and they can pose security and privacy concerns if not handled correctly.

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: HTTP Request (initial visit)
    Note over Server: Server decides to set a cookie
    Server->>Client: HTTP Response with Set-Cookie header
    Note over Client: Client stores the cookie
    Client->>Server: Subsequent HTTP Request with cookie
    Note over Server: Server reads cookie and personalizes response
    Server->>Client: Personalized HTTP Response
```

### Server-Side Sessions

- Server stores session data, and a session identifier is typically sent to the client (often in a cookie or in HTTP headers/URL query params).
- Can be used for user authentication and storing for user-specific data while the user navigates the application.
- Requires server resources and proper management to ensure scalability and security.

### URL Parameters and Query Strings

- Passing state information in URLs as query strings. For example: `https://example.com/app?user=johndoe&theme=dark`.
- Maintaining state across different pages without needing server or client-side storage, such as filtering or search parameters.
- Visible to users, can be modified, and have length limitations.

### Hidden Form Fields

- Storing state information in hidden fields within HTML forms. For example:

  ```html
  <form action="https://example.com/app">
    <input type="hidden" name="user" value="johndoe" />
    <input type="hidden" name="theme" value="dark" />
    <input type="submit" value="Submit" />
  </form>
  ```

- Maintaining state across form submissions.
- Only applicable for state that needs to persist across form submissions.

### Why Cookies and Sessions Are Not Used in RESTful APIs

While cookies and sessions are useful for traditional web applications, **they are not used in RESTful APIs** due to the following reasons:

1. **Statelessness Principle**: REST architecture is based on the principle of statelessness. Each request from a client to a server must contain all the information needed to understand and process the request. The server should not store any client context between requests. Sessions violate this principle because they require the server to maintain state about the client.

2. **Scalability Issues**: Session-based authentication requires the server to store session data, which creates challenges when scaling horizontally across multiple servers. Each server would need access to the same session store, adding complexity and potential bottlenecks.

3. **Cross-Origin Resource Sharing (CORS)**: Cookies have limitations with CORS, making them less suitable for APIs that need to be accessed from different domains or by mobile applications. Browsers handle cookies differently across domains, which can complicate API usage.

4. **Platform Independence**: RESTful APIs are designed to be consumed by various clients (web browsers, mobile apps, IoT devices, etc.). Cookies are browser-specific and not universally supported across all platforms. Token-based authentication works consistently across all client types.

5. **No Automatic Attachment**: Unlike cookies that are automatically sent with every request by browsers, tokens must be explicitly included in request headers. This gives developers more control and makes the authentication mechanism more transparent and intentional.

**Instead of cookies and sessions, RESTful APIs typically use token-based authentication** (such as JWT) where the client includes the token in the request header (usually in the `Authorization` header). This approach maintains statelessness, supports better scalability, and works across different platforms and devices.

### Token-Based Authentication

- Tokens (e.g. [JSON Web Tokens (JWT)](https://jwt.io/)) are used to maintain user state and authentication information.
  - usually used for authentication and authorization purposes, especially in Single Page Applications (SPAs) and API services.
- Consider security in storing and transporting tokens, typically requires HTTPS.

The process of authentication using JWT and subsequent requests with the token in typical client-server interaction:

```mermaid
sequenceDiagram
    participant Client
    participant Server
  Client->>Server: HTTP Request with Credentials (login)
  Note over Server: Validates credentials and creates/signs JWT
  Server->>Client: HTTP Response with JWT
  Note over Client: Stores JWT (e.g., in localStorage or memory)

  Client->>Server: Subsequent HTTP Request with JWT in header
  Note over Server: Verifies JWT
  alt JWT valid
    Server->>Client: Authorized Response
  else JWT invalid or expired
    Server->>Client: Unauthorized Response
  end
```

### Client-Side State Management

- Web Storage API
  - Local Storage: Stores data with no expiration date, and it"s accessible across browser sessions.
  - Session Storage: Similar to Local Storage but limited to a single session. The data is cleared when the page session ends.
  - Storing user data like settings, application state, and other temporary data that doesn’t need to persist long-term.
  - Data is only accessible on the client side and limited to about 5 MB.
- Client-Side Frameworks and Libraries like React (with Context API or Redux), Angular (with services), Vue.js (with Vuex) provide their own mechanisms for maintaining state on the client side.

[Simple practical frontend example of using JWT for authentication.](authentication-jsexample.md)

---

### Cookies and Sessions

Flask provides built-in support for cookies and sessions:

- **Cookies:**

  - To **read** a cookie, access the `request.cookies` dictionary: `request.cookies.get("cookie_name")`.
  - To **set** a cookie, use the `response` object"s `set_cookie` method before returning the response: `response.set_cookie("cookie_name", "value")`. You"ll need to make the response first, e.g., `response = make_response(render_template(...))`.

- **Sessions:** Flask comes with a built-in, secure **session object** . This is the most common and recommended way to manage state.
  - The session data is stored in a **client-side cookie** that is cryptographically **signed** by the server"s `SECRET_KEY` to prevent tampering.
  - You access the session like a dictionary:
    - To **set** a value: `session["key"] = "value"`
    - To **get** a value: `user_id = session.get("user_id")`
  - **Note:** You must set a `SECRET_KEY` in your application configuration for sessions to work securely.

---

### JWT-Based Authentication

For implementing **JSON Web Tokens (JWTs)**, you use standard Python packages:

- **JWT Generation and Verification:** One popular package for generating and verifying JWT tokens is **`PyJWT`**.
  - You `encode` (create) and `decode` (verify) tokens within your Flask routes.
  - You can use [decorators](decorator.md) to protect routes, extract tokens from headers, and handle refreshing tokens, simplifying the implementation of JWT-based authentication in a Flask application.

## User Authentication and Authorization with JWT

- **Authentication** is the process of verifying the identity of a user. For example, checking if the username and password provided by the user match those stored in the database.
- **Authorization** is the process of verifying that the user has access to the requested resource. For example, checking if the authenticated user has the necessary permissions to access a specific endpoint or perform a certain action.

In web applications, authentication is typically done by verifying a username and password combination. Authorization is typically done by checking the user"s role or permissions for the requested resource.

1. Install required packages

   ```bash
   pip install PyJWT bcrypt
   ```

2. Generate a secret key for signing the tokens and store it in the `.env` file: `JWT_SECRET_KEY=...`

   - Use a long random string.

3. Modify the route for registering new users to hash the password before storing it in the database:

   ```python
    # File: models/users_model.py
    import bcrypt
    # ... other imports ...

    class User:
         # ... init ...

         @staticmethod
         def create_user(user):
              """
              Creates a new user with hashed password.
              """
              # 1. Generate salt and hash the password
              salt = bcrypt.gensalt()
              hashed_password = bcrypt.hashpw(user.password.encode("utf-8"), salt)

              # 2. Create and save the user object with hashed password
              user = User(
                username=user.username,
                password=hashed_password.decode("utf-8"),
                email=user.email
              )
              user.save()
              return user
   ```

   - update the `users_controller.py` to use the new `create_user` method when registering users:

   ```python
   def create_user():
       data = request.get_json()
       user = User(**data)
       User.create_user(user)  # Uses hashed password internally
       return {"message": "user created successfully"}, 201
   ```

4. Create a route `POST /api/v1/auth/login` that accepts a username and password in the request body.

   - Add a route handler to the Blueprint file: `/api/v1/auth/auth_routes.py`.

   - The route handler calls the controller method `post_login`, passing the request data.

   ```python
   from api.v1.auth.auth_controller import post_login

   # File: /api/v1/auth/auth_routes.py
   @auth_bp.route("/login", methods=["POST"])
   def login():
       return post_login()
   ```

5. In the _model_ implement a method for verifying the username and password combination and returning the user object if found:

   ```python
   # File: models/users_model.py
   import bcrypt
   # ... other imports ...

   class User:
       # ... init ...

       @staticmethod
       def verify_credentials(username, password):
           """
           Finds a user by username and verifies the password using bcrypt.
           Returns the User object on success, or None on failure.
           """
           # 1. Look up user by username in the database
           user = User.objects(username=username).first()

           # 2. If user exists, verify password hash using bcrypt
           if user and bcrypt.checkpw(password.encode("utf-8"), user.password.encode("utf-8")):
               return user

           return None
   ```

6. In the controller implement token generation for the logged in user:

   ```python
   # File: /api/v1/auth_controller.py
   from flask import request
   import jwt
   from datetime import datetime, timezone, timedelta
   import os
   from models.users_model import User

   def post_login():
       data = request.get_json()
       # ... credential check ...
       username = data.get("username")
       password = data.get("password")
       user = User.verify_credentials(username, password) # Uses bcrypt check internally

       if user:
           jwt_secret = os.getenv("JWT_SECRET_KEY", "fallback-jwt-secret")

           # 1. Create the payload with expiration time
           payload = {
            "user_id": str(user.id),
            "username": user.username,
            "exp": datetime.now(timezone.utc) + timedelta(hours=24)  # Token expires in 24 hours
           }

           # 2. Encode the token
           token = jwt.encode(payload, jwt_secret, algorithm="HS256")

           # 3. Return the user data and the token
           return {
                "message": "Login successful",
                "user": UserSchema().dump(user),
                "token": token
            }, 200
       else:
           return {"message": "Invalid credentials"}, 401
   ```

   - If the user is found, `jwt.encode()` is used to generate a JWT token. The token is sent back to the client along with the user object.

7. Create a decorator for handling requests to endpoints where authentication is needed in the file `utils/auth_utils.py`.

   - This function, typically named `token_required`, performs token extraction, validation (`jwt.decode()`), user lookup, and passes the user object to the route handler.

   - Use `functools.wraps` to prevent Flask endpoint-name collisions when the same decorator wraps multiple views (e.g. `/cats`, `/users`).

   ```python
   # File: utils/auth_utils.py
   from functools import wraps
   from flask import request, jsonify, current_app
   import jwt
   from jwt import ExpiredSignatureError, InvalidTokenError
   from models.users_model import User

   def token_required(f):
    """Require JWT; inject `current_user` on success."""

        @wraps(f)  # preserve original function metadata
        def decorated(*args, **kwargs):
            auth = request.headers.get("Authorization")

            if not auth:
                return jsonify({"message": "Authorization header is missing"}), 401

            parts = auth.split()

            if parts[0].lower() != "bearer" or len(parts) != 2:
                return jsonify({"message": "Authorization header must be: Bearer <token>"}), 401

            token = parts[1]

            try:
                payload = jwt.decode(token, current_app.config.get("JWT_SECRET_KEY"), algorithms=["HS256"])
            except ExpiredSignatureError:
                return jsonify({"message": "Token has expired"}), 401
            except InvalidTokenError:
                return jsonify({"message": "Invalid token"}), 401

            user = User.objects(id=payload.get("user_id")).first()

            if not user:
                return jsonify({"message": "User not found"}), 401

            return f(current_user=user, *args, **kwargs)

    return decorated
   ```

8. Add a protected route handler that returns the authenticated user"s object (single `/me` example):

   ```python
   # File: blueprints/api/v1/auth_routes.py
   @auth_bp.route("/me", methods=["GET"])
   @token_required
   def me(current_user):
   """Returns the authenticated user."""
   # Controller function should return (payload, status_code)
   response, status_code = get_current_user(current_user)
   return jsonify(response), status_code
   ```

   ```python
   # Controller (example)
   def get_current_user(current_user):
   user = User.objects.get(id=current_user.id)
   return UserSchema().dump(user), 200
   ```

9. Test registering a new user and login and the protected route with VS Code REST Client:

   ```http
    ### Register new user
    POST http://localhost:3000/api/v1/users
    content-type: application/json
    {
      "username": "JohnDoe",
      "password": "12345",
      "email": "johndoe@example.com"
    }
   ```

   ```http
   ### Post login
   POST http://localhost:3000/api/auth/login
   content-type: application/json

   {
     "username": "JohnDoe",
     "password": "12345"
   }
   ```

   - Check the response and copy the token from the response body.

   ```http
   ### Get my user info
   GET http://localhost:3000/api/auth/me
   Authorization: Bearer <put-your-token-from-login-response-here>
   ```

   - or test with Postman (set "Bearer Token" on the Authorization tab after successful login POST).

10. Now you can use the authentication decorator with any protected route.

    - Information about the authenticated user is passed to the route as the parameter `current_user` injected by `@token_required`.

---

## Assignment

1. Continue your existing Flask app and create a branch `authentication`
1. Implement user authentication to your app
   - Add endpoint `POST /api/v1/auth/login`
   - Use JWT for authentication
   - Continue adapting MVC pattern, use Flask Blueprints to modularize your routes for separate endpoints
1. Implement authorization for protected routes, e.g.:
   - `PUT /api/v1/cats/:id` - only file owner can update cats
   - `DELETE /api/v1/cats/:id` - only file owner can delete cats
   - `PUT /api/v1/users/me` - users can update only their own user info
   - and so on...
1. Implement user roles (e.g. admin, user) with different permissions (role based resource authorization)
   - Regular users can only delete and edit their own data and cats, etc.
     - Modify find() in controllers so that queries will also check that the owner of the item (user_id) matches the `user_id` property in the `current_user` parameter.
   - Admin level users can update or delete any cats, user info, etc.
     - Use e.g. conditional statements in the controllers to decide updates and deletes based on the user level.

---
