# Nginx for Beginners: Course Content

*DevOps Training · Web Server Module · Ubuntu 22.04 / 24.04*

## Syllabus at a Glance

| # | Topic | What students will be able to do |
| --- | --- | --- |
| 1 | Introduction to Web Servers | Explain what a web server does |
| 2 | Nginx vs Apache2 | Choose the right server for a job |
| 3 | Installation | Install Nginx and verify it runs |
| 4 | systemctl and journalctl | Start, enable and debug services |
| 5 | nginx.conf explained | Read the main configuration file |
| 6 | sites-available and sites-enabled | Host a site using virtual hosts |
| 7 | Symlinks, inodes, soft vs hard links | Understand why Nginx uses symlinks |
| 8 | Static website | Serve HTML files |
| 9 | Nginx with Python Flask | Put Nginx in front of a Python app |
| 10 | Upstream servers (ports 3000 and 3001) | Route traffic to different apps |

---

## 1. Introduction to Web Servers

A **web server** is a program that waits for requests from browsers and sends back responses (HTML pages, images, or data).

```
Browser  ---- request: GET /index.html ---->  Web Server
Browser  <--- response: 200 OK + HTML -----  Web Server
```

Every website you visit is handled this way. A web server can do two jobs:

- **Static content:** send files exactly as they are on disk (HTML, CSS, images).
- **Dynamic content:** pass the request to an application (Python, Node.js, PHP) and send back whatever the app produces.

> **Key idea:** Web servers listen on a **port**. HTTP uses port **80**, HTTPS uses port **443**.

---

## 2. Nginx and Apache2

The two most popular web servers. Both are free and open source.

|  | **Nginx** | **Apache2** |
| --- | --- | --- |
| How it works | Event-driven: one worker handles thousands of connections | Process/thread based: one thread per connection |
| Memory use | Low | Higher under heavy load |
| Best for | High traffic, reverse proxy, load balancer | Flexible setups, PHP, `.htaccess` per-folder rules |
| Main config | `/etc/nginx/nginx.conf` | `/etc/apache2/apache2.conf` |
| Sites folders | `sites-available/`, `sites-enabled/` | `sites-available/`, `sites-enabled/` |
| Logs | `/var/log/nginx/` | `/var/log/apache2/` |

> **Simple way to remember:** Apache is like a restaurant that gives every guest their own waiter. Nginx is like one very fast waiter serving many tables at once.

In this course we focus on **Nginx**.

---

## 3. Installing Nginx

**Step 1: Update the package list**

```bash
sudo apt update
```

**Step 2: Install Nginx**

```bash
sudo apt install nginx -y
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `sudo` | Run the command as administrator |
| `apt` | Ubuntu's package manager (installs software) |
| `install` | Tell apt to install a package |
| `nginx` | The name of the package |
| `-y` | Short for **yes**: answers the 'Do you want to continue?' question automatically |

**Step 3: Check the version**

```bash
nginx -v
```

`-v` stands for **v**ersion. It prints the installed version and exits.

Expected output: `nginx version: nginx/1.x.x`

**Step 4: Allow web traffic through the firewall** (only if `ufw` is turned on)

```bash
sudo ufw allow 'Nginx Full'
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `ufw` | Ubuntu's firewall tool |
| `allow` | Let traffic through |
| `'Nginx Full'` | A ready-made rule that opens port 80 (HTTP) and port 443 (HTTPS) |

**Step 5: Open the default page**

```bash
curl http://localhost
```

Expected output: HTML text containing **Welcome to nginx!**. You can also open `http://localhost` in a browser.

---

## 4. Managing Nginx with systemctl and journalctl

On Ubuntu, every service is controlled by **systemd**. The tool to talk to it is `systemctl`.

### systemctl: control the service

| Command | Meaning |
| --- | --- |
| `sudo systemctl start nginx` | Start now |
| `sudo systemctl stop nginx` | Stop now |
| `sudo systemctl restart nginx` | Stop, then start (drops connections) |
| `sudo systemctl reload nginx` | Re-read config without dropping connections |
| `sudo systemctl enable nginx` | Start automatically at boot |
| `sudo systemctl disable nginx` | Do not start at boot |
| `systemctl status nginx` | Show current state |

> **start vs enable:** `start` means *run it now*. `enable` means *run it every time the machine boots*. They are independent, so you usually need both.

Shortcut: `sudo systemctl enable --now nginx` does both in one command.

`--now` is a flag that also **starts the service right now**. Without it, `enable` only takes effect at the next boot.

### journalctl: read service logs

systemd stores the logs of every service in a **journal**. `journalctl` reads it.

```bash
journalctl -u nginx              # all logs of nginx
journalctl -u nginx -n 20        # last 20 lines
journalctl -u nginx -f           # follow live (like tail -f)
journalctl -u nginx --since today
journalctl -u nginx -p err       # only errors
```

**Flag breakdown**

| Flag | Meaning |
| --- | --- |
| `-u nginx` | **u**nit: show logs for this service only |
| `-n 20` | **n**umber of lines: show only the last 20 |
| `-f` | **f**ollow: keep running and show new lines as they appear (press `Ctrl+C` to stop) |
| `--since today` | Show only logs from today |
| `-p err` | **p**riority: show only errors and worse |

Use it first when a service **fails to start**.

### Nginx's own log files

| File | Contains |
| --- | --- |
| `/var/log/nginx/access.log` | Every request received |
| `/var/log/nginx/error.log` | Problems and failures |

```bash
sudo tail -f /var/log/nginx/access.log
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `tail` | Show the end of a file |
| `-f` | **f**ollow: keep watching and print new lines live |

### Common HTTP status codes

| Code | Meaning | Typical cause |
| --- | --- | --- |
| 200 | OK | Everything worked |
| 301 | Moved Permanently | Redirect |
| 403 | Forbidden | Wrong file permissions |
| 404 | Not Found | File or path does not exist |
| 500 | Internal Server Error | Application crashed |
| 502 | Bad Gateway | Backend app is down or on the wrong port |

---

## 5. The Nginx Configuration File

Main file: **`/etc/nginx/nginx.conf`**

This is a simplified version of the Ubuntu default:

```nginx
user www-data;                  # Linux user Nginx runs as
worker_processes auto;          # one worker per CPU core
pid /run/nginx.pid;

events {
    worker_connections 768;     # connections each worker can handle
}

http {                          # everything about web traffic lives here
    sendfile on;                # efficient file sending
    include /etc/nginx/mime.types;   # maps .html, .css, .png to content types

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    gzip on;                    # compress responses to save bandwidth

    include /etc/nginx/conf.d/*.conf;       # extra config files
    include /etc/nginx/sites-enabled/*;     # <-- your websites are loaded here
}
```

### Understanding the structure

Nginx config is made of **blocks** (`{ }`) called **contexts**, and **directives** (lines ending with `;`).

```
main
 ├── events { }
 └── http { }
       └── server { }          <- one website (virtual host)
             └── location { }  <- one URL path
```

> **Remember:** the last line `include /etc/nginx/sites-enabled/*;` is what connects the main file to your websites.

---

## 6. sites-available and sites-enabled

One server can host many websites. Each website gets its own config file (a **virtual host**).

| Folder | Purpose |
| --- | --- |
| `/etc/nginx/sites-available/` | Stores **all** site configs, active or not |
| `/etc/nginx/sites-enabled/` | Holds only the sites that are **turned on** |

Nginx reads **only** `sites-enabled/`.

```
sites-available/            sites-enabled/
 ├── mysite     <-----------  mysite  (link)
 ├── blog
 └── shop       <-----------  shop    (link)
```

**Enable a site:** create a link. **Disable a site:** remove the link. The original file stays safe.

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/mysite      # disables it
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `ln` | Make a link |
| `-s` | **s**ymbolic: make a soft link (a shortcut). Without `-s` you get a hard link |
| first path | The original file (the target) |
| second path | Where the link will be created |
| `rm` | Remove a file. On a link it removes only the link, not the original |

---

## 7. Why Symbolic Links? Inodes, Soft Links and Hard Links

### Why use a symlink for sites-enabled?

- **One real copy:** the config lives only in `sites-available`. No duplicate files to keep in sync.
- **Easy on/off:** add or remove a link without touching the config.
- **Safe:** deleting the link never deletes your config.

### What is an inode?

On Linux, a file has two separate parts:

- **The name** (what you see in `ls`)
- **The inode** (a record holding the file's details: owner, permissions, size, timestamps, and where the data sits on disk)

A directory is simply a list that maps **names to inode numbers**.

```bash
ls -li                  # -i shows the inode number
stat myfile.txt         # full inode details
```

**Flag breakdown**

| Part | Meaning |
| --- | --- |
| `ls` | List files |
| `-l` | **l**ong format: show permissions, owner, size and date |
| `-i` | **i**node: show each file's inode number |
| `stat` | Show all the details stored in a file's inode |

### Hard link vs soft (symbolic) link

```
HARD LINK                              SOFT (SYMBOLIC) LINK

 name: file.txt ---+                    name: file.txt ----> inode 100 ----> DATA
                   +--> inode 100 --> DATA
 name: hard.txt ---+                    name: soft.txt ----> inode 205 (stores the
                                                            PATH text 'file.txt')
 Two names, SAME inode                  Soft link has its OWN inode and just
                                        points to the name
```

Try it:

```bash
echo 'hello' > file.txt
ln file.txt hard.txt           # hard link
ln -s file.txt soft.txt        # soft link
ls -li
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `>` | Send the output into a file (replaces the old content) |
| `ln file.txt hard.txt` | No flag, so this makes a **hard** link |
| `ln -s file.txt soft.txt` | `-s` makes a **soft** link |
| `ls -li` | List files in long format (`-l`) with inode numbers (`-i`) |

The output shows `file.txt` and `hard.txt` with the **same inode number**, while `soft.txt` has a different one and shows `soft.txt -> file.txt`.

|  | **Hard link** | **Soft link** |
| --- | --- | --- |
| Command | `ln file link` | `ln -s file link` |
| Inode | Same as original | Its own new inode |
| Original deleted | Still works (data stays) | **Breaks** (dangling link) |
| Works across disks | No | Yes |
| Can link a folder | No | Yes |
| Can point to a path | No | Yes |

**Test the difference:**

```bash
rm file.txt
cat hard.txt      # prints: hello   (data still exists)
cat soft.txt      # error: No such file or directory
```

> **Why Nginx uses soft links:** they are easy to see (`ls -l` shows the arrow), they can point to a file anywhere, and a broken link clearly tells you the config is gone.

---

## 8. Hosting a Static Website

A **static site** is plain HTML files served as-is.

**Step 1: Create the site folder and a page**

```bash
sudo mkdir -p /var/www/mysite/html
echo '<h1>Hello from Nginx!</h1>' | sudo tee /var/www/mysite/html/index.html
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `mkdir` | **M**a**k**e a **dir**ectory (folder) |
| `-p` | **p**arents: also create any missing folders in the path (`/var/www/mysite/html` all at once) |
| `echo` | Print text |
| `tee` | Write text into a file (needed here because `sudo` does not apply to the `>` symbol) |

The `|` symbol is called a **pipe**. It sends the output of the command on its left into the command on its right.

The folder holds your website files. The second command writes one HTML page into it.

**Step 2: Create the Nginx file**

```bash
sudo nano /etc/nginx/sites-available/mysite
```

Paste this, then save (`Ctrl+O`, `Enter`) and exit (`Ctrl+X`):

```nginx
server {
    listen 80;
    server_name mysite.local;
    root /var/www/mysite/html;
    index index.html;
}
```

| Line | Plain meaning |
| --- | --- |
| `listen 80` | Accept visitors on port 80 |
| `server_name` | The website name this file answers to |
| `root` | The folder where the files live |
| `index` | The page to show first |

**Step 3: Turn the site on**

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
```

**Step 4: Test the file, then reload Nginx**

```bash
sudo nginx -t
sudo systemctl reload nginx
```

`-t` stands for **t**est. Nginx checks your files for mistakes and changes nothing.

Expected output from the test: `syntax is ok` and `test is successful`.

**Step 5: Tell your computer the name `mysite.local`**

```bash
echo '127.0.0.1  mysite.local' | sudo tee -a /etc/hosts
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `tee` | Write text into a file |
| `-a` | **a**ppend: add to the end of the file instead of replacing everything |
| `/etc/hosts` | The file that maps names to IP addresses on your machine |

This says: the name `mysite.local` means this machine (`127.0.0.1`).

**Step 6: Open the site**

```bash
curl http://mysite.local
```

Expected output: `<h1>Hello from Nginx!</h1>`

> **Golden rule:** run `sudo nginx -t` before every reload. It catches typos before they take your site down.

---

## 9. Nginx with a Python Flask App

Flask can serve pages by itself, but it is not built to face the internet. The standard setup is:

```
Browser --> Nginx (port 80) --> Gunicorn / Flask (port 5000)
            reverse proxy        the application
```

Nginx acts as a **reverse proxy**: it receives the request and forwards it to the app behind it.

**Step 1: Install the Python tool for virtual environments**

```bash
sudo apt install python3-venv -y
```

A virtual environment gives your project its own private Python packages.

**Step 2: Create the project folder and environment**

```bash
mkdir ~/flaskapp
cd ~/flaskapp
python3 -m venv venv
source venv/bin/activate
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `mkdir` | Make a folder |
| `cd` | **C**hange **d**irectory: move into that folder |
| `python3 -m venv venv` | `-m` means run a Python **m**odule as a program. The module is `venv`, and the last word `venv` is the name of the new environment folder |
| `source venv/bin/activate` | Switch your terminal into the virtual environment |

Expected result: your prompt now starts with `(venv)`.

**Step 3: Install Flask and Gunicorn**

```bash
pip install flask gunicorn
```

Flask is the app framework. Gunicorn is the server that runs the Flask app.

**Step 4: Write the app**

```bash
nano app.py
```

Paste this, then save (`Ctrl+O`, `Enter`) and exit (`Ctrl+X`):

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return 'Hello from Flask behind Nginx!'
```

**Step 5: Start the app**

```bash
gunicorn -b 127.0.0.1:5000 app:app
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `gunicorn` | The server that runs the Flask app |
| `-b 127.0.0.1:5000` | **b**ind: listen on this address and port. `127.0.0.1` means only this machine can reach it |
| `app:app` | `file name`:`variable name` (file `app.py`, variable `app`) |

`app:app` means: file `app.py`, variable `app`. This terminal is now busy running the app, so **open a second terminal** for the next steps. To stop the app later, press `Ctrl+C` here.

**Step 6: Check the app directly (second terminal)**

```bash
curl http://127.0.0.1:5000
```

Expected output: `Hello from Flask behind Nginx!`

**Step 7: Create the Nginx file**

```bash
sudo nano /etc/nginx/sites-available/flaskapp
```

Paste this, then save and exit:

```nginx
server {
    listen 80;
    server_name flask.local;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

| Line | Plain meaning |
| --- | --- |
| `proxy_pass` | Send the request to the Flask app on port 5000 |
| `Host` | Tell the app which website name was asked for |
| `X-Real-IP` | Tell the app the visitor's real IP address |

**Step 8: Turn the site on**

```bash
sudo ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled/
```

**Step 9: Test the file, then reload Nginx**

```bash
sudo nginx -t
sudo systemctl reload nginx
```

`-t` stands for **t**est. Nginx checks your files for mistakes and changes nothing.

**Step 10: Tell your computer the name `flask.local`**

```bash
echo '127.0.0.1  flask.local' | sudo tee -a /etc/hosts
```

`tee -a` adds the line to the end of `/etc/hosts` (`-a` = **a**ppend).

**Step 11: Open the app through Nginx**

```bash
curl http://flask.local
```

Expected output: `Hello from Flask behind Nginx!` This time the answer came through Nginx on port 80.

> **Seeing 502 Bad Gateway?** Gunicorn is not running, or it is on a different port than `proxy_pass`.

---

## 10. Upstream Servers: Different Apps on Ports 3000 and 3001

An **upstream** is a named backend that Nginx can send traffic to. It is useful when one server runs **several apps**, each on its own port, and you want **one entry point** (port 80).

```
                      +--> /app1/ --> App One (port 3000)
Browser --> Nginx :80 |
                      +--> /app2/ --> App Two (port 3001)
```

**Step 1: Create two small apps**

```bash
mkdir ~/app1 ~/app2
echo 'This is App One' > ~/app1/index.html
echo 'This is App Two' > ~/app2/index.html
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `mkdir` | Make a folder (two folders here) |
| `~` | Your home folder (for example `/home/ubuntu`) |
| `echo` | Print text |
| `>` | Send the text into a file instead of the screen |

Each app is just a folder with one page inside.

**Step 2: Start App One on port 3000**

```bash
python3 -m http.server 3000 --directory ~/app1 &
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `python3 -m http.server` | `-m` runs a Python **m**odule as a program. `http.server` is a built-in mini web server |
| `3000` | The port number to listen on |
| `--directory ~/app1` | Serve the files from this folder |
| `&` | Run in the background so the terminal stays free |

The `&` at the end runs it in the background, so you can keep using the same terminal.

**Step 3: Start App Two on port 3001**

```bash
python3 -m http.server 3001 --directory ~/app2 &
```

**Step 4: Check both apps directly**

```bash
curl http://localhost:3000
curl http://localhost:3001
```

Expected output: `This is App One` and `This is App Two`. If you see both, the apps are running.

**Step 5: Create the Nginx file**

```bash
sudo nano /etc/nginx/sites-available/myapps
```

Paste this, then save (`Ctrl+O`, `Enter`) and exit (`Ctrl+X`):

```nginx
upstream app_one {
    server 127.0.0.1:3000;
}

upstream app_two {
    server 127.0.0.1:3001;
}

server {
    listen 80;
    server_name localhost;

    location /app1/ {
        proxy_pass http://app_one/;
    }

    location /app2/ {
        proxy_pass http://app_two/;
    }
}
```

| Part | Plain meaning |
| --- | --- |
| `upstream app_one` | A name for the app running on port 3000 |
| `upstream app_two` | A name for the app running on port 3001 |
| `location /app1/` | If the URL starts with /app1/ ... |
| `proxy_pass http://app_one/` | ... send the request to App One |

**Step 6: Turn the site on**

```bash
sudo ln -s /etc/nginx/sites-available/myapps /etc/nginx/sites-enabled/
```

**Step 7: Test the file, then reload Nginx**

```bash
sudo nginx -t
sudo systemctl reload nginx
```

`-t` stands for **t**est. Nginx checks your files for mistakes and changes nothing.

Expected output from the test: `syntax is ok` and `test is successful`.

**Step 8: Open both apps through Nginx**

```bash
curl http://localhost/app1/
curl http://localhost/app2/
```

Expected output: `This is App One` and `This is App Two`. One address (port 80), two different apps.

**Step 9: Clean up**

```bash
pkill -f http.server
```

**Command breakdown**

| Part | Meaning |
| --- | --- |
| `pkill` | Stop programs by name |
| `-f` | **f**ull command line: match against the whole command, so `http.server` is found inside `python3 -m http.server ...` |

This stops both test apps.

> **Trailing slash matters:** `proxy_pass http://app_one/;` ends with `/`, which removes `/app1/` from the path before forwarding. Without it, the app would receive `/app1/` and likely return 404.

### Bonus: Load balancing with upstream

If both ports run the **same** app, put them in one upstream and Nginx shares the traffic (round-robin by default):

```nginx
upstream backend {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

If one server goes down, Nginx automatically sends traffic to the other.

---

## Troubleshooting Cheat Sheet

| Problem | Check |
| --- | --- |
| Nginx will not start | `sudo nginx -t` then `journalctl -u nginx -n 20` |
| Page shows default Nginx welcome | Site not linked, or `default` site still enabled |
| 403 Forbidden | File permissions and the `root` path |
| 404 Not Found | `root` path, file name, `location` block |
| 502 Bad Gateway | Is the backend running? Is the port correct? |
| Config change has no effect | Did you run `systemctl reload nginx`? |

---

## Practice Exercises

1. Install Nginx, then stop it, disable it, and enable it again. Check each state with `systemctl status`.
2. Break `nginx.conf` on purpose (remove a `;`) and use `nginx -t` and `journalctl` to find the error.
3. Create two static sites, `site1.local` and `site2.local`, on the same server.
4. Create a file, make a hard link and a soft link, delete the original, and explain what happens to each.
5. Run a Flask app on port 5000 and serve it through Nginx.
6. Run two different apps on ports 3000 and 3001 and reach both through one Nginx server.
7. Stop one app and observe the 502 error in `access.log` and `error.log`.
