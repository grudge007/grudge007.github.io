---
layout: post
title: Setting Up Nginx as a Reverse Proxy with Load Balancing
date: 21-02-2025
categories: [technology basics, reverse-proxy, load balancer]
tags: [software, configuration, beginners guide, nginx]
---

## **Overview**
This guide explains how to configure Nginx as a reverse proxy with load balancing to distribute traffic between two backend servers: `192.168.1.100` and `192.168.1.101`.

A reverse proxy with load balancing improves performance, distributes traffic evenly, and provides failover protection.

---

## **Step 1: Install Nginx**

### **Ubuntu/Debian**
```bash
sudo apt update && sudo apt install nginx -y
```

### **CentOS/RHEL**
```bash
sudo yum install epel-release -y
sudo yum install nginx -y
```

Once installed, verify that Nginx is running:
```bash
sudo systemctl status nginx
```
If it is not running, start it:
```bash
sudo systemctl start nginx
sudo systemctl enable nginx  # Enables it to start on boot
```

---

## **Step 2: Configure Nginx as a Reverse Proxy with Load Balancing**

Nginx configurations are stored in different locations depending on the OS:
- **Ubuntu/Debian**: `/etc/nginx/sites-available/`
- **CentOS/RHEL**: `/etc/nginx/conf.d/`

### **Create a Load Balancing Configuration**
Edit the configuration file:

#### **Ubuntu/Debian**
```bash
sudo nano /etc/nginx/sites-available/loadbalancer
```

#### **CentOS/RHEL**
```bash
sudo nano /etc/nginx/conf.d/loadbalancer.conf
```

Add the following content:
```nginx
upstream backend_servers {
    least_conn;  # Load balancing method: Sends traffic to the server with the least connections

    server 192.168.1.100;
    server 192.168.1.101;
}

server {
    listen 80;
    server_name _;  # Accepts all requests

    location / {
        proxy_pass http://backend_servers;  # Forward requests to the backend servers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### **Explanation:**
- `listen 80;` → Listens for HTTP traffic on port 80.
- `server_name _;` → Catches all incoming requests regardless of domain or IP.
- `upstream backend_servers {}` → Defines a group of backend servers.
- `least_conn;` → Distributes traffic to the server with the fewest active connections.
- `proxy_pass http://backend_servers;` → Forwards requests to the backend servers.
- `proxy_set_header` → Ensures the backend servers see the correct request headers.

---

## **Step 3: Enable the Configuration**

### **Ubuntu/Debian:**
Enable the new site and disable the default one:
```bash
sudo ln -s /etc/nginx/sites-available/loadbalancer /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
```

### **CentOS/RHEL:**
No symlink is needed; just ensure that the configuration file exists in `/etc/nginx/conf.d/`.

---

## **Step 4: Test & Restart Nginx**

Before restarting Nginx, test the configuration to avoid syntax errors:
```bash
sudo nginx -t
```
If the test is successful, restart Nginx:
```bash
sudo systemctl restart nginx
```

---

## **Step 5: Firewall Configuration (If Needed)**
Ensure that your firewall allows HTTP traffic:

### **Ubuntu/Debian (UFW Firewall)**
```bash
sudo ufw allow 80/tcp
sudo ufw reload
```

### **CentOS/RHEL (firewalld)**
```bash
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
```

---

## **Step 6: Verify Load Balancing**

### **Option 1: Use `curl`**
Run multiple requests from a client machine:
```bash
curl -I http://192.168.1.10
```
You should see responses alternating between `192.168.1.100` and `192.168.1.101`.

### **Option 2: Check Nginx Logs**
Monitor Nginx logs to see requests being distributed:
```bash
sudo tail -f /var/log/nginx/access.log
```

### **Option 3: Use a Web Browser**
1. Open a browser and go to `http://192.168.1.10`
2. Refresh multiple times – traffic should alternate between servers.

---

## **Step 7: Optional - Enable Health Checks**
To automatically remove unhealthy servers, modify the `upstream` block:
```nginx
upstream backend_servers {
    least_conn;
    server 192.168.1.100 max_fails=3 fail_timeout=10s;
    server 192.168.1.101 max_fails=3 fail_timeout=10s;
}
```
- If a server fails 3 times within 10 seconds, it will be **temporarily removed** from load balancing.

---

## **Conclusion**
You have now successfully set up Nginx as a reverse proxy with load balancing between `192.168.1.100` and `192.168.1.101`. This setup ensures efficient traffic distribution, improves reliability, and provides failover protection. 

You can further enhance this setup by adding HTTPS with Let's Encrypt, session persistence, or caching mechanisms.

