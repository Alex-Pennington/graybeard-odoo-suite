# Security Policy

## Supported Versions

| Version | Supported |
| ------- | ------------------ |
| 19.x   | :white_check_mark: |
| < 19.0   | :x:                |

We currently support Greybeard modules for Odoo 19+. Older versions may work but are not officially supported.

---

## Reporting a Vulnerability

**Please DO NOT open public GitHub issues for security vulnerabilities.**

### How to Report

1. **Email:** Send details to the repository owner via GitHub
2. **Include:**
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if known)

3. **Response Time:**
   - Initial response: Within 48 hours
   - Status update: Within 7 days
   - Fix timeline: Depends on severity

### What to Expect

- **Acknowledged:** We'll confirm receipt and begin investigation
- **Assessment:** We'll evaluate severity and impact
- **Fix Development:** We'll develop and test a fix
- **Disclosure:** We'll coordinate disclosure timing with you
- **Credit:** You'll be credited for the discovery (unless you prefer anonymity)

---

## Security Best Practices

### Environment Variables

**CRITICAL:** Never commit `.env` files to version control.

```bash
# Always set restrictive permissions
chmod 600 .env

# Example .env structure
ODOO_URL=https://your-odoo.com
ODOO_PASSWORD=strong_password_here
GOOGLE_MAPS_API_KEY=your_key_here
```

**Each repository includes `.env.example`** - copy this and fill in your values.

### API Keys

- **Rotate regularly** - Change API keys every 90 days
- **Use least privilege** - Grant minimum required permissions
- **Monitor usage** - Watch for unusual API call patterns
- **Separate environments** - Different keys for dev/staging/prod

### Webhook Security

**For Quo Integration:**
```python
# Signature verification is built-in and required
QUO_SIGNING_SECRET=your_secret_here  # From Quo dashboard
```

- HTTPS required for all webhook endpoints
- Signature verification always enabled
- Invalid signatures are rejected automatically

### Odoo Security

**User Permissions:**
- Create dedicated Odoo users for integrations
- Use security groups appropriately
- Review permissions quarterly
- Enable two-factor authentication

**Database Security:**
- Use strong PostgreSQL passwords
- Restrict database access to localhost
- Regular backups with encryption
- Monitor for unusual queries

### File Permissions

**Linux/Mac:**
```bash
# Configuration files
chmod 600 .env
chmod 600 config/*.ini

# Application directories
chmod 755 /path/to/app
chown odoo:odoo /path/to/app

# SQLite databases (Flask apps)
chmod 660 *.db
chown odoo:odoo *.db
```

**Windows:**
- Right-click → Properties → Security
- Remove "Everyone" group
- Add only necessary users
- Set to "Read" for service accounts

### HTTPS/TLS

**Production deployments MUST use HTTPS:**

```nginx
# Nginx configuration
ssl_certificate /path/to/cert.pem;
ssl_certificate_key /path/to/key.pem;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers HIGH:!aNULL:!MD5;
```

- Use Let's Encrypt for free certificates
- Renew certificates before expiry
- Test with SSL Labs: https://www.ssllabs.com/ssltest/

### Network Security

**Firewall Rules:**
```bash
# Allow only necessary ports
sudo ufw allow 443/tcp   # HTTPS
sudo ufw allow 22/tcp    # SSH (restrict to known IPs)
sudo ufw deny 8069/tcp   # Block direct Odoo access
sudo ufw deny 5000/tcp   # Block direct Flask access
sudo ufw enable
```

**Reverse Proxy:**
- Use Nginx/Apache as reverse proxy
- Never expose Odoo/Flask ports directly
- Enable rate limiting
- Log all requests

---

## Common Vulnerabilities

### ⚠️ Hardcoded Credentials

**Bad:**
```python
ODOO_PASSWORD = "admin123"  # NEVER DO THIS
```

**Good:**
```python
import os
ODOO_PASSWORD = os.environ.get('ODOO_PASSWORD')
```

### ⚠️ SQL Injection

**Bad:**
```python
query = f"SELECT * FROM table WHERE id = {user_input}"
```

**Good:**
```python
# Odoo ORM handles this automatically
self.env['model'].search([('id', '=', user_input)])
```

### ⚠️ Insufficient Input Validation

**Always validate:**
- Phone numbers (format, length)
- Email addresses (regex)
- File uploads (type, size)
- URL parameters (whitelist)

### ⚠️ Exposed Debug Information

**Production:**
```python
# Flask
DEBUG_MODE=False

# Odoo
log_level = warn
```

Never run production with debug mode enabled.

---

## Security Checklist

### Before Deployment

- [ ] All `.env` files created and configured
- [ ] File permissions set correctly (600 for configs)
- [ ] HTTPS configured and tested
- [ ] Firewall rules applied
- [ ] Debug mode disabled
- [ ] Default passwords changed
- [ ] API keys generated (not using examples)
- [ ] Webhook signatures verified
- [ ] Backup system tested
- [ ] Monitoring configured

### Regular Maintenance

- [ ] Review user permissions (quarterly)
- [ ] Rotate API keys (every 90 days)
- [ ] Update dependencies (monthly)
- [ ] Review access logs (weekly)
- [ ] Test backups (monthly)
- [ ] Security patches applied (as released)

---

## Security Features by Component

### Odoo Modules

**Delivery Calculator:**
- API keys in environment variables only
- No user input to external APIs
- Uses Odoo's built-in security

**Driver Portal:**
- Odoo authentication required
- Security groups enforce access control
- No external API calls

**Dashboard:**
- Read-only data access
- Optional weather API (no auth required)
- No user data transmission

### Flask Applications

**Scheduler:**
- Odoo authentication via XML-RPC
- Session-based auth for web UI
- No public endpoints (except health check)

**Quo Integration:**
- HMAC-SHA256 webhook signature verification
- API token required for automation endpoints
- Odoo authentication for Director dashboard
- Double opt-in for SMS consent
- TCPA compliance features

---

## Compliance

### Data Protection

- **GDPR:** Flask apps store minimal personal data (phone numbers only)
- **TCPA:** Quo Integration includes consent management
- **PCI:** No credit card data is processed or stored

### Logging

**What We Log:**
- Authentication attempts
- API calls
- Webhook events
- Error conditions

**What We DON'T Log:**
- Passwords
- API keys
- Full credit card numbers
- Personal conversations

---

## Updates and Patches

### Security Updates

- Critical: Within 24 hours
- High: Within 7 days
- Medium: Next release
- Low: Backlog

### Notification

Watch repository releases for security announcements.

---

## Third-Party Dependencies

We rely on well-maintained packages:

**Python:**
- Flask, SQLAlchemy, requests
- All kept up-to-date
- Vulnerability scanning enabled

**External Services:**
- Google Maps API (HTTPS only)
- Quo/OpenPhone (webhook signatures)
- National Weather Service (read-only, public data)

---

## Questions?

For non-sensitive security questions, open a GitHub Discussion.

For security vulnerabilities, contact privately via GitHub.

---

**Last Updated:** 2025-12-28  
**Security Version:** 1.0
