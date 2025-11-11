# File upload in Flask

Handling file uploads is a common requirement in web applications. Flask provides a straightforward way to manage file uploads using the `request` object and the `werkzeug` library.

## Common Example Of Setting Up File Uploads

1. **Install Flask and Werkzeug** (if not already installed):

   ```bash
   pip install Flask Werkzeug
   ```

2. **Configure Your Flask App**:

   ```python
   from flask import Flask, request, redirect, url_for
   from werkzeug.utils import secure_filename
   import os

   app = Flask(__name__)

   # Configure upload folder and allowed extensions
   # use 'public' because it is served by default
   UPLOAD_FOLDER = 'public/'
   ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif', 'pdf', 'txt'}

   app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

   if not os.path.exists(UPLOAD_FOLDER):
        os.makedirs(UPLOAD_FOLDER)
   ```

3. **Define a Function to Check Allowed File Extensions**:

   ```python
   def allowed_file(filename):
       return '.' in filename and \
              filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS
   ```

4. **Define a Function to Generate Unique Filenames**:

   ```python
   def unique_filename(filename):
       import uuid
       ext = filename.rsplit('.', 1)[1].lower()
       return f"{uuid.uuid4().hex}.{ext}"
   ```

5. **Create a Route to Handle File Uploads**:

   ```python

   @app.route('/upload', methods=['GET', 'POST'])
   def upload_file():
        # For POST request, handle file upload
       if request.method == 'POST':
           # Check if the post request has the file part
           if 'file' not in request.files:
               return 'No file part', 400
           file = request.files['file']
           # If user does not select file, browser may submit an empty part
           if file.filename == '':
               return 'No selected file', 400
           if file and allowed_file(file.filename):
               filename = unique_filename(file.filename)
               file.save(os.path.join(current_app.config['UPLOAD_FOLDER'], filename))
               return 'File successfully uploaded', 200
       # For GET request, render upload form
       return '''
         <form method="post" enctype="multipart/form-data">
              <input type="file" name="file">
              <input type="submit" value="Upload">
            </form>
            '''
   ```

6. **Run the Flask App**:

   ```python
   if __name__ == '__main__':
         app.run(debug=True)
   ```

7. **Testing the File Upload**:
   - Start your Flask application.
   - Navigate to `http://localhost:5000/upload` in your web browser.
   - Use the form to select and upload a file.
   - Verify that the file is saved in the `uploads/` directory on your server.

## Important Considerations

- **Security**: Always validate and sanitize file names using `secure_filename` to prevent directory traversal attacks.
  - Checking Mime Types: In addition to checking file extensions, consider validating the MIME type of the uploaded files for added security.
- **File Size Limits**: You can set a maximum file size limit by configuring `app.config['MAX_CONTENT_LENGTH']`.
- **Storage**: For production applications, consider using cloud storage services (like AWS S3) for storing uploaded files instead of the local filesystem.
- **Error Handling**: Implement proper error handling to manage issues like unsupported file types, upload failures, etc.

## Uploading Files in a REST API

When building a REST API, file uploads can be handled similarly, but typically without rendering HTML forms. Instead, clients will send files using multipart/form-data requests.

1. **API Endpoint for Posting Cats with Images**:

   ```python

   # File: /api/v1/cats/cats_controller.py
    # import necessary modules

    def create_cat(current_user):
        if 'file' not in request.files:
            return {'message': 'No file part'}, 400
        file = request.files['file']
        if file.filename == '':
            return {'message': 'No selected file'}, 400
        if file and allowed_file(file.filename):
            filename = unique_filename(file.filename)
            file.save(os.path.join(current_app.config['UPLOAD_FOLDER'], filename))
            # get other cat data from FormData
            data = request.form.to_dict()
            data['image'] = filename
            data['owner'] = current_user.id
            data['location'] = json.loads(data['location'])
            cat = Cat(**data)
            cat.save()
        return {'message': 'File type not allowed'}, 400
   ```

2. Put `allowed_file` and `unique_filename` functions in a common utils file to reuse them.

3. Test the API endpoint using tools like Postman or VS Code REST Client by sending a multipart/form-data request with the file and other cat data.

   ```http
   POST /api/v1/cats HTTP/1.1
   Host: localhost:3000
   Authorization: Bearer <your_jwt_token>
   Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

   ------WebKitFormBoundary7MA4YWxkTrZu0gW
   Content-Disposition: form-data; name="cat_name"

   Whiskers
   ------WebKitFormBoundary7MA4YWxkTrZu0gW
   Content-Disposition: form-data; name="birthdate"

   2020-05-01
   ------WebKitFormBoundary7MA4YWxkTrZu0gW
   Content-Disposition: form-data; name="location"

   {"type": "Point", "coordinates": [-104.9903, 39.7392]}
   ------WebKitFormBoundary7MA4YWxkTrZu0gW
   Content-Disposition: form-data; name="file"; filename="cat.jpg"
   Content-Type: image/jpeg

   <binary data>
   ------WebKitFormBoundary7MA4YWxkTrZu0gW--
   ```

   - Note that the file "cat.jpg" needs to be in the same directory as the request file or provide the full path to it.

## Assignment

- Implement file upload functionality in your Flask application.
- Create a new branch 'file-upload' for this feature.
- Modify the existing cat creation endpoint to accept image uploads.
- Test the file upload feature using Postman or VS Code REST Client.
