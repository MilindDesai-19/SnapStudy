# ⚡ SnapStudy

> **Learn Smart, Study Fast** — Upload your study material and get AI-generated summaries and practice quizzes instantly.

---

## 📖 Overview

SnapStudy is a Flask-based web application that helps students study more efficiently. Upload a PDF or paste text from your study material, and SnapStudy will:

- Generate a structured **AI summary** using Facebook's BART model
- Automatically create a **fill-in-the-blank quiz** from the content
- Track your **quiz scores** over time
- Provide an **admin panel** to manage users and monitor platform activity

---

## ✨ Features

| Feature | Description |
|---|---|
| 📄 PDF Upload | Upload PDF files or paste text directly |
| 🤖 AI Summarization | Summaries powered by `facebook/bart-large-cnn` |
| 📝 Auto Quiz Generation | Rule-based NLP generates fill-in-the-blank questions |
| 📊 Score Tracking | Quiz results saved and displayed on dashboard |
| 🔐 Auth System | Student registration/login with hashed passwords |
| 🛡️ Admin Panel | Manage users, view stats, delete accounts |

---

## 🛠️ Tech Stack

- **Backend:** Python, Flask 3.0, Flask-Login
- **Database:** MySQL with connection pooling
- **AI Model:** `facebook/bart-large-cnn` via HuggingFace Transformers
- **PDF Parsing:** PyPDF2
- **Frontend:** Bootstrap 5, Bootstrap Icons
- **Auth:** Werkzeug password hashing

---

## 📁 Project Structure

```
snapstudy/
├── run.py                  # App entry point
├── requirements.txt        # Python dependencies
├── .env                    # Environment variables (not committed)
│
├── app/
│   ├── __init__.py         # App factory, blueprint registration
│   ├── auth.py             # User & AdminUser classes (Flask-Login)
│   ├── database.py         # MySQL connection pool
│   ├── models.py           # DB models: User, Module, Summary, Quiz, Admin
│   │
│   ├── routes/
│   │   ├── auth_routes.py    # /, /register, /login, /logout
│   │   ├── module_routes.py  # /dashboard, /upload, /summary, /quiz
│   │   └── admin_routes.py   # /admin/login, /admin/dashboard, /admin/delete_user
│   │
│   ├── services/
│   │   ├── pdf_processor.py  # PDF text extraction (PyPDF2)
│   │   ├── summarizer.py     # BART summarization pipeline
│   │   └── quiz_generator.py # Rule-based quiz generation
│   │
│   └── templates/
│       ├── base.html
│       ├── landing.html
│       ├── login.html / register.html
│       ├── dashboard.html
│       ├── upload.html
│       ├── summary.html
│       ├── quiz.html
│       ├── admin_login.html
│       └── admin_dashboard.html
```

---

## ⚙️ Setup & Installation

### Prerequisites

- Python 3.10+
- MySQL 8.0+
- ~2GB disk space (for BART model download)

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/snapstudy.git
cd snapstudy
```

### 2. Create a virtual environment

```bash
python -m venv venv
source venv/bin/activate        # macOS/Linux
venv\Scripts\activate           # Windows
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

> ⚠️ The first run will download the `facebook/bart-large-cnn` model (~1.6GB). This takes a few minutes.

### 4. Configure environment variables

Create a `.env` file in the project root:

```env
DB_HOST=localhost
DB_USER=your_mysql_username
DB_PASSWORD=your_mysql_password
DB_NAME=snapstudy
SECRET_KEY=your-secret-key-here
```

### 5. Set up the MySQL database

```sql
CREATE DATABASE snapstudy;
USE snapstudy;

CREATE TABLE User (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    date_joined DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE Admin (
    admin_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL,
    password VARCHAR(255) NOT NULL
);

CREATE TABLE Modules (
    module_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    module_name VARCHAR(200) NOT NULL,
    file_path VARCHAR(500),
    upload_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(user_id)
);

CREATE TABLE Summary (
    summary_id INT AUTO_INCREMENT PRIMARY KEY,
    module_id INT NOT NULL,
    summary_text LONGTEXT NOT NULL,
    FOREIGN KEY (module_id) REFERENCES Modules(module_id)
);

CREATE TABLE Quiz (
    quiz_id INT AUTO_INCREMENT PRIMARY KEY,
    module_id INT NOT NULL,
    total_questions INT NOT NULL,
    FOREIGN KEY (module_id) REFERENCES Modules(module_id)
);

CREATE TABLE Questions (
    question_id INT AUTO_INCREMENT PRIMARY KEY,
    quiz_id INT NOT NULL,
    question_text TEXT NOT NULL,
    option_a VARCHAR(500) NOT NULL,
    option_b VARCHAR(500) NOT NULL,
    option_c VARCHAR(500) NOT NULL,
    option_d VARCHAR(500) NOT NULL,
    correct_answer VARCHAR(10) NOT NULL,
    FOREIGN KEY (quiz_id) REFERENCES Quiz(quiz_id)
);

CREATE TABLE Result (
    result_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    quiz_id INT NOT NULL,
    score INT NOT NULL,
    attempt_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(user_id),
    FOREIGN KEY (quiz_id) REFERENCES Quiz(quiz_id)
);
```

### 6. Run the application

```bash
python run.py
```

Visit `http://localhost:5000` in your browser.

---

## 🚀 Usage

### Student Flow

1. Register or log in at `/register` or `/login`
2. Upload a PDF or paste study text at `/upload`
3. View the AI-generated summary
4. Take the auto-generated quiz
5. Track your scores on the dashboard

### Admin Flow

1. Go to `/admin/login` and log in with admin credentials
2. View all registered users and their module counts
3. Delete users and all their associated data if needed

> To create an admin account, insert directly into the `Admin` table with a hashed password (use `werkzeug.security.generate_password_hash`).

---

## 📦 Dependencies

```
flask==3.0.0
flask-login==0.6.3
mysql-connector-python==8.2.0
PyPDF2==3.0.1
python-dotenv==1.0.0
werkzeug==3.0.1
transformers==4.35.0
torch==2.1.0
sentencepiece==0.1.99
```

---

## ⚠️ Known Limitations

- **First upload is slow** — the BART model (~1.6GB) loads on first use, taking 2–3 minutes
- **Quiz quality varies** — the quiz generator is rule-based; short or poorly formatted text may produce weak questions
- **No file management** — uploaded PDFs are not deletable from the UI
- **Single admin role** — no permission tiers for admins

---

## 🔒 Security Notes

- Passwords are hashed using Werkzeug's `generate_password_hash`
- Keep your `.env` file out of version control (add it to `.gitignore`)
- Change the `SECRET_KEY` to a strong random value in production
- This app is intended for local/educational use — additional hardening is needed before production deployment

---

## 📄 License

This project is licensed under the MIT License.

---

## 🙌 Acknowledgements

- [Facebook BART](https://huggingface.co/facebook/bart-large-cnn) for the summarization model
- [HuggingFace Transformers](https://github.com/huggingface/transformers)
- [Bootstrap 5](https://getbootstrap.com/)
- [Flask](https://flask.palletsprojects.com/)
