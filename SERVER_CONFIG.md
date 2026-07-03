# Nginx Security Configuration for TRAVKA
# Use this if hosting on custom server (nginx/Apache)

# Main server block
server {
    listen 80;
    listen [::]:80;
    server_name your-domain.com www.your-domain.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    # SSL Configuration
    ssl_certificate /etc/ssl/certs/your-domain.crt;
    ssl_certificate_key /etc/ssl/private/your-domain.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security Headers
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # CSP Header
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' https://api.telegram.org; frame-src 'none'; object-src 'none'; base-uri 'self'; form-action 'self';" always;

    # Root directory
    root /var/www/travka;
    index index.html;

    # Main location
    location / {
        try_files $uri $uri/ /index.html;
        
        # Enable caching
        expires 1h;
        add_header Cache-Control "public, max-age=3600";
        
        # Gzip compression
        gzip on;
        gzip_types text/plain text/css text/javascript application/json;
        gzip_min_length 1000;
    }

    # Block access to sensitive files
    location ~ /\. {
        deny all;
    }

    location ~ ~$ {
        deny all;
    }

    location ~ \.(log|md|git|github)$ {
        deny all;
    }

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=form_limit:10m rate=3r/h;
    location ~ ^/api/submit {
        limit_req zone=form_limit burst=5;
        return 405; # Method not allowed (API doesn't exist)
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "OK";
    }

    # Redirect old URLs if migrating
    # Example:
    # location /old-page {
    #     return 301 https://$server_name/;
    # }

    # Logging
    access_log /var/log/nginx/travka_access.log combined;
    error_log /var/log/nginx/travka_error.log warn;

    # Performance
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 20M;
}

# Rate limiting configuration
limit_req_status 429;

# Map to identify rate limit offenders
map $status $loggable {
    ~^[23] 1;
    default 0;
}

---

# Systemd Service File (if running as service)
# Save as: /etc/systemd/system/travka.service

[Unit]
Description=TRAVKA Security Landing Page
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/travka
ExecStart=/usr/sbin/nginx -g "daemon off;"
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/usr/sbin/nginx -s quit
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target

# Start service:
# sudo systemctl start travka
# sudo systemctl enable travka
# sudo systemctl status travka

---

# Apache Configuration (alternative to Nginx)
# Save as: /etc/apache2/sites-available/travka.conf

<VirtualHost *:80>
    ServerName your-domain.com
    ServerAlias www.your-domain.com
    
    # Redirect to HTTPS
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>

<VirtualHost *:443>
    ServerName your-domain.com
    ServerAlias www.your-domain.com
    
    # SSL
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/your-domain.crt
    SSLCertificateKeyFile /etc/ssl/private/your-domain.key
    SSLCertificateChainFile /etc/ssl/certs/your-domain.ca
    
    SSLProtocol -all +TLSv1.2 +TLSv1.3
    SSLCipherSuite HIGH:!aNULL:!MD5
    SSLHonorCipherOrder on
    
    # Document root
    DocumentRoot /var/www/travka
    
    # Security headers
    Header set X-Frame-Options "DENY"
    Header set X-Content-Type-Options "nosniff"
    Header set X-XSS-Protection "1; mode=block"
    Header set Referrer-Policy "strict-origin-when-cross-origin"
    Header set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    Header set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; connect-src 'self' https://api.telegram.org;"
    
    # Compression
    <IfModule mod_deflate.c>
        AddOutputFilterByType DEFLATE text/plain text/css text/javascript application/json
        AddOutputFilterByType DEFLATE image/svg+xml
    </IfModule>
    
    # Caching
    <IfModule mod_expires.c>
        ExpiresActive On
        ExpiresDefault "access plus 1 hour"
    </IfModule>
    
    # Rewrite rules
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteBase /
        RewriteRule ^index\.html$ - [L]
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule . /index.html [L]
    </IfModule>
    
    # Block sensitive files
    <FilesMatch "^\.">
        Order allow,deny
        Deny from all
    </FilesMatch>
    
    <FilesMatch "(\.log|\.md|\.git|\.github)$">
        Order allow,deny
        Deny from all
    </FilesMatch>
    
    # Logging
    CustomLog ${APACHE_LOG_DIR}/travka_access.log combined
    ErrorLog ${APACHE_LOG_DIR}/travka_error.log
</VirtualHost>

# Enable site:
# sudo a2ensite travka
# sudo systemctl restart apache2

---

# Docker Configuration (for containerization)
# Save as: Dockerfile

FROM nginx:alpine

# Copy static files
COPY index.html /usr/share/nginx/html/
COPY *.md /usr/share/nginx/html/

# Copy nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port
EXPOSE 80 443

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/health || exit 1

# Run nginx
CMD ["nginx", "-g", "daemon off;"]

# Build:
# docker build -t travka:latest .

# Run:
# docker run -p 80:80 -p 443:443 travka:latest

---

# Docker Compose Configuration
# Save as: docker-compose.yml

version: '3.8'

services:
  web:
    image: nginx:alpine
    container_name: travka-web
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./:/usr/share/nginx/html:ro
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 5s

# Run:
# docker-compose up -d

---

# SSL Certificate Generation (Let's Encrypt)

# Install Certbot:
sudo apt-get update
sudo apt-get install certbot python3-certbot-nginx

# Get certificate:
sudo certbot certonly --nginx -d your-domain.com -d www.your-domain.com

# Auto-renew:
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer

# Manual renew:
sudo certbot renew

---

# Monitoring Script
# Save as: monitor.sh

#!/bin/bash

LOG_FILE="/var/log/travka_monitor.log"
ALERT_EMAIL="admin@your-domain.com"

# Check if service is running
if ! curl -f https://your-domain.com/health > /dev/null 2>&1; then
    echo "[ERROR] Service down at $(date)" >> $LOG_FILE
    # Send alert (requires mail setup)
    # echo "TRAVKA is down!" | mail -s "Alert" $ALERT_EMAIL
fi

# Check disk space
DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 90 ]; then
    echo "[WARNING] Disk usage: $DISK_USAGE% at $(date)" >> $LOG_FILE
fi

# Check SSL expiration
SSL_EXPIRY=$(date -d "$(openssl x509 -enddate -noout -in /etc/ssl/certs/your-domain.crt | cut -d= -f2)" +%s)
CURRENT=$(date +%s)
DAYS_LEFT=$(( ($SSL_EXPIRY - $CURRENT) / 86400 ))

if [ $DAYS_LEFT -lt 30 ]; then
    echo "[WARNING] SSL expiring in $DAYS_LEFT days at $(date)" >> $LOG_FILE
fi

# Cron job (run every hour):
# 0 * * * * /root/monitor.sh

---

# Performance Tuning

## OS Level:
- Increase file descriptors: `ulimit -n 65535`
- Enable TCP optimization: `sysctl -w net.ipv4.tcp_fin_timeout=30`
- Use NVMe SSD for best performance

## Nginx Tuning:
- worker_processes: auto
- worker_connections: 1024+
- keepalive_timeout: 65s
- gzip_level: 6 (balance compression/CPU)

## Monitoring Tools:
- Prometheus: Metrics collection
- Grafana: Dashboards
- NewRelic: APM monitoring
- Datadog: Infrastructure monitoring

---

Last Updated: 2026-07-03
Production Ready: YES
