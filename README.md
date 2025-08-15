app.py: from flask import Flask, render_template, request, redirect
import mysql.connector
from datetime import datetime

app = Flask(__name__)

# MySQL Database Connection (AWS RDS)
db = mysql.connector.connect(
    host="chatdb-instance.cpoguyq0syid.eu-north-1.rds.amazonaws.com",
    user="admin",
    password="venkatasrisatyavarshini",
    database="chatdb"
)
cursor = db.cursor()

# Home route - show messages
@app.route('/')
def home():
    cursor.execute("SELECT sender, message, timestamp FROM messages ORDER BY timestamp DESC")
    messages = cursor.fetchall()
    return render_template("index.html", messages=messages)

# Route to send a message
@app.route('/send', methods=['POST'])
def send():
    sender = request.form['sender']
    message = request.form['message']
    timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')

    cursor.execute(
        "INSERT INTO messages (sender, message, timestamp) VALUES (%s, %s, %s)",
        (sender, message, timestamp)
    )
    db.commit()
    return redirect('/')

if __name__ == '__main__':
    # Run on all interfaces so EC2 can serve it
    app.run(host='0.0.0.0', port=5000, debug=True)


index.html: {% extends "layout.html" %}

{% block content %}
<h2>Chat Room</h2>

{% for sender, message, timestamp in messages %}
<div class="message">
    <strong>{{ sender }}</strong>: {{ message }}<br>
    <span class="timestamp">{{ timestamp }}</span>
</div>
{% endfor %}

<h3>Send a Message</h3>
<form method="POST" action="/send">
    <input type="text" name="sender" placeholder="Your name" required>
    <textarea name="message" placeholder="Type your message here..." required></textarea>
    <button type="submit">Send</button>
</form>
{% endblock %}


layout.html: <!DOCTYPE html>
<html>
<head>
    <title>Simple Chat App</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f5f5f5; padding: 20px; }
        .chat-box { background: white; padding: 20px; border-radius: 10px; width: 500px; margin: auto; }
        .message { border-bottom: 1px solid #ddd; padding: 5px; }
        .timestamp { font-size: 0.8em; color: gray; }
        input, textarea { width: 100%; padding: 8px; margin: 5px 0; }
        button { padding: 10px 15px; background-color: #4CAF50; color: white; border: none; cursor: pointer; }
        button:hover { background-color: #45a049; }
    </style>
</head>
<body>
    <div class="chat-box">
        {% block content %}{% endblock %}
    </div>
</body>
</html>


chatapp_sqlquery: -- 1. Create the database
CREATE DATABASE IF NOT EXISTS chatdb;

-- 2. Use the database
USE chatdb;

-- 3. Drop the old table if it exists (so we can start fresh)
DROP TABLE IF EXISTS messages;

-- 4. Create the messages table with correct columns
CREATE TABLE messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    sender VARCHAR(50) NOT NULL,
    message TEXT NOT NULL,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);
