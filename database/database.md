# MariaDB Setup and Configuration Guide (AWS EC2 — Ubuntu)

A step-by-step guide to installing MariaDB, securing it, creating a database, user, and table for the Spring Boot backend.

---

## Prerequisites

- AWS EC2 instance running Ubuntu
- SSH access to your EC2 instance
- Port **3306** open in EC2 Security Group inbound rules (only if backend is on a separate instance)

---

## Step 1 — Install MariaDB

```bash
apt update && apt install mariadb-server -y
```

### Start and enable MariaDB service

```bash
systemctl start mariadb
systemctl enable mariadb
```

### Verify MariaDB is running

```bash
systemctl status mariadb
```

---

## Step 2 — Secure MariaDB

Run the built-in security script:

```bash
mysql_secure_installation
```

Follow the prompts:

| Prompt | Recommended Action |
|---|---|
| Set root password | Yes — set a strong password |
| Remove anonymous users | Yes |
| Disallow root login remotely | Yes |
| Remove test database | Yes |
| Reload privilege tables | Yes |

---

## Step 3 — Login to MariaDB

```bash
mysql -u root -p
```

Enter the root password when prompted.

---

## Step 4 — Create Database and User

Run the following SQL commands inside the MariaDB shell:

### Create the database

```sql
CREATE DATABASE student_db;
```

### Create a user and grant privileges

```sql
GRANT ALL PRIVILEGES ON student_db.* TO 'username'@'localhost' IDENTIFIED BY 'your_password';
FLUSH PRIVILEGES;
```

> Replace `username` and `your_password` with your desired values.

> **Note:** The original guide had a typo — `springbackend.*` was used instead of `student_db.*`. Make sure the database name in `GRANT` matches the one you created (`student_db`).

---

## Step 5 — Create the Students Table

Switch to the database:

```sql
USE student_db;
```

Create the `students` table:

```sql
CREATE TABLE `students` (
  `id`             bigint(20)   NOT NULL AUTO_INCREMENT,
  `name`           varchar(255) DEFAULT NULL,
  `email`          varchar(255) DEFAULT NULL,
  `course`         varchar(255) DEFAULT NULL,
  `student_class`  varchar(255) DEFAULT NULL,
  `percentage`     double       DEFAULT NULL,
  `branch`         varchar(255) DEFAULT NULL,
  `mobile_number`  varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=80 DEFAULT CHARSET=latin1 COLLATE=latin1_swedish_ci;
```

### Verify the table was created

```sql
SHOW TABLES;
DESCRIBE students;
```

---

## Step 6 — Exit MariaDB

```sql
EXIT;
```

---

## Step 7 — Database Credentials for Backend

Use these values in your Spring Boot `application.properties`:

| Credential | Value |
|---|---|
| `DB_HOST` | Private IP of your DB EC2 instance (or `localhost` if same instance) |
| `DB_PORT` | `3306` |
| `DB_NAME` | `student_db` |
| `DB_USER` | `username` (whatever you set in Step 4) |
| `DB_PASS` | `your_password` (whatever you set in Step 4) |

These map to your `application.properties` like this:

```properties
server.port=8080
spring.datasource.url=jdbc:mariadb://<DB_HOST>:3306/student_db
spring.datasource.username=<DB_USER>
spring.datasource.password=<DB_PASS>
```

---

## (Optional) Step 8 — Allow Remote Access for Backend on Separate EC2

If your Spring Boot backend runs on a **different EC2 instance**, the user must allow connections from that instance's IP instead of `localhost`:

```bash
mysql -u root -p
```

```sql
GRANT ALL PRIVILEGES ON student_db.* TO 'username'@'<BACKEND_PRIVATE_IP>' IDENTIFIED BY 'your_password';
FLUSH PRIVILEGES;
EXIT;
```

Also open port **3306** in the DB instance's EC2 Security Group inbound rules, restricted to the backend instance's private IP.

---

## Quick Reference — All Commands in Order

```bash
# 1. Install MariaDB
apt update && apt install mariadb-server -y
systemctl start mariadb
systemctl enable mariadb

# 2. Secure installation
mysql_secure_installation

# 3. Login
mysql -u root -p
```

```sql
-- 4. Create database and user
CREATE DATABASE student_db;
GRANT ALL PRIVILEGES ON student_db.* TO 'username'@'localhost' IDENTIFIED BY 'your_password';
FLUSH PRIVILEGES;

-- 5. Create table
USE student_db;
CREATE TABLE `students` (
  `id`             bigint(20)   NOT NULL AUTO_INCREMENT,
  `name`           varchar(255) DEFAULT NULL,
  `email`          varchar(255) DEFAULT NULL,
  `course`         varchar(255) DEFAULT NULL,
  `student_class`  varchar(255) DEFAULT NULL,
  `percentage`     double       DEFAULT NULL,
  `branch`         varchar(255) DEFAULT NULL,
  `mobile_number`  varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=80 DEFAULT CHARSET=latin1 COLLATE=latin1_swedish_ci;

-- 6. Verify and exit
SHOW TABLES;
DESCRIBE students;
EXIT;
```

---

## Troubleshooting

| Issue | Fix |
|---|---|
| `mysql: command not found` | Run `apt install mariadb-server -y` |
| `Access denied for user root` | Use `sudo mysql -u root` without `-p` first, then set password |
| Backend cannot connect to DB | Check `DB_HOST`, credentials, and port 3306 is open in Security Group |
| `GRANT` not working | Make sure you run `FLUSH PRIVILEGES;` after the grant |
| Table already exists error | Run `DROP TABLE IF EXISTS students;` before creating again |
| Remote connection refused | Grant privileges using backend's private IP instead of `localhost` |

---

*Guide covers: MariaDB install · mysql_secure_installation · database & user creation · students table · backend credentials · remote access setup*