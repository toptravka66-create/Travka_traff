# Cloudflare Configuration for TRAVKA
# This file contains recommended Cloudflare security settings
# Free tier should be sufficient for this project

## Installation Steps:
# 1. Go to cloudflare.com and sign up (free)
# 2. Add your domain (example.com)
# 3. Change nameservers to Cloudflare's
# 4. Apply these settings in Cloudflare dashboard

---

## Page Rules (Caching & Performance)

### Rule 1: Cache Everything
**URL:** https://your-domain.com/*
- Cache Level: Cache Everything
- Browser Cache TTL: 1 hour
- Edge Cache TTL: 1 day
- Auto Minify: JavaScript, CSS, HTML

### Rule 2: API Endpoints (No Cache)
**URL:** https://your-domain.com/api/*
- Cache Level: Bypass
- Browser Cache TTL: 30 minutes

---

## Firewall Rules (DDoS Protection)

### Rule 1: Rate Limit
Block IPs with >100 requests/minute
```
(cf.threat_score > 50) or 
(cf.bot_management.score > 30) or
(http.request.rate > 100)
```
Action: Challenge (CAPTCHA) or Block

### Rule 2: Block Known Bots
```
(cf.bot_management.category == "spam") or
(cf.bot_management.category == "scanner")
```
Action: Block

### Rule 3: Geo-blocking (Optional)
If attacks from specific countries:
```
ip.geoip.country in {"RU" "CN"}
```
Action: Challenge

---

## Security Headers (Automatic via Cloudflare)

Enable in Cloudflare → Security → HTTP Response Headers

- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY
- X-XSS-Protection: 1; mode=block
- Referrer-Policy: strict-origin-when-cross-origin
- Content-Security-Policy: (see index.html meta tag)

---

## WAF (Web Application Firewall)

### Sensitivity: High
- Cloudflare Managed Ruleset: ON
- Owasp ModSecurity Core Rule Set: ON
- Sensitivity Level: High
- Paranoia Level: 2

### Rules to Enable:
- SQL Injection Protection: ON
- XSS Protection: ON
- Local File Inclusion: ON
- Remote File Inclusion: ON

---

## DDoS Protection (Automatic)

Cloudflare automatically:
- ✅ Detects and mitigates DDoS attacks
- ✅ Blocks traffic from known botnets
- ✅ Analyzes traffic patterns
- ✅ Adapts to threats in real-time

### Advanced (Pro+ only):
- Sensitivity: High
- Advanced DDoS Settings: Enabled

---

## Performance Optimization

### Caching:
- Browser Cache TTL: 1 hour
- Edge Cache TTL: 1 day
- Cache Purge: Automatic
- Cache Everything: Yes (for static content)

### Compression:
- Gzip: ON
- Brotli: ON
- Minify: HTML, CSS, JavaScript

### Connection:
- HTTP/2: ON
- HTTP/3 (QUIC): ON
- TLS 1.3: ON
- Always Use HTTPS: ON

---

## Monitoring & Analytics

### View in Cloudflare Dashboard:
- Analytics → Overview
  - Requests: Total/Cached/Blocked
  - Data Transfer: Saved/Total
  - Performance: Average response time
  
- Security → Events
  - Blocked Requests
  - Challenges (CAPTCHAs)
  - Threats Detected

---

## SSL/TLS Configuration

### Recommended:
- SSL/TLS Encryption Mode: Full (Strict)
- Minimum TLS Version: TLS 1.2
- HSTS: Enable (max-age: 31536000)
- Certificate: Auto (free)
- OCSP Stapling: ON

### HSTS Header:
Automatically adds:
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

---

## Email Routing (Optional)

If you want admin notifications:

1. Enable Email Routing
2. Add forwarding rule:
   - Catch-all: @your-domain.com
   - Forward to: @gmail.com or @company-email.com

Benefits:
- Get security alerts
- Receive admin notifications
- Backup admin contact

---

## Free Tier Features Included:

✅ DDoS Protection (unlimited)
✅ SSL/TLS Certificate (auto)
✅ Global CDN (42+ datacenters)
✅ Web Application Firewall (basic rules)
✅ Analytics & Reporting
✅ Email Routing
✅ Page Rules (3 rules)
✅ Firewall Rules (basic)

---

## Upgrade Path (when/if needed):

### Cloudflare Pro ($20/month):
- Advanced Firewall Rules
- Rate Limiting (custom rules)
- WAF Sensitivity Control
- Bot Management (basic)
- PagerDuty Alerts

### Cloudflare Business ($200/month):
- Priority Support
- Advanced Bot Management
- Custom SSL Certificates
- CNAME Setup

---

## Monitoring with Cloudflare Workers (Advanced)

Optional: Create a worker to log attacks locally:

```javascript
export default {
  async fetch(request) {
    // Log suspicious activity
    if (request.headers.get('cf-threat-score') > 50) {
      // Send alert to admin
      console.log('Potential threat detected');
    }
    return fetch(request);
  }
};
```

---

## Backup: Using GitHub Pages only (No Cloudflare)

If Cloudflare setup is too complex:

GitHub Pages provides:
- ✅ Automatic HTTPS
- ✅ Basic DDoS protection
- ✅ Global CDN
- ✅ 99.9% uptime SLA

No additional configuration needed!

---

## Quick Start: 5-Minute Setup

1. Go to cloudflare.com
2. Sign up (free account)
3. Add domain: `your-domain.com`
4. Copy nameservers
5. Update domain registrar
6. Wait 15-30 minutes for propagation
7. Enable "Always Use HTTPS"
8. Done! ✅

---

## Testing & Verification

After setup:

### Check HTTPS:
```bash
curl -I https://your-domain.com
# Should show: HTTP/2 200
```

### Check Headers:
```bash
curl -I https://your-domain.com | grep -i "x-frame"
# Should show: x-frame-options: DENY
```

### Check SSL Grade:
Visit: ssllabs.com/ssltest
Should get: A+ rating

---

Last Updated: 2026-07-03
Status: Ready to Deploy
