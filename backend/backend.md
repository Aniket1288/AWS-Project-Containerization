# Spring Boot Backend Deployment Guide (AWS EC2 — Ubuntu)

A step-by-step guide to deploying a Spring Boot backend on an AWS EC2 instance with MariaDB database connectivity.

---

## Prerequisites

- AWS EC2 instance running Ubuntu
- Your Spring Boot project pushed to GitHub
- Port **8080** open in EC2 Security Group inbound rules
- SSH access to your EC2 instance
- MariaDB database ready (host, name, username, password)

---

## Step 1 — Clone Your GitHub Repo

SSH into your EC2 instance, then clone your project:

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

> Replace `<your-username>` and `<your-repo>` with your actual GitHub values.

---

## Step 2 — Install Java (JDK 17)

```bash
apt update && apt install openjdk-17-jdk -y
```

### Verify Installation

```bash
java -version
```

Expected output:

```
openjdk version "17.x.x" ...
```

---

## Step 3 — Install Maven

```bash
apt install maven -y
```

### Verify Installation

```bash
mvn -version
```

---

## Step 4 — Configure Database Credentials

Navigate into the backend directory and open `application.properties`:

```bash
cd backend
vim src/main/resources/application.properties
```

Inside vim:
- Press `i` to enter **insert mode**
- Update the following properties:

```properties
server.port=8080
spring.datasource.url=jdbc:mariadb://<DB_HOST>:3306/<DB_NAME>
spring.datasource.username=<DB_USER>
spring.datasource.password=<DB_PASS>
```

- Press `Esc`, then type `:wq` and press `Enter` to save and exit.

| Placeholder | Replace With |
|---|---|
| `<DB_HOST>` | Private or public IP of your MariaDB server |
| `<DB_NAME>` | Your database name |
| `<DB_USER>` | Your database username |
| `<DB_PASS>` | Your database password |

---

## Step 5 — Build the Spring Boot Application

Run Maven to compile and package the application into a JAR file:

```bash
mvn clean package
```

This creates a JAR file inside the `target/` directory:

```
target/spring-backend-v1.jar
```

> The `-DskipTests` flag can be added to skip tests during build if needed:
> ```bash
> mvn clean package -DskipTests
> ```

---

## Step 6 — Run the Application

### Run directly (foreground — for testing)

```bash
java -jar target/spring-backend-v1.jar
```

The application will start and be accessible at:

```
http://<EC2_PUBLIC_IP>:8080
```

---

## Step 7 — Keep the Application Running in Background

Use `nohup` to keep the app running after you close the SSH session:

```bash
nohup java -jar target/spring-backend-v1.jar > app.log 2>&1 &
```

- Output logs are saved to `app.log`
- The `&` runs the process in the background

### Check if the app is running

```bash
ps aux | grep spring-backend-v1.jar
```

### View live logs

```bash
tail -f app.log
```

### Stop the application

```bash
# Find the process ID (PID)
ps aux | grep spring-backend-v1.jar

# Kill the process
kill <PID>
```

---

## (Optional) Step 8 — Run as a systemd Service

For production, it's better to manage the app as a systemd service so it restarts automatically on reboot.

### Create a service file

```bash
vim /etc/systemd/system/springboot.service
```

Add the following content (update paths as needed):

```ini
[Unit]
Description=Spring Boot Backend
After=network.target

[Service]
User=root
WorkingDirectory=/root/<your-repo>/backend
ExecStart=/usr/bin/java -jar target/spring-backend-v1.jar
SuccessExitStatus=143
Restart=always
RestartSec=10
StandardOutput=append:/var/log/springboot.log
StandardError=append:/var/log/springboot.log

[Install]
WantedBy=multi-user.target
```

### Enable and start the service

```bash
systemctl daemon-reload
systemctl enable springboot
systemctl start springboot
```

### Check service status

```bash
systemctl status springboot
```

### View logs

```bash
tail -f /var/log/springboot.log
```

---

## Quick Reference — All Commands in Order

```bash
# 1. Clone repo
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>/backend

# 2. Install Java 17
apt update && apt install openjdk-17-jdk -y

# 3. Install Maven
apt install maven -y

# 4. Configure DB credentials
vim src/main/resources/application.properties

# 5. Build the JAR
mvn clean package -DskipTests

# 6. Run in background with nohup
nohup java -jar target/spring-backend-v1.jar > app.log 2>&1 &

# 7. Check logs
tail -f app.log
```

---

## Troubleshooting

| Issue | Fix |
|---|---|
| `java: command not found` | Run `apt install openjdk-17-jdk -y` |
| `mvn: command not found` | Run `apt install maven -y` |
| `Cannot connect to DB` | Check `<DB_HOST>`, credentials in `application.properties`, and DB port 3306 is open |
| App not accessible on browser | Ensure port **8080** is open in EC2 Security Group inbound rules |
| Build fails | Try `mvn clean package -DskipTests` to skip test errors |
| App stops after SSH logout | Use `nohup` or set up a systemd service |
| Port 8080 already in use | Run `kill $(lsof -t -i:8080)` to free the port |

---

*Guide covers: Java 17 · Maven · Spring Boot JAR build · nohup background run · systemd service setup*