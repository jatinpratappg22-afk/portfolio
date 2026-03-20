# My Portfolio App 🚀

A Node.js portfolio site with automatic deployment to AWS EC2 via GitHub Actions.

## Project Structure

```
myapp/
├── server.js              # Express server
├── package.json
├── public/
│   └── index.html         # Your portfolio page
└── .github/
    └── workflows/
        └── deploy.yml     # Auto-deploy on git push
```

---

## 🖥️ Local Development

```bash
# Install dependencies
npm install

# Run locally
npm start

# Visit http://localhost:3000
```

---

## ☁️ AWS EC2 Setup (One-Time)

### Step 1 — Launch EC2 Instance
1. Go to **AWS Console → EC2 → Launch Instance**
2. Choose **Ubuntu 22.04 LTS** (free tier eligible)
3. Instance type: **t2.micro** (free tier)
4. Create a new key pair → download the `.pem` file
5. Security Group — allow inbound:
   - **SSH** (port 22) — your IP
   - **HTTP** (port 80) — anywhere
   - **Custom TCP** (port 3000) — anywhere

### Step 2 — Connect to EC2 & Install Dependencies

```bash
# SSH into your server (replace with your details)
ssh -i your-key.pem ubuntu@YOUR_EC2_IP

# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Install PM2 (keeps app running after you log out)
sudo npm install -g pm2

# Install Git
sudo apt install -y git

# Clone your repo
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git ~/myapp
cd ~/myapp
npm install

# Start the app
pm2 start server.js --name myapp
pm2 save            # Auto-restart on reboot
pm2 startup         # Run the command it gives you
```

### Step 3 — Test It
Visit `http://YOUR_EC2_IP:3000` in your browser — you should see your portfolio!

---

## 🔄 GitHub Actions Auto-Deploy Setup

Every time you `git push` to `main`, GitHub will automatically SSH into your EC2 and deploy.

### Add GitHub Secrets

Go to your GitHub repo → **Settings → Secrets and variables → Actions → New secret**

| Secret Name   | Value                                      |
|---------------|--------------------------------------------|
| `EC2_HOST`    | Your EC2 public IP (e.g. `13.229.xx.xx`)  |
| `EC2_USER`    | `ubuntu`                                   |
| `EC2_SSH_KEY` | Contents of your `.pem` file (entire text) |

### How to copy your .pem key contents:
```bash
cat your-key.pem
# Copy everything including -----BEGIN RSA PRIVATE KEY----- lines
```

### That's it! Now test it:
```bash
git add .
git commit -m "My first deployment"
git push origin main
```

Watch the **Actions** tab in GitHub — you'll see it deploy live! ✅

---

## 🌐 Optional: Use Port 80 (No port number in URL)

```bash
# On your EC2 server — redirect port 80 to 3000
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 3000
```

Now visit `http://YOUR_EC2_IP` directly (no `:3000` needed).

---

## GitHub vs GitLab — Which to Use?

**Use GitHub** ✅ — Reasons:
- GitHub Actions is easier to set up for beginners
- Larger community and more tutorials
- Free private repos and generous free CI/CD minutes (2,000/month)
- Better AWS integration documentation

Use GitLab if: your company uses it, or you need more advanced self-hosted CI/CD options later.
