# Project Screenshots — CloudBlitz Student Registration App

This document contains screenshots showing the working proof of the full-stack deployment on AWS EC2.

---

## Screenshot 1 — Frontend (React App Live on EC2)

**URL:** `http://35.154.66.193` (Apache2 on Port 80)

![CloudBlitz Student Registration Frontend](screenshot/home.png)

### What this shows:
- React frontend successfully deployed on AWS EC2 using Apache2
- The **CloudBlitz Student Registration** form is live and accessible via the EC2 public IP
- Form fields include: Name, Email, Course, Highest Education, Percentage, Branch or Stream, Mobile Number
- The frontend is connected to the Spring Boot backend running on port 8080

---

## Screenshot 2 — Database (MariaDB Data Verification)

**Tool:** AWS EC2 Terminal — MariaDB CLI

![MariaDB Student Data](data_png.png)

### What this shows:
- MariaDB is running and the `students` table is storing data correctly
- Two student records inserted via the frontend registration form:

| id | branch | course | email | mobile_number | name | percentage | student_class |
|---|---|---|---|---|---|---|---|
| 1 | asdf | sdf | abc@gmail.com | 12345678 | asdfgh | 123 | sdfg |
| 2 | cse | cdec | johan123@gmail.com | 123456789 | johan | 80 | b.tech |

- Data flows correctly: **Frontend → Spring Boot Backend → MariaDB Database**
- End-to-end integration is verified and working

---

## Architecture Summary

```
User Browser
     │
     ▼
React Frontend (Apache2 — Port 80)
EC2 Public IP: 35.154.66.193
     │
     ▼
Spring Boot Backend (Java — Port 8080)
REST API: http://<BACKEND_IP>:8080/api
     │
     ▼
MariaDB Database (Port 3306)
Database: student_db
Table: students
```

---

## Deployment Stack

| Layer | Technology | Port |
|---|---|---|
| Frontend | React + Vite → Apache2 | 80 |
| Backend | Spring Boot (Java 17 + Maven) | 8080 |
| Database | MariaDB | 3306 |
| Cloud | AWS EC2 (Ubuntu) | — |

---

*Screenshots captured after successful full-stack deployment on AWS EC2 — CloudBlitz Student Registration Project*
