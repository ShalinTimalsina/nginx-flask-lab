# Nginx for Beginners: Complete Course Lab

[![Nginx](https://img.shields.io/badge/Nginx-Web%20Server-009639?logo=nginx&logoColor=white)](https://nginx.org)
[![Python](https://img.shields.io/badge/Python-Flask-3776AB?logo=python&logoColor=white)](https://python.org)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20%7C%2024.04-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com)

Welcome to the hands-on repository for the **Nginx for Beginners** course!  
This repository is designed so learners can read through the course document [Nginx.md](Nginx.md) and follow along directly with the code in each corresponding folder.

---

## Course Roadmap & Folder Structure

The repository is organized to match the hands-on modules in [Nginx.md](Nginx.md):

```text
nginx-flask-lab/
├── README.md                      # This quickstart guide
├── Nginx.md                       # Complete course theory & lesson notes
├── .gitignore                     # Git ignore rules for Python virtual environments & logs
│
├── static-website/                # Module 8: Hosting a Static Website
│   ├── index.html                 # Simple HTML landing page
│   ├── mysite.conf                # Nginx server block for mysite.local
│   └── README.md                  # Lab instructions for Module 8
│
├── flask-reverse-proxy/           # Module 9: Nginx with a Python Flask App
│   ├── app.py                     # Simple 6-line Flask application
│   ├── requirements.txt           # Python dependencies (flask, gunicorn)
│   ├── flaskapp.conf              # Nginx reverse proxy configuration for flask.local
│   └── README.md                  # Lab instructions for Module 9
│
└── upstream-servers/              # Module 10: Upstream Servers (Ports 3000 & 3001)
    ├── app1/                      # App One test page
    │   └── index.html
    ├── app2/                      # App Two test page
    │   └── index.html
    ├── myapps.conf                # Nginx configuration routing /app1/ and /app2/
    └── README.md                  # Lab instructions for Module 10
```

---

## Prerequisites (Ubuntu 22.04 / 24.04)

Install Nginx and Python:
```bash
sudo apt update
sudo apt install nginx python3-venv python3-pip curl -y
```

Check that Nginx is running:
```bash
systemctl status nginx
curl http://localhost
```

---

## Lab 1: Host a Static Website (Module 8)

Inside [`static-website/`](static-website/):

```bash
cd static-website

# 1. Create document folder and copy HTML
sudo mkdir -p /var/www/mysite/html
sudo cp index.html /var/www/mysite/html/

# 2. Copy Nginx virtual host config
sudo cp mysite.conf /etc/nginx/sites-available/mysite

# 3. Enable the site with a symlink
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/

# 4. Test syntax and reload Nginx
sudo nginx -t
sudo systemctl reload nginx

# 5. Map mysite.local on your machine
echo '127.0.0.1  mysite.local' | sudo tee -a /etc/hosts

# 6. Test the website!
curl http://mysite.local
```
> Output: `<h1>Hello from Nginx!</h1>`

---

## Lab 2: Nginx with Python Flask (Module 9)

Inside [`flask-reverse-proxy/`](flask-reverse-proxy/):

```
Browser  ---- request (Port 80) ---->  Nginx (Reverse Proxy)
                                             │
                                   proxy_pass http://127.0.0.1:5000
                                             │
                                             ▼
                                  Flask App (Port 5000)
```

```bash
cd flask-reverse-proxy

# 1. Set up virtual environment and install Flask + Gunicorn
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 2. Run Flask with Gunicorn on port 5000
gunicorn -b 127.0.0.1:5000 app:app
```
*(Keep this terminal open, and open a **second terminal** for the next commands)*:

```bash
# 3. Test Flask directly (2nd terminal)
curl http://127.0.0.1:5000

# 4. Copy Nginx reverse proxy config
sudo cp flaskapp.conf /etc/nginx/sites-available/flaskapp
sudo ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled/

# 5. Test syntax and reload Nginx
sudo nginx -t
sudo systemctl reload nginx

# 6. Map flask.local in /etc/hosts
echo '127.0.0.1  flask.local' | sudo tee -a /etc/hosts

# 7. Test Flask through Nginx on Port 80!
curl http://flask.local
```
> Output: `Hello from Flask behind Nginx!`

---

## Lab 3: Upstream Servers on Ports 3000 & 3001 (Module 10)

Inside [`upstream-servers/`](upstream-servers/):

```
                      ┌──> /app1/ ──> App One (Port 3000)
Browser ──> Nginx :80 │
                      └──> /app2/ ──> App Two (Port 3001)
```

```bash
cd upstream-servers

# 1. Start App 1 and App 2 in the background
python3 -m http.server 3000 --directory app1 &
python3 -m http.server 3001 --directory app2 &

# 2. Verify backend apps directly
curl http://localhost:3000
curl http://localhost:3001

# 3. Copy and enable Nginx upstream config
sudo cp myapps.conf /etc/nginx/sites-available/myapps
sudo ln -s /etc/nginx/sites-available/myapps /etc/nginx/sites-enabled/

# 4. Test syntax and reload Nginx
sudo nginx -t
sudo systemctl reload nginx

# 5. Access both apps through Nginx on Port 80!
curl http://localhost/app1/
curl http://localhost/app2/

# 6. Stop background test apps when done
pkill -f http.server
```

---

## Common Commands & Troubleshooting

| Problem | Diagnosis Command | Fix |
| --- | --- | --- |
| **Typo in Nginx file** | `sudo nginx -t` | Check line number reported by test |
| **502 Bad Gateway** | `curl http://127.0.0.1:5000` | Start Gunicorn backend |
| **403 Forbidden** | `ls -la /var/www/mysite/html` | Fix folder permissions or check `index.html` |
| **Service status** | `systemctl status nginx` | Start service with `sudo systemctl start nginx` |
| **View recent logs** | `sudo tail -f /var/log/nginx/error.log` | Inspect error details |
