# AWS Final Project: Full Web App Deployment (Bek)

This project demonstrates a full-stack web application deployment using **AWS services**:

- ✅ EC2 (backend server with Flask)
- ✅ RDS (PostgreSQL database)
- ✅ S3 (static hosting for frontend)

---

## 💻 Technologies

- Python 3.12
- Flask
- HTML + CSS (hosted on S3)
- PostgreSQL (hosted on RDS)

---

## 🌐 How the App Works

1. `Add Data`: Inserts one random row into the database.
2. `Show Data`: Displays all rows from the PostgreSQL table.
3. `Delete Data`: Deletes one specific row.
4. `Delete All Data`: Clears the entire table.

---

## 🔐 AWS Setup

- **EC2 Instance**: Ubuntu 22.04, public IP with Elastic IP.
- **RDS Instance**: PostgreSQL, security group allows EC2 access.
- **S3 Bucket**: Public read access, static website enabled.

---

## ⚙️ How to Recreate the App

1. Launch EC2 and RDS.
2. SSH into EC2 and install:
    ```bash
    sudo apt update
    sudo apt install python3-pip
    python3 -m venv venv
    source venv/bin/activate
    pip install flask flask-cors psycopg2-binary
    ```

3. Run `app.py`:
    ```bash
    python3 app.py
    ```

4. Upload your `index.html` to S3 and link it to your backend:
    ```html
    fetch("http://<EC2_IP>:5000/results")
    ```

---

## 🧪 Final Defense Tasks

During the exam, I was asked to:
- Rename the database table
- Change the database name
- Drop and recreate columns
- Update frontend/backend logic accordingly

---

## 📁 Project Structure

```
📦 aws_final_project
├── app.py           # Flask backend
├── index.html       # Frontend (uploaded to S3)
└── README.md
```

---

### ✅ Everything was tested and works even after reboot with static IPs.
