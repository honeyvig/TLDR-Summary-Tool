# TLDR-Summary-Tool
 create a TLDR summary tool that can summarize text from any website. Want to also make sure the user doesn’t need a ChatGPT account or anything like that.The extension should be user-friendly and provide concise summaries of web content with a focus on speed and accuracy. 
------------
Creating a TLDR summary tool as a browser extension requires several key steps. Here's an outline of how you can implement this in Python for the backend, with the frontend as a browser extension, ideally built with JavaScript. The goal is to create an efficient and simple extension that provides summaries of web content directly, using natural language processing techniques, without requiring a ChatGPT account or any external API.
Steps:

    Backend (Python API):
        A Python web API that performs the text summarization.
        Use libraries like BeautifulSoup for HTML parsing and transformers for text summarization (e.g., using a model like BART, T5, or GPT-2).

    Frontend (Browser Extension - JavaScript):
        A simple extension with a popup interface that fetches content from the current page and sends it to the backend for summarization.
        Display the summary within the extension.

Step 1: Python Backend for Text Summarization

You will need to create a simple API in Python using Flask (or FastAPI) and use HuggingFace Transformers for text summarization.
1.1 Install required libraries:

pip install flask beautifulsoup4 transformers torch requests

1.2 Create the Flask Backend (app.py):

from flask import Flask, request, jsonify
from transformers import pipeline
import requests
from bs4 import BeautifulSoup

# Initialize Flask app
app = Flask(__name__)

# Initialize HuggingFace summarization pipeline
summarizer = pipeline("summarization", model="facebook/bart-large-cnn")

# Function to extract text from a URL
def extract_text_from_url(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')

    # Extract text content from paragraphs and headings
    paragraphs = soup.find_all('p')
    text = ' '.join([para.get_text() for para in paragraphs])
    return text

@app.route('/summarize', methods=['POST'])
def summarize():
    data = request.get_json()
    url = data.get('url')

    if not url:
        return jsonify({"error": "URL is required"}), 400

    # Extract text from the given URL
    web_content = extract_text_from_url(url)

    # Summarize the extracted text
    summary = summarizer(web_content, max_length=200, min_length=50, do_sample=False)

    return jsonify({"summary": summary[0]['summary_text']})


if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)

Explanation of the Code:

    Flask: A lightweight web framework for Python to build the API.
    BeautifulSoup: Used to parse the HTML page and extract content from <p> tags (paragraphs).
    HuggingFace transformers library: We use the facebook/bart-large-cnn model, which is pre-trained for summarization.
    The /summarize endpoint accepts a POST request with a URL and returns a summary.

Step 2: Frontend (Browser Extension - JavaScript)
2.1 Create Extension Structure

TLDR-Extension/
    ├── background.js
    ├── content.js
    ├── popup.html
    ├── popup.js
    ├── manifest.json
    ├── style.css

2.2 manifest.json (Browser Extension Manifest)

The manifest.json file configures the browser extension and defines its permissions.

{
  "manifest_version": 2,
  "name": "TLDR Summary Tool",
  "version": "1.0",
  "description": "Get a TLDR summary of any web page",
  "permissions": [
    "activeTab",
    "storage",
    "https://*/*"
  ],
  "browser_action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icon16.png",
      "48": "icon48.png",
      "128": "icon128.png"
    }
  },
  "background": {
    "scripts": ["background.js"],
    "persistent": false
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ]
}

2.3 popup.html (User Interface)

This is the HTML interface that pops up when the user clicks on the extension icon.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TLDR Summary Tool</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div id="content">
    <h3>Summarize this Page</h3>
    <button id="summarizeBtn">Get TLDR</button>
    <div id="summary"></div>
  </div>
  <script src="popup.js"></script>
</body>
</html>

2.4 popup.js (Handling Button Click)

When the user clicks the Get TLDR button, we extract the current tab URL and call the Python backend for the summary.

document.getElementById('summarizeBtn').addEventListener('click', function() {
    chrome.tabs.query({ active: true, currentWindow: true }, function(tabs) {
        let url = tabs[0].url;

        // Call the backend for summary
        fetch('http://127.0.0.1:5000/summarize', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ url: url })
        })
        .then(response => response.json())
        .then(data => {
            if (data.summary) {
                document.getElementById('summary').innerText = data.summary;
            } else {
                document.getElementById('summary').innerText = 'Error: ' + data.error;
            }
        })
        .catch(error => {
            document.getElementById('summary').innerText = 'An error occurred: ' + error;
        });
    });
});

2.5 style.css (Simple Styling)

This file styles the popup interface.

body {
  font-family: Arial, sans-serif;
  padding: 10px;
}

#content {
  text-align: center;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}

#summary {
  margin-top: 20px;
  font-size: 14px;
  white-space: pre-wrap;
  word-wrap: break-word;
}

Step 3: Running and Testing

    Start the Flask Backend:

python app.py

This will run the Flask app on http://127.0.0.1:5000.

    Load the Browser Extension:

    Go to chrome://extensions/ in your browser (Chrome/Edge).
    Enable Developer mode.
    Click on Load unpacked and select the extension folder (TLDR-Extension).
    The extension icon should appear in the browser toolbar.

    Test the Extension:
        Click on the extension icon.
        Click the Get TLDR button.
        It should display the summary from the page you are currently viewing.

Conclusion

This solution consists of:

    A Flask backend that fetches text from any URL and summarizes it using the HuggingFace model.
    A browser extension (written in JavaScript) that lets the user quickly summarize the content of a webpage by clicking the extension icon.

The user doesn’t need an account, and the system works entirely on the local setup, ensuring user-friendly, fast, and accurate results.
