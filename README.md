# DNS setup on Github pages and DigitalOcean droplet

## Project Overview
Connect custom domain to 
1. **Github pages** - for static website hosting
2. **DigitalOcean Droplet** - for hosting custom server or website application.

---

## Task 1: Setting up Custom Domain for Github pages

### Step A: Add your Domain in Github
1. Navigate to Github repository -> **Settings -> Pages**
2. Under **Custom Domain**, enter your domain name (alakikanju.xyz)
3. Save changes: A CNAME file will be automatically created in your repository


---

### Step B: Configure DNS Records
1. Login to your domain registrar (GoDaddy in this case)
2. Navigate to **Networking** section and add the following records under **Domain** tab:

#### Root domain ("alakikanju.xyz")
| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | @ |185.199.108.153 | 3600 |
| A | @ |185.199.109.153 | 3600 |
| A | @ |185.199.110.153 | 3600 |
| A | @ |185.199.111.153 | 3600 |

#### Subdomain ('www.alakikanju.xyz')
| Type | Name | Value | TTL |
|------|------|-------|-----|
| CNAME | www | `<yourusername>.github.io` | 3600 |

> Replace `<yourusername>` with your GitHub username.


---

### Step C: Enable HTTPS
After DNS propagation (usually 10 - 30 minutes):
1. Navigate to **Settings -> Pages** in Github
2. Check **Enforce HTTPS**

Your site will now be accessible securely at: https://alakikanju.xyz


---

Task 2: Setting up Custom Domain for DigitalOcean droplet
1. Navigate to **Networking** section of your DigitalOcean account

2. Click on **Add a Domain** and enter the necessary information required to create a domain.

3. Click on **Create Records** and add the following information:
### Information
| Record Type | Hostname | Will direct to | TTL (seconds) |
|-------------|----------|----------------|
| A           |     @    | 45.55.177.156  | 3600 |
| A           |    www   | 45.55.177.156  | 3600 |

4. SSH into the remote server (Digitalocean) using
   ```ssh -i <key-pair> root@SERVER-IP```

5. Install Nginx
```bash sudo apt update
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

6. Verify Nginx to check if its running
```systemctl status nginx```
You should see the default Nginx welcome page when visiting your server IP in a browser.

7. Upload your static website files and give the necessary permissions
```bash
sudo mkdir -p /var/www/static-site
sudo chown -R $USER:$USER /var/www/static-site
sudo chmod -R 755 /var/www/static-site
```

8. On your local machine, upload your site files using **`rysnc`**
```bash rsync -avz --delete static-site/ root@138.197.82.224:/var/www/static-site/
```

9. Configure Nginx for your Domain
```bash
sudo nano /etc/nginx/sites-available/alakikanju
```

10. Paste the following:
```nginx
server {
    listen 80;
    server_name alakikanju.xyz www.alakikanju.xyz;

    root /var/www/static-site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

11. Enable the configuration and reload Nginx
```bash sudo ln -s /etc/nginx/sites-available/alakikanju /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

12. Enable HTTPS with Certbot
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d alakikanju.xyz -d www.alakikanju.xyz
```

Test your domain `http://alakikanju.xyz`

Project URL: https://roadmap.sh/projects/basic-dns
