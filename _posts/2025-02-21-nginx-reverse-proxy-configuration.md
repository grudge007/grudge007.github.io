---
layout: post
title: Nginx Reverse Proxy Configuration
date: 21-02-2025
categories: [technology basics, reverse-proxy]
tags: [software, configuration, beginners guide]
---

# **Setting Up Nginx as a Reverse Proxy**

## **Overview**
This guide explains how to configure Nginx as a reverse proxy to forward requests from `192.168.1.10` to a backend server at `192.168.1.100`.

A reverse proxy allows clients to access a backend server without exposing it directly, providing additional security, load balancing, and caching benefits.

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

## **Step 2: Configure Nginx as a Reverse Proxy**

Nginx configurations are stored in different locations depending on the OS:
- **Ubuntu/Debian**: `/etc/nginx/sites-available/`
- **CentOS/RHEL**: `/etc/nginx/conf.d/`

### **Create a Reverse Proxy Configuration**
Edit the configuration file:

#### **Ubuntu/Debian**
```bash
sudo nano /etc/nginx/sites-available/reverse-proxy
```

#### **CentOS/RHEL**
```bash
sudo nano /etc/nginx/conf.d/reverse-proxy.conf
```

Add the following content:
```nginx
server {
    listen 80;
    server_name _;  # Accepts all domain/IP requests

    location / {
        proxy_pass http://192.168.1.100;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### **Explanation:**
- `listen 80;` → Listens for HTTP traffic on port 80.
- `server_name _;` → Catches all incoming requests regardless of domain or IP.
- `proxy_pass http://192.168.1.100;` → Forwards requests to the backend server.
- `proxy_set_header` → Ensures the backend server sees the correct request headers.

---

## **Step 3: Enable the Configuration**

### **Ubuntu/Debian:**
Enable the new configuration by creating a symbolic link:
```bash
sudo ln -s /etc/nginx/sites-available/reverse-proxy /etc/nginx/sites-enabled/
```
Disable the default Nginx welcome page:
```bash
sudo rm /etc/nginx/sites-enabled/default
```

### **CentOS/RHEL:**
No symlink is needed; simply ensure that the configuration file exists in `/etc/nginx/conf.d/`.

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

## **Step 6: Verify Reverse Proxy Functionality**
To confirm that requests are correctly proxied, use a browser or `curl`:
```bash
curl -I http://192.168.1.10
```
If everything is set up correctly, you should see a response from `192.168.1.100` instead of the default Nginx page.

---

## **Troubleshooting**

### **Seeing Nginx Default Page Instead of the Backend?**
1. Check active server blocks:
   ```bash
   sudo nginx -T | grep server_name
   ```
   Ensure only the reverse proxy configuration is active.

2. Ensure the configuration is loaded:
   ```bash
   sudo systemctl restart nginx
   ```

3. Verify that `192.168.1.100` is accessible from the proxy server:
   ```bash
   curl -I http://192.168.1.100
   ```

### **Want to Redirect Instead of Proxy?**
If you want to redirect users to `192.168.1.100` instead of just forwarding requests, use:
```nginx
server {
    listen 80;
    server_name _;
    return 301 http://192.168.1.100$request_uri;
}
```
This will make users' browsers change the URL to `192.168.1.100`.

---

## **Conclusion**
You have now successfully set up an Nginx reverse proxy on `192.168.1.10` to forward traffic to `192.168.1.100`. This setup improves security and allows centralized control over incoming requests. You can further enhance the configuration by adding HTTPS with Let's Encrypt, rate limiting, or caching mechanisms.

