🗂 Todo Application API
RESTful Task Management Service using Node.js & SQLite
📌 Project Overview

This project is a backend REST API that manages daily tasks with priority, category, status tracking, and due date scheduling.

It enables users to:

✔ create tasks
✔ filter & search todos
✔ track progress
✔ manage deadlines
✔ update task status dynamically

The system is built for efficiency, validation accuracy, and structured task tracking.

🎯 Problem It Solves

Traditional task lists lack:

• structured prioritization
• category organization
• deadline tracking
• progress monitoring
• filtered retrieval

This API provides a structured productivity engine for task management systems.

🛠 Technology Stack

Backend

Node.js

Express.js

Database

SQLite

Libraries

date-fns → date validation & formatting

sqlite → database connection

🧩 Database Schema
Todo Table
Column	Type	Description
id	INTEGER	Unique task ID
todo	TEXT	Task description
priority	TEXT	HIGH / MEDIUM / LOW
status	TEXT	TO DO / IN PROGRESS / DONE
category	TEXT	WORK / HOME / LEARNING
due_date	DATE	Task deadline
⚙️ Core Features
✅ 1. Create Todo

Add tasks with priority, status, category, and due date.

✅ 2. Retrieve Todos

Filter using:

search text

category

priority

status

✅ 3. Retrieve Single Todo

Fetch task using ID.

✅ 4. Agenda View

View tasks due on a specific date.

✅ 5. Update Todo

Update individual fields:

✔ status
✔ priority
✔ category
✔ task text
✔ due date

✅ 6. Delete Todo

Remove tasks permanently.

🔍 Advanced Functionalities
✔ Input Validation Middleware

Ensures:

valid priority values

valid categories

valid statuses

valid date format

✔ Date Validation Engine

Uses date-fns to ensure correct due date formatting and validation.

✔ Dynamic Query Filtering

Builds SQL queries based on user filters.
👩‍💻 Author

Keerthana Reddy Full Stack Developer
