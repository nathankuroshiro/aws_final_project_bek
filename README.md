# final\_Nodirbek

## 1. Set up EC2 Instance

* Go to AWS Console → EC2 → “Launch Instance”
* Name: `webapp_bek`
* AMI: Ubuntu
* Instance type: `t2.micro`
* Key pair: Create or choose existing `.pem` file (e.g., `nodirbek-key.pem`)

### Configure Security Group (Inbound Rules)

| Type       | Protocol | Port Range | Source              | Purpose                  |
| ---------- | -------- | ---------- | ------------------- | ------------------------ |
| SSH        | TCP      | 22         | Your IP             | Connect to EC2 using SSH |
| HTTP       | TCP      | 80         | 0.0.0.0/0           | Web server access        |
| Custom TCP | TCP      | 5000       | 0.0.0.0/0           | Access Flask API         |
| PostgreSQL | TCP      | 5432       | 0.0.0.0/0 or EC2 SG | Connect RDS from EC2     |

Click **Launch Instance**.

## 2. Set up RDS PostgreSQL Database

* Go to AWS Console → RDS → Create database
* Choose:

  * Creation method: **Standard create**
  * Engine: **PostgreSQL**
  * DB instance identifier: `db_bek`
  * Master username: `adminbek`
  * Master password: `adminbek`
  * Public access: **Yes**
* Expand “Connectivity”:

  * VPC: default
  * Subnet group: default
  * Security group: same as EC2
* Click **Create database**

### RDS Security Group

* Go to RDS → Databases → `db_bek` → Connectivity & security → VPC Security Group
* Edit inbound rules:

| Type       | Protocol | Port Range | Source (EC2 SG ID) | Purpose                 |
| ---------- | -------- | ---------- | ------------------ | ----------------------- |
| PostgreSQL | TCP      | 5432       | EC2 security group | Allow EC2 to access RDS |

## 3. Connect to RDS and Import Data Using DBeaver

### Step 1: Connect to RDS

1. Open DBeaver
2. Database → New Database Connection → Select PostgreSQL
3. Enter:

   * Host: `db-bek.c32geugqgvkj.ap-southeast-1.rds.amazonaws.com`
   * Database: `postgres`
   * User: `adminbek`
   * Password: `adminbek`
4. Test Connection → Finish

### Step 2: Create New Database

* Right-click on `postgres` → Create → Database
* Name: `db_bek`
* Click OK

### Step 3: Import CSV to New Table

* Right-click on `db_bek` → Tools → Import Data from CSV
* Select CSV file
* Table name: `tbl_bek_exam_results`
* Click Next → Finish

## 4. SSH into EC2

```bash
ssh -i "D:\Загрузки\nodirbek-key.pem" ubuntu@18.142.189.10
```

## 5. Set up Flask Environment

```bash
sudo apt update
sudo apt install python3-pip python3.10-venv -y
```

## 6. Create Project Folder and Virtual Env

```bash
mkdir webapp_bek
cd webapp_bek
python3 -m venv venv
source venv/bin/activate
```

## 7. Install Python Dependencies

```bash
pip install flask flask-cors psycopg2-binary
```

## 8. Create Flask App (app.py)

```python
from flask import Flask, jsonify
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
    app.run(host="0.0.0.0", port=5000)
```

## 9. Run Flask App

```bash
python app.py
```

Expected: `Running on http://0.0.0.0:5000/`

## 10. Create index\_amal.html

Create file `index_bek.html`:

```html
<!DOCTYPE html>
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
```

## 11. Upload HTML to S3

* Go to AWS Console → S3 → Create bucket
* Name: `bekbucket`
* Uncheck: **Block all public access**
* Go to **Properties → Static website hosting**

  * Enable
  * Index document: `index_bek.html`
* Upload your `index_bek.html` file
* Go to **Permissions → Bucket Policy** and add:

```json
{
  "Version":"2012-10-17",
  "Statement":[{
    "Effect":"Allow",
    "Principal":"*",
    "Action":"s3:GetObject",
    "Resource":"arn:aws:s3:::netflix-dashboard-amal/*"
  }]
}
```

## 12. Final Test

Open:

```
http://bek-bucket.s3-website-ap-southeast-1.amazonaws.com
```

