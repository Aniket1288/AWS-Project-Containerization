# React Frontend Deployment Guide (Windows → AWS EC2)

A step-by-step guide to deploying a React (Vite) frontend on an AWS EC2 instance using Apache2.

---

## Prerequisites

- AWS EC2 instance running Ubuntu
- Your React project pushed to GitHub
- Port **80 (HTTP)** open in EC2 Security Group inbound rules
- SSH access to your EC2 instance

---

## Step 1 — Clone Your GitHub Repo

SSH into your EC2 instance, then clone your project:

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

> Replace `<your-username>` and `<your-repo>` with your actual GitHub values.

---

## Step 2 — Install Node.js and npm

```bash
apt update && apt install nodejs npm -y
```

### Verify Installation

```bash
node -v
npm -v
```

---

## Step 3 — Install Project Dependencies

Inside your project folder, run:

```bash
npm install
```

This reads `package.json` and installs all required packages into `node_modules/`. This may take 1–2 minutes.

---

## Step 4 — Configure the `.env` File

Open the `.env` file and set your backend EC2 public IP:

```bash
vim .env
```

Inside vim:
- Press `i` to enter **insert mode**
- Add the following line:

```env
VITE_API_URL="http://<BACKEND_PUBLIC_IP>:8080/api"
```

- Press `Esc`, then type `:wq` and press `Enter` to save and exit.

> Replace `<BACKEND_PUBLIC_IP>` with the actual public IPv4 address of your backend EC2 instance.

---

## Step 5 — Build the React App for Production

```bash
npm run build
```

This creates a `dist/` directory containing optimized, production-ready files.

---

## Step 6 — Deploy to Apache2

### Install and start Apache2

```bash
apt install apache2 -y
systemctl start apache2
```

### Copy build files to Apache's web root

```bash
cp -rf dist/* /var/www/html/
```

### (Recommended) Enable Apache to start on reboot

```bash
systemctl enable apache2
```

---

## Accessing the Application

```
http://<EC2_PUBLIC_IP>:80
```

> Make sure port **80** is open in your EC2 Security Group inbound rules.

---

## Fix React Router (SPA) — Prevent 404 on Page Refresh

If your app uses `react-router-dom`, create an `.htaccess` file so all routes point to `index.html`:

```bash
vim /var/www/html/.htaccess
```

Add the following content:

```apache
Options -MultiViews
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.html [QSA,L]
```

Then enable the Apache rewrite module and restart:

```bash
a2enmod rewrite
systemctl restart apache2
```

---

## Quick Reference — All Commands in Order

```bash
# 1. Clone repo
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

# 2. Install Node.js
apt update && apt install nodejs npm -y

# 3. Install dependencies
npm install

# 4. Set backend URL in .env
vim .env
# VITE_API_URL="http://<BACKEND_PUBLIC_IP>:8080/api"

# 5. Build production bundle
npm run build

# 6. Install and start Apache2
apt install apache2 -y
systemctl start apache2
systemctl enable apache2

# 7. Deploy files
cp -rf dist/* /var/www/html/

# 8. (Optional) Fix React Router
vim /var/www/html/.htaccess
a2enmod rewrite && systemctl restart apache2
```

---

## Troubleshooting

| Issue | Fix |
|---|---|
| App not loading on browser | Check port 80 is open in EC2 Security Group |
| API calls failing | Verify `VITE_API_URL` in `.env` has the correct backend IP |
| 404 on page refresh | Add `.htaccess` file and enable `mod_rewrite` |
| `npm install` fails | Try `sudo npm install` or check Node.js version |
| Backend not responding | Ensure port 8080 is open in the **backend** EC2 Security Group |

---

*Guide covers: Node.js setup · npm install · Vite build · Apache2 deployment · React Router SPA fix*