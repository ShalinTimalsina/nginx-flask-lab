# Upstream Servers (Ports 3000 and 3001)

This lab shows you how to route traffic to two different apps from one single port 80 domain, and how to set up **round-robin load balancing**.

---

## Files

- `app1/index.html`: Web page for App One (`This is App One`).
- `app2/index.html`: Web page for App Two (`This is App Two`).
- `myapps.conf`: Upstream config that routes `/app1/` to port 3000 and `/app2/` to port 3001.
- `load-balancer.conf`: Bonus upstream config that shares traffic between both ports (round-robin).

---

## Lab A: Path-Based Routing (/app1/ and /app2/)

```
                      ┌──> /app1/ ──> App One (Port 3000)
Browser ──> Nginx :80 │
                      └──> /app2/ ──> App Two (Port 3001)
```

### Step 1: Start App 1 and App 2 in the background
```bash
python3 -m http.server 3000 --directory app1 &
python3 -m http.server 3001 --directory app2 &
```

### Step 2: Check both apps directly
```bash
curl http://localhost:3000
curl http://localhost:3001
```

### Step 3: Configure Nginx
```bash
# Copy the config
sudo cp myapps.conf /etc/nginx/sites-available/myapps

# Turn the site ON
sudo ln -s /etc/nginx/sites-available/myapps /etc/nginx/sites-enabled/

# Test syntax and reload Nginx
sudo nginx -t
sudo systemctl reload nginx
```

### Step 4: Open both apps through Nginx (Port 80)
```bash
curl http://localhost/app1/
curl http://localhost/app2/
```

---

## Lab B: Bonus Load Balancing (Round-Robin)

Instead of routing by URL path, Nginx shares incoming requests evenly across both servers:

```
Browser ──> Nginx :80 ──> Upstream backend ──┬──> App One (Port 3000)
                                              └──> App Two (Port 3001)
```

### Step 1: Enable the load balancer config
```bash
# Copy and enable the config
sudo cp load-balancer.conf /etc/nginx/sites-available/loadbalancer
sudo ln -s /etc/nginx/sites-available/loadbalancer /etc/nginx/sites-enabled/

# Test syntax and reload
sudo nginx -t
sudo systemctl reload nginx
```

### Step 2: Test round-robin distribution
Make multiple requests and watch Nginx alternate between App One and App Two:
```bash
for i in {1..4}; do curl http://localhost; echo ""; done
```

---

## Cleanup
Stop background test servers when finished:
```bash
pkill -f http.server
```
