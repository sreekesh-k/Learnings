
# Step-by-Step Guide to Setting Up a Backend Server on DigitalOcean and Accessing It via a Custom Domain

## Step 1: Create a Droplet on DigitalOcean
1. Log in to your DigitalOcean account.
2. Create a new Droplet:
   - **Image:** Ubuntu 22.04 (LTS) x64.
   - **Plan:** Choose based on your requirements.
   - **Data Center Region:** Select a preferred location.
   - **Authentication:** Add SSH keys (recommended) or choose a password.
   - Finalize and create your Droplet.
3. Note the Droplet's public IP address.

## Step 2: SSH into Your Droplet
1. Open a terminal.
2. Connect to your Droplet:
   ```bash
   ssh root@<your_droplet_ip>
   ```

## Step 3: Update and Upgrade the Server
1. Update the package lists and upgrade installed packages:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

## Step 4: Install Node.js and NPM
1. Install Node.js and npm:
   ```bash
   sudo apt install nodejs npm -y
   ```
2. Verify the installations:
   ```bash
   node -v
   npm -v
   ```

## Step 5: Clone Your Backend Repository from GitHub
1. Install Git if it's not already installed:
   ```bash
   sudo apt install git -y
   ```
2. Clone your repository:
   ```bash
   git clone https://github.com/<your-username>/<your-repo>.git /var/www/<your-backend-folder>
   ```

## Step 6: Install PM2 to Manage Your Application
1. Install PM2 globally:
   ```bash
   sudo npm install pm2@latest -g
   ```
2. Navigate to your project directory:
   ```bash
   cd /var/www/<your-backend-folder>
   ```
3. Install dependencies:
   ```bash
   npm install
   ```
4. Start your application with PM2:
   ```bash
   pm2 start <your-entry-file.js> --name <your-app-name>
   ```
5. Save the PM2 process list and enable auto-start:
   ```bash
   pm2 save
   pm2 startup systemd
   ```

## Step 7: Install Nginx
1. Install Nginx:
   ```bash
   sudo apt install nginx -y
   ```

## Step 8: Configure Nginx as a Reverse Proxy
1. Create a new Nginx configuration file:
   ```bash
   sudo nano /etc/nginx/sites-available/<your-domain>
   ```
2. Add the following configuration:
   ```nginx
   server {
       listen 80;
       server_name <your-domain>;

       location / {
           proxy_pass http://localhost:<your-app-port>;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```
3. Enable the configuration:
   ```bash
   sudo ln -s /etc/nginx/sites-available/<your-domain> /etc/nginx/sites-enabled/
   ```
4. Test and reload Nginx:
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

## Step 9: Obtain an SSL Certificate with Certbot
1. Install Certbot:
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   ```
2. Obtain and configure the SSL certificate:
   ```bash
   sudo certbot --nginx -d <your-domain>
   ```

## Step 10: Configure Domain DNS
1. Log in to your domain registrar.
2. Add an A record pointing to your Droplet's IP:
   - **Host:** `@` (or subdomain like `www` if applicable).
   - **Value:** Your Droplet's IP address.
   - **TTL:** Automatic or 1 hour.
3. Wait for the DNS changes to propagate.

## Step 11: Test Your Setup
1. Visit `http://<your-domain>` to check if your backend is accessible.
2. Use `https://<your-domain>` for a secure connection.

## Step 12: Deploy Updates
1. To apply code changes:
   - SSH into your Droplet.
   - Navigate to your backend directory:
     ```bash
     cd /var/www/<your-backend-folder>
     ```
   - Pull the latest changes:
     ```bash
     git pull origin main
     ```
   - Restart PM2:
     ```bash
     pm2 restart <your-app-name>
     ```
2. No need to restart Nginx unless its configuration changes.

---
