# Module 9: Nginx with a Python Flask App

This lab shows you how to put Nginx in front of a Python Flask web application as a **reverse proxy**.

---

## Files

- `app.py`: The Python Flask app.
- `requirements.txt`: Python packages (`flask` and `gunicorn`).
- `flaskapp.conf`: The Nginx configuration that sends traffic to Flask.

---

## Step-by-Step Instructions

### Step 1: Install Python tools and dependencies
```bash
sudo apt install python3-venv -y

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Step 2: Start the Flask app with Gunicorn
```bash
gunicorn -b 127.0.0.1:5000 app:app
```
> *Leave this running in this terminal. Open a **second terminal** to continue.*

### Step 3: Test Flask directly (2nd terminal)
```bash
curl http://127.0.0.1:5000
```
> Expected output: `Hello from Flask behind Nginx!`

### Step 4: Configure Nginx to forward traffic to Flask
```bash
# Copy the config file
sudo cp flaskapp.conf /etc/nginx/sites-available/flaskapp

# Turn the site ON
sudo ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled/

# Test syntax and reload Nginx
sudo nginx -t
sudo systemctl reload nginx

# Tell your computer the domain name flask.local
echo '127.0.0.1  flask.local' | sudo tee -a /etc/hosts
```

### Step 5: Open the app through Nginx on Port 80
```bash
curl http://flask.local
```
> Expected output: `Hello from Flask behind Nginx!`
