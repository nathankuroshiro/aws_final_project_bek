
# Final Project: Full Web Application Deployment on AWS

### Student: Nodirbek Fatkhullaev

---

## 💡 Project Description
This project demonstrates the deployment of a full-stack web application using three AWS services:
- **Amazon RDS** for hosting a PostgreSQL database
- **Amazon EC2** for backend logic using Flask (Python)
- **Amazon S3** for static frontend (HTML/CSS/JS)

The app allows users to **add**, **delete (one)**, **delete all**, and **view** rows in the database via buttons on a web page.

---

## 🔎 Project Components

### 1. **Amazon RDS** (Database)
- **DB Instance name**: `db_bek`
- **Engine**: PostgreSQL
- **Endpoint**: `db-bek.c32geugqgvkj.ap-southeast-1.rds.amazonaws.com`
- **Database Name**: `postgres`
- **Table**: `tbl_bek_studentscore`
- **Username**: `adminbek`
- **Password**: `adminbek`

**Table Schema**:
```sql
id SERIAL PRIMARY KEY,
name VARCHAR(50),
score INTEGER
```

### 2. **Amazon S3** (Frontend)
- **Bucket name**: `bek-webapp-bucket`
- **Static Hosting Enabled**
- **Main HTML File**: `index_bek.html`

Hosted at:
```
http://bek-webapp-bucket.s3-website-ap-southeast-1.amazonaws.com
```

HTML has four buttons:
- Add Data
- Delete One
- Delete All
- Show Data

### 3. **Amazon EC2** (Backend)
- **OS**: Ubuntu
- **Elastic IP Assigned**: Yes
- **Deployed Python App**: `app.py` with Flask
- **Installed**: Flask, psycopg2, flask_cors
- **Run Command**:
```bash
source venv/bin/activate
python3 app.py

```

App listens on:
```
http://<your-elastic-ip>:8000 or 5000
```

---

## 📆 Step-by-Step: How to Deploy



### ✅ 1. Setup Virtual Environment on EC2
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### ✅ 2. Run Flask Backend
```bash
python3 app.py
```
`from flask import Flask, jsonify
from flask_cors import CORS
import psycopg2
import random

app = Flask(__name__)
CORS(app)


conn = psycopg2.connect(
    host="db-bek.c32geugqgvkj.ap-southeast-1.rds.amazonaws.com",
    database="postgres",
    user="adminbek",
    password="adminbek"
)

@app.route("/")
def home():
    return "Welcome to Exam Results API"

@app.route("/results")
def get_results():
    try:
        conn.rollback()
        cur = conn.cursor()
        cur.execute("SELECT * FROM exam_results")
        rows = cur.fetchall()
        cur.close()
        return jsonify(rows)
    except Exception as e:
        return jsonify({"error": str(e)})

@app.route("/add", methods=["POST"])
def add_data():
    try:
        cur = conn.cursor()
        genders = ["male", "female"]
        groups = ["group A", "group B", "group C", "group D", "group E"]
        educations = ["high school", "some high school", "some college", "associate's degree", "bachelor's degree", "ma>        lunches = ["standard", "free/reduced"]
        preps = ["none", "completed"]
        scores = [random.randint(40, 100) for _ in range(3)]  # math, reading, writing

        query = """
        INSERT INTO exam_results VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
        """
        cur.execute(query, (
            random.choice(genders),
            random.choice(groups),
            random.choice(educations),
            random.choice(lunches),
            random.choice(preps),
            scores[0],
            scores[1],
            scores[2]
        ))
        conn.commit()
        cur.close()
        return "Random data added!"
    except Exception as e:
 return jsonify({"error": str(e)})

@app.route("/delete", methods=["POST"])
def delete_one():
    cur = conn.cursor()
    try:
        cur.execute("""
            DELETE FROM exam_results
            WHERE ctid IN (
                SELECT ctid FROM exam_results LIMIT 1
            )
            RETURNING *;
        """)
        deleted_row = cur.fetchone()
        conn.commit()
        cur.close()
        if deleted_row:
            return "One row deleted!"
        else:
            return "No rows found."
    except Exception as e:
        return jsonify({"error": str(e)})

@app.route("/delete_all", methods=["POST"])
def delete_all():
    cur = conn.cursor()
    cur.execute("DELETE FROM exam_results")
    conn.commit()
    cur.close()
    return "All rows deleted!"


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)`

Make sure port 8000 or 5000 is allowed in EC2 security group.

### ✅ 3. Upload Static Files to S3
- Go to S3 > your bucket > Upload
- Upload `index_bek.html`
 `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Bek's Web App</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        h1 {
            color: #222;
        }
        button {
            margin: 5px;
            padding: 10px;
            font-size: 14px;
        }
        table, th, td {
            border: 1px solid black;
            border-collapse: collapse;
            padding: 6px;
        }
        table {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <h1>Welcome to Bek's Web App</h1>
    <button onclick="addData()">Add Data</button>
    <button onclick="deleteOne()">Delete Data</button>
    <button onclick="deleteAll()">Delete All Data</button>
    <button onclick="showData()">Show Data</button>
    <div id="table-container"></div>

    <script>
        const apiBase = "http://18.142.189.10:5000";

        async function addData() {
            try {
                const response = await fetch(`${apiBase}/add`, { method: 'POST' });
                const result = await response.text();
                alert(result);
            } catch (error) {
                alert("Error: " + error);
            }
        }

        async function deleteOne() {
            try {
                const response = await fetch(`${apiBase}/delete`, { method: 'POST' });
                const result = await response.text();
                alert(result);
            } catch (error) {
                alert("Error: " + error);
            }
        }

        async function deleteAll() {
            try {
                const response = await fetch(`${apiBase}/delete_all`, { method: 'POST' });
                const result = await response.text();
                alert(result);
            } catch (error) {
                alert("Error: " + error);
            }
        }

        async function showData() {
            try {
                const response = await fetch(`${apiBase}/results`);
                const data = await response.json();
                let html = "<table><tr><th>Gender</th><th>Group</th><th>Edu</th><th>Lunch</th><th>Prep</th><th>Math</th><th>Reading</th><th>Writing</th></tr>";
                data.forEach(row => {
                    html += `<tr>${row.map(val => `<td>${val}</td>`).join("")}</tr>`;
                });
                html += "</table>";
                document.getElementById("table-container").innerHTML = html;
            } catch (error) {
                alert("Error: " + error);
            }
        }
    </script>
</body>
</html>

- Enable **static website hosting** in properties

---

## 🎓 Final Deliverables

- [x] EC2 Flask backend working
- [x] S3 frontend connected and visible
- [x] RDS PostgreSQL database accessible
- [x] Add/Delete/Show buttons working
- [x] Buttons connect frontend and backend


---

## 🚀 Extra Notes
- **Elastic IP**: assigned and won't change after stop/start.
- **Security Groups**: 8000 and 5432 ports open.
- **CORS enabled** for frontend-backend communication.
- **Randomized data** added on each Add click.
