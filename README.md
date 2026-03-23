# End To End DevSecOps Project Implementation

## 🛤️ Jerney — Blog Platform

A Gen-Z vibe blog platform built with a 3-tier architecture — React frontend, Node.js backend, and PostgreSQL database.

![Tech Stack](https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react)
![Tech Stack](https://img.shields.io/badge/Node.js-20-339933?style=flat-square&logo=node.js)
![Tech Stack](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql)

---

> [!IMPORTANT]
> **Want the full DevSecOps project implementation?**
> Switch to the [`devops`](../../tree/devops) branch for Docker, Kubernetes (EKS Auto Mode), Terraform, CI/CD with GitHub Actions, container security scanning, and more.
>
> ```bash
> git checkout devops
> ```

---

## ✨ Features

- 📝 Create blog posts with emoji vibes
- ✏️ Edit your existing posts
- 🗑️ Delete posts you're not feeling anymore
- 💬 Comment on posts
- 🎨 Gen-Z dark UI with glassmorphism and gradients

## 🏗️ Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Frontend   │────▶│   Backend    │────▶│  PostgreSQL   │
│   (React +   │◀────│  (Node.js +  │◀────│              │
│    Nginx)    │     │   Express)   │     │              │
│   Port 80    │     │  Port 5000   │     │  Port 5432   │
└──────────────┘     └──────────────┘     └──────────────┘
```

## 📁 Project Structure

```
Jerney/
├── frontend/                # React (Vite) frontend
│   ├── src/                 # React components & pages
│   ├── nginx.conf           # Nginx config for serving the app
│   └── package.json
├── backend/                 # Node.js Express API
│   ├── src/                 # Routes, DB connection
│   └── package.json
├── deploy/                  # EC2 deployment scripts
│   ├── setup.sh             # One-click EC2 setup script
│   └── jerney-nginx.conf    # Nginx reverse proxy config
└── README.md
```

---

## 🚀 Deploy on AWS EC2

### Prerequisites

- An AWS EC2 instance running **Ubuntu 22.04+**
- Security Group allowing inbound traffic on ports **22** (SSH), **80** (HTTP) and **5000** (backend)
- A `.pem` key file for SSH access
- SSH access to the instance


---

### Step 1: Fix SSH Key Permissions (Windows/GitBash Users) to avoid issues only if required.

> ⚠️ **Windows users must do this before SSH.** Skipping causes `Permission denied (publickey)`.

Open **PowerShell** (not GitBash) and run:

```powershell
icacls "C:\path\to\your-key.pem" /reset
icacls "C:\path\to\your-key.pem" /inheritance:r /grant:r "%USERNAME%:(R)"
```

Then copy the key to your Linux home directory in GitBash:

```bash
cp /mnt/c/path/to/your-key.pem ~/your-key.pem
chmod 400 ~/your-key.pem
```

> ✅ Always use `~/your-key.pem` (Linux filesystem) instead of `/mnt/d/your-key.pem` (Windows filesystem). `chmod` works correctly only on the Linux filesystem.

---

### Step 2: SSH into the EC2 Instance

```bash
ssh -i "~/your-key.pem" ubuntu@<EC2_PUBLIC_IP>
```

---

### Step 3: Handle the Pending Kernel Upgrade Prompt

After SSH login, you may see a **"Pending kernel upgrade"** dialog. This is normal on fresh EC2 instances.

- Press **Enter** to dismiss it, then reboot:

```bash
sudo reboot
```

- Wait 30–40 seconds and SSH back in:

```bash
ssh -i "~/your-key.pem" ubuntu@<EC2_PUBLIC_IP>
```

- To prevent this dialog from blocking future `apt` installs, run once:

```bash
sudo sed -i "s/#\$nrconf{restart} = 'i';/\$nrconf{restart} = 'a';/" /etc/needrestart/needrestart.conf
```

---

### Step 4: Clone the Repository

```bash
git clone https://github.com/nsaishiva/E2E-DevSecOps.git
cd E2E-DevSecOps
```

### Step 5: Create the Backend `.env` File

> ⚠️ This step is **required** and not handled by `setup.sh`. Without it, the backend crashes with a PostgreSQL connection error.

```bash
cat > /var/www/e2e-devsecops/backend/.env << EOF
PORT=5000
DB_HOST=localhost
DB_PORT=5432
DB_NAME=jerney_db
DB_USER=jerney_user
DB_PASSWORD=jerney_pass_2026
EOF
```

---

### Step 6 : Run the Setup Script

```bash
cd ~/E2E-DevSecOps/deploy
export NEEDRESTART_MODE=a
export DEBIAN_FRONTEND=noninteractive
./setup.sh
```

> ✅ The two `export` lines **must always be set** before running the script. They prevent the kernel upgrade dialog from blocking `apt` mid-install — the root cause of most setup failures.

This script will:

1. Update system packages
2. Install **Node.js 20.x**, **PostgreSQL**, **Nginx**, and **PM2**
3. Create the database and user
4. Install backend dependencies
5. Build the React frontend
6. Configure Nginx as a reverse proxy
7. Start the backend with PM2 (auto-restarts on crash/reboot)

---

Then restart the backend if required:

```bash
pm2 restart e2e-backend
```

---

### Step 7: Verify Everything is Running

```bash
# Check backend process
pm2 status

# Check backend logs (should show: 🚀 backend running on port 5000)
pm2 logs e2e-backend --lines 10

# Check Nginx
sudo systemctl status nginx

# Check PostgreSQL
sudo systemctl status postgresql
```

Expected output:

- PM2 status → `e2e-backend` is **online**
- Nginx → **active (running)**
- PostgreSQL → **active**

---

### Step 8: Access the App

Get your EC2 public IP:

```bash
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

Open in your browser:

```
http://<EC2_PUBLIC_IP>
```

---

## 🔧 Useful Commands

```bash
pm2 status                                    # Check backend status
pm2 logs e2e-backend --lines 30               # View backend logs
pm2 restart e2e-backend                       # Restart backend
pm2 save                                      # Save PM2 process list (survives reboots)
pm2 startup                                   # Enable PM2 auto-start on reboot
sudo systemctl restart nginx                  # Restart Nginx
sudo -u postgres psql -d jerney_db            # Connect to database
```

## 🧑‍💻 Local Development (Without Docker)

### Prerequisites

- Node.js 20+
- PostgreSQL 16+

### Backend

```bash
cd backend
npm install

export DB_HOST=localhost
export DB_PORT=5432
export DB_USER=jerney_user
export DB_PASSWORD=jerney_pass_2026
export DB_NAME=jerney_db
export PORT=5000

npm start
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

The dev server starts on `http://localhost:3000` and proxies `/api` requests to the backend at `http://localhost:5000`.


## Iac implementation with Terraform

```sh
aws configure

terraform --version

terraform init

terraform plan

terraform apply  # This will take up to 15 mins.

aws eks update-kubeconfig --name jerney-eks --region us-east-1
kubectl get pods -A 
kubectl get nodes  # No nodes why because we created EKS Auto mode, Karpenter will set nodes based on loads.

kubectl create deployment nginx --image nginx:latest

```

## Containerizing the Application

```sh
cd frontend
docker build -t jerney-frontend:latest .

cd ../backend
docker build -t jeeney-backend:latest .

# Docker Compose file 

cat ../E2E-DevSecOps/docker-compose.yml

docker compose up -d

```

```sh

docker images
docker login


docker tag 21dae9b74b38 nsaishiva12/jerney-frontend:latest

docker push nsaishiva12/jerney-frontend:latest


docker tag 79ee26c26a18 nsaishiva12/jerney-backend:latest

docker push nsaishiva12/jerney-backend:latest

```

- Now we can access the Jerney application by localhost:80

## Project implementation with Kubernetes Manifests

Frontend, Backend and Database

Deployment + Persistent Volume Claim + Storage Class

Frontend -- Deployment + Service
Backend --  Deployment + Service + Database Secret + Network Policies --> Will accept the only if its coming from frontend.
Database -- Deployment + Service + Persistent Volume Claim + Storage Class --> Will Allow request from backend.

```sh

kubectl get nodes

kubectl apply -f e2e-devsecops.yaml

kubectl get svc -n jerney

kubectl port-forward svc/jerney-frontend 8100:80 -n jerney --address 0.0.0.0


```

- Now we can access app localhost:8100

## Project implementation with GitHub Actions for CI

- Updated  .github/workflows/ci-cd.yml file with required code
- Push code to devops branch to run the CI Pipeline.

## Deployment with Argo CD.

 Install Argo CD

```sh
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl get all -n argocd

kubectl get secrets -n argocd

kubectl edit secret argocd-initial-admin-secret -n argocd

# dTQtejQ0TkdYaTBMTlg0Tw==

echo dTQtejQ0TkdYaTBMTlg0Tw== | base64 --decode

#u4-z44NGXi0LNX4O

kubectl get svc -n argocd

kubectl port-forward svc/argocd-server 7111:80 -n argocd --address 0.0.0.0

```

- ArgoCD can be accessible - localhost:7111
- Use browser and configure the application with github repo and k8s manifest path for automatic deployment in to ArgoCd.

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check |
| GET | `/api/posts` | Get all posts |
| GET | `/api/posts/:id` | Get single post with comments |
| POST | `/api/posts` | Create a new post |
| PUT | `/api/posts/:id` | Update a post |
| DELETE | `/api/posts/:id` | Delete a post |
| GET | `/api/comments/post/:postId` | Get comments for a post |
| POST | `/api/comments` | Create a comment |
| DELETE | `/api/comments/:id` | Delete a comment |


---

## 🌿 Branch Strategy

| Branch | Purpose |
|--------|---------|
| `main` | Source code + EC2 bare-metal deployment |
| `devops` | Full DevSecOps — Docker, Kubernetes (EKS), Terraform, CI/CD pipeline, security scanning |

---


