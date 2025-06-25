# 🚀 n8n Self-Hosted on Google Cloud with SSL

This guide provides **step-by-step instructions** to self-host [n8n](https://n8n.io), a free and open-source workflow automation tool, on a **Linux server** using **Docker**, **Nginx**, and **Certbot** for SSL with a **custom domain name**.

---

## ☁️ Step 0: Launching a Free VM on Google Cloud

### ✅ Requirements:
- Google Cloud account with billing enabled (Free Tier works)
- Domain (e.g., from GoDaddy, Namecheap)

### 🧾 Steps:

1. **Go to VM Instances**  
   → [https://console.cloud.google.com/compute/instances](https://console.cloud.google.com/compute/instances)

2. **Click "Create Instance"** and use:
   - **Name:** `n8n-vm`
   - **Region:** `us-west1` (Free Tier eligible)
   - **Machine type:** `e2-micro`
   - **Boot Disk:** Ubuntu 22.04 LTS
   - **Allow HTTP & HTTPS traffic**

3. **Firewall Rule (Allow Ports)**  
   SSH into your VM and run:

   ```bash
   gcloud compute firewall-rules create allow-n8n \
     --allow=tcp:80,tcp:443,tcp:5678 \
     --target-tags=n8n-server \
     --source-ranges=0.0.0.0/0 \
     --description="Allow traffic to n8n"
   ```

4. **Tag your VM:**  
   Add tag `n8n-server` in network settings.

5. **Point Your Domain to External IP:**  
   Add an A record from your domain (like `n8n.yourdomain.com`) to your VM's external IP.

---

## 📦 Step 1: Installing Docker

```bash
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

---

## 🐳 Step 2: Starting n8n in Docker

### Root domain:
```bash
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="your-domain.com" \
-e WEBHOOK_TUNNEL_URL="https://your-domain.com/" \
-e WEBHOOK_URL="https://your-domain.com/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n
```

### Subdomain:
```bash
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="subdomain.your-domain.com" \
-e WEBHOOK_TUNNEL_URL="https://subdomain.your-domain.com/" \
-e WEBHOOK_URL="https://subdomain.your-domain.com/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n
```

✅ This command:
- Runs the n8n Docker container
- Exposes it on port `5678`
- Mounts `~/.n8n` for persistent storage

---

## 🌐 Step 3: Installing Nginx

```bash
sudo apt install nginx
```

---

## 🔁 Step 4: Configuring Nginx

```bash
sudo nano /etc/nginx/sites-available/n8n.conf
```

Paste this:

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_read_timeout 86400;
    }
}
```

Then run:

```bash
sudo ln -s /etc/nginx/sites-available/n8n.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## 🔐 Step 5: SSL with Certbot

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
# or for subdomain
# sudo certbot --nginx -d subdomain.your-domain.com
```

---

## ✅ Final Output

- Visit: `https://your-domain.com`  
- Your n8n instance is now securely accessible over HTTPS 🎉

---

## 📺 Youtube Tutorial

[![Youtube Tutorial]()

---

## 💡 Pro Tips

- ✅ Open ports `80`, `443`, and `5678` in your VM firewall
- ♻️ Restart `docker` and `nginx` after any config changes
- 🔒 Use `ufw` (Uncomplicated Firewall) to add additional security
- 🔁 Renew SSL automatically with `certbot renew`
- 🆙 Regularly update Docker and the n8n image

---
## 🐞 Debugging Steps (If Website Doesn’t Load)

If you encounter issues accessing your n8n instance, systematically check the following:

### 1. Check if Nginx is running:

```bash
sudo systemctl status nginx
# If not active (dead), restart it:
sudo systemctl restart nginx
```
### 2. Verify Nginx Configuration:
```bash
sudo nginx -t
# If there are errors, correct them in /etc/nginx/sites-available/n8n.conf
# Then restart Nginx:
sudo systemctl restart nginx
```
### 3. Check SSL Certificate with Certbot:
```bash
sudo certbot certificates
# Or try to renew (dry-run) to check for issues:
sudo certbot renew --dry-run
```

### 4. Ensure Docker is Running and n8n Container is Healthy:
```bash
sudo docker ps
# Look for a container named 'n8n' with status 'Up'. If not running:
sudo docker restart n8n
# Check container logs for errors:
sudo docker logs n8n
```

### 5. Open Ports on Google Cloud Firewall:

Confirm your firewall rules are correctly configured and applied to your VM. You can do this in the Google Cloud Console under VPC network > Firewall rules, or by running the gcloud command again (though it will fail if the rule already exists, but it confirms the syntax).

```bash
# This command creates the rule. If it exists, it will show an error,
# but confirms the command structure. Check the console for existing rules.
gcloud compute firewall-rules create allow-n8n \
  --allow=tcp:80,tcp:443,tcp:5678 \
  --target-tags=n8n-server \
  --source-ranges=0.0.0.0/0 \
  --description="Allow traffic to n8n"
```
 ### Also, ensure your VM instance has the n8n-server network tag applied


### 👨‍💻 Need help?

Create an issue or comment on the [YouTube tutorial]() 💬
