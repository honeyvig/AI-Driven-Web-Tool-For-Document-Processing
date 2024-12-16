# AI-Driven-Web-Tool-For-Document-Processing
build a web application featuring AI-powered document processing. Core requirements include file uploads, automated processing of user data, and timed interactions. Experience with AI integrations (e.g., GPT APIs) is crucial.

Responsibilities:
Develop a secure, user-friendly site with seamless file handling
Implement AI-driven workflows or plugins
Manage scheduling or timer-based triggers
Ensure reliability and scalability
----------
To build a web application that integrates AI-powered document processing with features such as file uploads, automated processing, and timed interactions, we can use a combination of Python-based tools like Flask, OpenAI GPT API, Celery for task scheduling, and SQLAlchemy for database handling. Here's a Python code outline to help you get started.
Key Features:

    File Uploads: Users can upload documents (PDF, DOCX, etc.).
    Automated Processing: AI (e.g., OpenAI GPT) will process the uploaded files (e.g., text extraction, summarization).
    Timed Interactions: Using Celery to schedule processing tasks or send timed interactions.
    User Management: Optional for login and tracking document uploads.

Libraries Required:

    Flask: Lightweight web framework.
    OpenAI API: For GPT-powered document processing.
    Celery: Task scheduling for timed interactions.
    SQLAlchemy: Database management (if needed for user data or file tracking).
    Werkzeug: For secure file handling.

Python Code Implementation
1. Install Required Libraries

You need to install the following dependencies:

pip install Flask openai celery sqlalchemy werkzeug

2. Flask Web Application with File Uploads and AI Processing

from flask import Flask, render_template, request, redirect, url_for, flash, send_from_directory
from werkzeug.utils import secure_filename
import openai
import os
from celery import Celery
import time

# Initialize Flask app and OpenAI API key
app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = 'uploads/'
app.config['ALLOWED_EXTENSIONS'] = {'pdf', 'docx', 'txt'}
app.secret_key = 'your_secret_key'

openai.api_key = 'YOUR_OPENAI_API_KEY'

# Celery configuration
app.config['CELERY_BROKER_URL'] = 'redis://localhost:6379/0'
app.config['CELERY_RESULT_BACKEND'] = 'redis://localhost:6379/0'
celery = Celery(app.name, broker=app.config['CELERY_BROKER_URL'])
celery.conf.update(app.config)

# Allowed file extensions
def allowed_file(filename):
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in app.config['ALLOWED_EXTENSIONS']

# Route to upload files
@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)
        file = request.files['file']
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
            file.save(filepath)
            flash('File successfully uploaded')
            process_document.delay(filepath)  # Trigger AI processing in the background
            return redirect(url_for('uploaded_file', filename=filename))
    return render_template('upload.html')

# AI-powered document processing using OpenAI GPT API
@celery.task(bind=True)
def process_document(self, filepath):
    try:
        # Read the uploaded file (assuming it's a text file for simplicity)
        with open(filepath, 'r') as file:
            content = file.read()

        # Process the content using OpenAI API (e.g., summarization)
        prompt = f"Summarize the following text:\n\n{content}"
        response = openai.Completion.create(
            engine="text-davinci-003",  # GPT-3 model
            prompt=prompt,
            max_tokens=200
        )

        summary = response.choices[0].text.strip()

        # Store or return the summary (for simplicity, printing here)
        print(f"Summary: {summary}")
        return summary
    except Exception as e:
        print(f"Error: {str(e)}")
        raise self.retry(exc=e)

# Route to display the uploaded file and result
@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return render_template('result.html', filename=filename)

# Route to serve uploaded files
@app.route('/uploads/<filename>', methods=['GET'])
def send_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'], filename)

# Start the Flask app
if __name__ == '__main__':
    # Create upload folder if it doesn't exist
    if not os.path.exists(app.config['UPLOAD_FOLDER']):
        os.makedirs(app.config['UPLOAD_FOLDER'])
    
    app.run(debug=True)

Key Features and Functions:

    File Upload: Users can upload files (PDF, DOCX, TXT) using the / route.
    Automated Processing: Once the file is uploaded, the process_document function is called in the background using Celery. The function reads the file content and sends it to the OpenAI API for processing (e.g., summarization).
    Timed Interaction: You can easily set up Celery to process tasks at specific intervals or after a time delay.
    AI Integration: The file content is processed using OpenAI’s GPT model to generate summaries or perform other NLP tasks. This can be modified to extract text from PDF or DOCX files, depending on the file format.

3. Celery Worker Setup

To run Celery, you'll need to start the Celery worker and have Redis installed and running as the broker.

Start the Celery worker in the command line:

celery -A app.celery worker --loglevel=info

4. AI Integration (OpenAI GPT)

    In this example, the AI integration is used for text summarization, but you can customize it to perform other tasks like sentiment analysis, entity extraction, etc.
    You will need to get your own OpenAI API key and set it in the code.

5. HTML Templates for File Upload and Result Display

Create two HTML templates for file uploads and showing the results.
upload.html:

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Upload File</title>
</head>
<body>
    <h1>Upload Document</h1>
    <form method="POST" enctype="multipart/form-data">
        <input type="file" name="file" accept=".txt,.pdf,.docx">
        <button type="submit">Upload</button>
    </form>
</body>
</html>

result.html:

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Processing Result</title>
</head>
<body>
    <h1>Document Summary</h1>
    <p>Your document has been processed successfully. Here is the summary:</p>
    <p>{{ summary }}</p>
</body>
</html>

6. Running the Application

    Start the Flask app:

python app.py

    Start the Celery worker as mentioned earlier to handle background tasks.

Conclusion

This Python-based web application integrates AI-powered document processing, allowing users to upload files and automatically process them using OpenAI GPT APIs. The application utilizes Celery for background task processing, ensuring reliability and scalability in document processing. You can extend this framework for other types of document analysis, add user authentication, or integrate with additional APIs depending on your requirements.
