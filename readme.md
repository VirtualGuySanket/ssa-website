# 🚀 Static Website Deployment Guide for https://successscienceacademy.com

This guide explains how to deploy a **static website** to your **Hostinger KVM VPS** using Docker, with automatic HTTPS via Let's Encrypt and CI/CD using GitHub Actions.

---

## 🌐 Domain

- **Website**: https://successscienceacademy.com  
- **Registrar**: GoDaddy  
- **Hosted on**: Hostinger KVM VPS

---

## 🧱 Project Structure (GitHub Repo)

```bash
your-static-site-repo/
├── app/                 # Static files (index.html, etc)
├── Dockerfile
├── docker-compose.yml
└── .github/
    └── workflows/
        └── deploy.yml   # GitHub Actions
📄 Dockerfile
dockerfile
Copy code
FROM nginx:alpine
COPY ./app /usr/share/nginx/html
EXPOSE 80
🐳 docker-compose.yml
yaml
Copy code
version: '3'
services:
  static-site:
    build: .
    container_name: success-science-app
    environment:
      VIRTUAL_HOST: successscienceacademy.com
      LETSENCRYPT_HOST: successscienceacademy.com
      LETSENCRYPT_EMAIL: youremail@example.com
    expose:
      - "80"
    restart: always
🔁 Reverse Proxy Setup (on VPS)
Run these steps as the deployer user.

1. SSH into your VPS
bash
Copy code
ssh deployer@<your-vps-ip>
2. Create proxy directory
bash
Copy code
mkdir -p /opt/reverse-proxy
cd /opt/reverse-proxy
3. Create docker-compose.yml
yaml
Copy code
version: "3"
services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - ./certs:/etc/nginx/certs
      - ./vhost.d:/etc/nginx/vhost.d
      - ./html:/usr/share/nginx/html
    restart: always

  letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: nginx-letsencrypt
    environment:
      NGINX_PROXY_CONTAINER: nginx-proxy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./certs:/etc/nginx/certs
      - ./vhost.d:/etc/nginx/vhost.d
      - ./html:/usr/share/nginx/html
    restart: always
4. Start the reverse proxy
bash
Copy code
docker compose up -d
This will:

Start an Nginx reverse proxy

Auto-generate HTTPS via Let’s Encrypt when you deploy your app

🌐 GoDaddy DNS Setup
In GoDaddy DNS manager:

A Record:

Host: @

Points to: <your VPS IP>

TTL: 600

🔐 CI/CD Setup with GitHub Actions
1. SSH Key Setup
On your local machine:

bash
Copy code
ssh-keygen -t ed25519 -C "ci-deploy" -f deploy_key
Upload deploy_key.pub to /home/deployer/.ssh/authorized_keys on VPS

Add deploy_key (private key) to GitHub repo as secret: DEPLOY_KEY

2. GitHub Actions Workflow
.github/workflows/deploy.yml

yaml
Copy code
name: Deploy to VPS

on:
  push:
    branches:
      - release

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up SSH
        uses: webfactory/ssh-agent@v0.7.0
        with:
          ssh-private-key: ${{ secrets.DEPLOY_KEY }}

      - name: Deploy to server
        run: |
          ssh deployer@<your-vps-ip> <<'EOF'
            mkdir -p /opt/successscienceacademy.com
            cd /opt/successscienceacademy.com
          EOF

      - name: Sync project files
        run: |
          rsync -avz --delete ./ deployer@<your-vps-ip>:/opt/successscienceacademy.com/

      - name: Build and start Docker container
        run: |
          ssh deployer@<your-vps-ip> <<'EOF'
            cd /opt/successscienceacademy.com
            docker compose down
            docker compose up -d --build
          EOF
✅ Final Checklist
Task	Status
Static files in /app	✅
Docker setup completed	✅
Reverse proxy running	✅
DNS A record set	✅
GitHub Actions working	✅
HTTPS with Let's Encrypt	✅

🧪 How to Test
Merge into release branch

GitHub Actions deploys site

Visit: https://successscienceacademy.com

✅ You should see your static site with HTTPS enabled!

📌 Next Steps
Add staging domain? (staging.successscienceacademy.com)

Add API backend container?

Set up monitoring or error tracking?
