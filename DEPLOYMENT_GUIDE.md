# Graybeard Odoo Modules - Deployment Guide

**Version:** 1.0  
**Date:** 2025-12-28  
**Target:** Odoo 19+

---

## Overview

This document provides comprehensive deployment instructions for all Graybeard Odoo modules and applications. These components were battle-tested in production managing delivery and manufacturing operations with significant scale.

**Total Components:** 5 (3 Odoo modules + 2 Flask applications)

---

## Odoo Modules (3)

### 1. Graybeard Delivery Calculator

**Repository:** https://github.com/Alex-Pennington/graybeard_odoo_delivery_calculator  
**Module Name:** `graybeard_odoo_delivery_calculator`  
**Version:** 19.0.1.0.0  
**License:** LGPL-3  
**Status:** ✅ Public

**Description:**  
GPS-based delivery cost calculator that automatically calculates delivery fees based on customer location. Integrates with Google Maps Distance Matrix API for accurate road distance calculations.

**Key Features:**
- Automatic delivery cost calculation from GPS coordinates
- Google Maps API integration (optional - falls back to Haversine formula)
- Configurable origin point, rate per mile, max distance
- Road distance multiplier for realistic routing
- Partner address geocoding
- Delivery carrier integration

**Dependencies:**
- `base`, `sale`, `delivery`, `contacts`

**Configuration Required:**
- `.env` file with delivery origin coordinates (latitude/longitude)
- Rate per mile settings
- Maximum delivery distance
- Optional: Google Maps API key for enhanced accuracy

**Installation:**
```bash
# Clone repository
git clone https://github.com/Alex-Pennington/graybeard_odoo_delivery_calculator

# Copy to Odoo addons
cp -r graybeard_odoo_delivery_calculator /path/to/odoo/addons/

# Configure environment
cd /path/to/odoo/addons/graybeard_odoo_delivery_calculator
cp .env.example .env
nano .env

# Restart Odoo
sudo systemctl restart odoo

# Install via Apps menu
```

**Notes for Dev Team:**
- All configuration via environment variables (no Odoo Settings UI)
- Uses Haversine formula for straight-line distance when Google Maps disabled
- Distance calculations happen automatically when delivery method selected
- Originally designed for regional delivery business

**Deployment Priority:** Medium (if delivery features needed)

---

### 2. Graybeard Driver Portal

**Repository:** https://github.com/Alex-Pennington/graybeard_odoo_driver_portal  
**Module Name:** `graybeard_odoo_driver_portal`  
**Version:** 19.0.1.0.0  
**License:** LGPL-3  
**Status:** ✅ Public

**Description:**  
Mobile-friendly driver portal for viewing assigned deliveries and recording on-site payments. Provides simplified interface for delivery drivers without full Odoo access.

**Key Features:**
- Mobile-optimized delivery list at `/driver/deliveries`
- View assigned deliveries (filtered by driver)
- Mark deliveries complete
- Record cash/check payments on-site
- Automatic payment reconciliation with invoices
- Access control via security groups

**Dependencies:**
- `base`, `sale`, `stock`, `account`

**Configuration Required:**
- None - works out of the box with Odoo authentication
- Add drivers to `group_delivery_driver` security group
- Ensure sale orders have driver assignment field

**Installation:**
```bash
# Clone repository
git clone https://github.com/Alex-Pennington/graybeard_odoo_driver_portal

# Copy to Odoo addons
cp -r graybeard_odoo_driver_portal /path/to/odoo/addons/

# Restart Odoo
sudo systemctl restart odoo

# Install via Apps menu

# Configure drivers
# In Odoo: Settings → Users → Add users to "Delivery Driver" group
```

**Notes for Dev Team:**
- No external dependencies or API keys needed
- Uses Odoo's standard authentication
- Extremely lightweight - just views and controllers
- Payment integration uses standard Odoo accounting journals
- Requires `x_driver_id` field on sale orders (may need custom field)

**Deployment Priority:** High (if using delivery drivers)

---

### 3. Graybeard Dashboard

**Repository:** https://github.com/Alex-Pennington/graybeard-odoo-dashboard  
**Module Name:** `graybeard_dashboard`  
**Version:** 19.0.1.0.0  
**License:** LGPL-3  
**Status:** ✅ Public

**Description:**  
Real-time operations dashboard providing at-a-glance daily metrics for delivery and manufacturing businesses. Built with Owl framework for fast, reactive updates.

**Key Features:**
- **Financial:** Bank balances, AR/AP, revenue tracking (today/MTD)
- **Production:** Weekly output, commission tracking, inventory levels
- **Operations:** Today's/tomorrow's deliveries, manufacturing orders
- **Sales:** New orders, pending quotes, low stock alerts, top products
- **Weather:** 3-day forecast with active alerts (US locations only, free National Weather Service API)
- **Leads:** CRM lead tracking
- Auto-refresh every 5 minutes
- Owl framework (fast, reactive UI)

**Dependencies:**
- `base`, `account`, `stock`, `mrp`, `sale`, `purchase`, `crm`

**Configuration Required:**
- Optional `.env` file for weather coordinates (US locations only)
- Weather uses National Weather Service API (free, no API key needed)

**Installation:**
```bash
# Clone repository
git clone https://github.com/Alex-Pennington/graybeard-odoo-dashboard

# Copy to Odoo addons
cp -r graybeard_dashboard /path/to/odoo/addons/

# Optional: Configure weather
cd /path/to/odoo/addons/graybeard_dashboard
cp .env.example .env
nano .env  # Set latitude/longitude

# Restart Odoo
sudo systemctl restart odoo

# Install via Apps menu
```

**Notes for Dev Team:**
- Weather is optional - dashboard works without it
- Commission calculations may need customization for your business
- Production targets hardcoded in Python model (easy to modify)
- Extremely useful for warehouse/delivery operations overview
- No database changes - purely read-only dashboard
- Originally built for firewood delivery but metrics are generic

**Deployment Priority:** Low (nice-to-have for management)

---

## Flask Applications (2)

### 4. Graybeard Odoo Scheduler

**Repository:** https://github.com/Alex-Pennington/graybeard-odoo-scheduler  
**Technology:** Flask + Waitress WSGI + SQLite  
**Version:** 1.0.0  
**License:** LGPL-3  
**Status:** ✅ Public

**Description:**  
Web-based delivery and manufacturing scheduling system that integrates with Odoo via XML-RPC. Provides calendar views for office staff, drivers, and manufacturing teams, plus a public booking widget for customers.

**Key Features:**
- **Office View** (`/scheduler/`) - Week-based scheduling, drag-and-drop delivery assignment
- **Driver View** (`/driver/`) - Daily delivery view for mobile devices
- **Manufacturing View** (`/manufacturing/`) - Production calendar coordination
- **Public Widget** (`/scheduler/public/booking`) - Customer self-scheduling
- Google Calendar sync support
- SMS integration capabilities
- Pulls trucks from Odoo Fleet module
- Pulls drivers from HR Employees

**Dependencies:**
- Python 3.8+
- Flask, Waitress, SQLAlchemy
- Odoo 19+ with XML-RPC enabled
- PostgreSQL or SQLite for scheduler state

**Configuration Required:**
- `.env` file with Odoo connection details (URL, database, username, password)
- Optional: Google Calendar API credentials
- Optional: SMS gateway credentials
- Nginx reverse proxy recommended for production

**Installation:**
```bash
# Clone repository
git clone https://github.com/Alex-Pennington/graybeard-odoo-scheduler
cd graybeard-odoo-scheduler

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # Windows: venv\\Scripts\\activate

# Install dependencies
pip install -r requirements.txt

# Configure
cp .env.example .env
nano .env  # Set Odoo URL, DB, credentials

# Run (development)
python app.py

# Run (production with Waitress)
waitress-serve --port=5001 app:application
```

**Architecture:**
- Standalone Flask app (not an Odoo module)
- Integrates via Odoo XML-RPC API
- Runs on separate port (default 5001)
- SQLite database for scheduler state
- Nginx proxies requests to Flask

**Nginx Configuration Example:**
```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;
    
    location /scheduler/ {
        proxy_pass http://127.0.0.1:5001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Notes for Dev Team:**
- This is a SEPARATE application, not an Odoo module
- Must run alongside Odoo, not inside it
- Very sophisticated delivery scheduling logic built in
- Drag-and-drop interface for easy assignment
- Public booking widget can be embedded on websites
- Real-time sync with Odoo via XML-RPC

**Deployment Priority:** Medium (powerful scheduling features if needed)

---

### 5. Graybeard Odoo Quo Integration

**Repository:** https://github.com/Alex-Pennington/graybeard-odoo-quo-integration  
**Technology:** Flask webhook service + SQLite  
**Version:** 1.0.0  
**License:** LGPL-3  
**Status:** ✅ Public

**Description:**  
Webhook integration service that automatically creates Odoo CRM leads when customers call or text your business phone number via Quo (formerly OpenPhone). Includes comprehensive consent management, SMS tracking, and monitoring dashboard.

**Key Features:**
- **Call Tracking:** Auto-creates leads from incoming/missed calls with duration, time, caller ID
- **SMS Integration:** Auto-creates leads from SMS inquiries, includes full message text
- **Lead Management:** Auto-assigns to team members, detects existing customers (no duplicates)
- **Consent & Compliance:** Double opt-in verification, consent ledger, TCPA compliance
- **Director Dashboard:** Web-based monitoring, call/SMS history, error logs, configuration viewer
- **Google Maps:** Address autocomplete, distance calculation, delivery cost estimation
- Multi-database architecture (consent, SMS, calls, errors)

**Dependencies:**
- Python 3.8+
- Flask, SQLAlchemy
- Odoo 19+ with XML-RPC enabled
- Quo (OpenPhone) account with webhook access
- Optional: Google Maps API key

**Configuration Required:**
- `.env` file with:
  - Odoo credentials (URL, database, username, password)
  - Quo webhook signing secret (from Quo dashboard)
  - Quo API key (for outbound SMS)
  - Business phone numbers
  - Default lead owner ID
  - Optional: Google Maps API keys

**Installation:**
```bash
# Clone repository
git clone https://github.com/Alex-Pennington/graybeard-odoo-quo-integration
cd graybeard-odoo-quo-integration

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure
cp .env.example .env
nano .env  # Set all required values

# Run (development)
python quo_odoo_lead_integration.py

# Run (production)
# See systemd service file in repository
```

**Quo Webhook Configuration:**
1. Log into Quo dashboard at https://app.openphone.com/
2. Go to Settings → Webhooks
3. Create new webhook pointing to: `https://your-domain.com/webhook/quo`
4. Subscribe to events: `call.completed`, `message.received`
5. Copy signing secret to `.env` file

**Architecture:**
- Standalone Flask webhook service
- Listens on `/webhook/quo` endpoint
- Multiple SQLite databases for tracking
- Director dashboard at `/director`
- Integrates with Odoo CRM via XML-RPC

**Nginx Configuration Example:**
```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;
    
    location /webhook/quo {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /director {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
    }
}
```

**Notes for Dev Team:**
- This is a SEPARATE application (~20,000 lines of Python)
- Webhook must be publicly accessible (HTTPS required)
- Quo dashboard needs webhook URL configured
- Very sophisticated consent/compliance system built in
- SMS verification workflows included
- Can be deployed on same server as Odoo (different port)
- Extremely valuable for any Odoo + Quo/OpenPhone users

**Deployment Priority:** Low (only needed if using Quo/OpenPhone)

---

## Deployment Strategy

### Recommended Phases

#### Phase 1: Core Odoo Modules
**Deploy First** (if using delivery features):
1. `graybeard_odoo_driver_portal` - Essential for driver operations
2. `graybeard_odoo_delivery_calculator` - GPS-based pricing

**Deploy Later**:
3. `graybeard_dashboard` - Nice-to-have for management

#### Phase 2: Flask Applications
**Deploy If Needed**:
4. `graybeard-odoo-scheduler` - Advanced scheduling features
5. `graybeard-odoo-quo-integration` - Quo phone system users only

### Installation Order
1. Install Odoo 19 and verify it's working
2. Install required OCA modules (if any dependencies)
3. Upload Graybeard Odoo modules to addons path
4. Restart Odoo and update Apps List
5. Install Graybeard modules via Apps menu
6. Configure `.env` files for each module
7. Deploy Flask apps separately (if needed)
8. Configure Nginx reverse proxy for Flask apps
9. Test all integrations

---

## Environment Variables

### Best Practices
- **Never commit `.env` files to version control**
- Set restrictive file permissions: `chmod 600 .env` (Linux/Mac)
- Use strong passwords for Odoo users
- Rotate API keys periodically
- Use HTTPS for all webhook endpoints
- Store production secrets in secure secrets manager

### Example `.env` Structure
```bash
# Odoo Connection
ODOO_URL=https://your-odoo-instance.com
ODOO_DB=your_database
ODOO_USERNAME=admin
ODOO_PASSWORD=strong_password_here

# Google Maps (optional)
GOOGLE_MAPS_API_KEY=your_api_key_here

# Quo Integration (if using)
QUO_SIGNING_SECRET=your_signing_secret
QUO_API_KEY=your_api_key

# Business Configuration
BUSINESS_PHONES=+15555555555
DEFAULT_LEAD_OWNER_ID=3
```

---

## Security Requirements

### Critical Security Practices
- ✅ All credentials in `.env` files
- ✅ File permissions restricted (600 on Linux)
- ✅ HTTPS for production deployments
- ✅ Webhook signature verification enabled
- ✅ Regular API key rotation
- ✅ Odoo user permission reviews
- ✅ Audit logging enabled

### API Keys Required

**Google Maps API:**
- Distance Matrix API (delivery calculator)
- Geocoding API (address lookup)
- Places API (autocomplete)

**Quo/OpenPhone API:**
- Webhook signing secret (verification)
- API key (outbound SMS)

---

## Testing Checklist

### Odoo Modules
- [ ] Module installs without errors
- [ ] No database migration issues
- [ ] Security groups configured correctly
- [ ] Views render properly
- [ ] Forms submit successfully
- [ ] Scheduled actions working (if any)

### Flask Applications
- [ ] Application starts without errors
- [ ] Health check endpoint responds
- [ ] Odoo XML-RPC connection successful
- [ ] Webhook signature verification works
- [ ] Database migrations complete
- [ ] Nginx reverse proxy configured
- [ ] HTTPS certificates valid

---

## Known Limitations

### Delivery Calculator
- Google Maps required for accurate road distances
- Falls back to straight-line (Haversine) without API key
- Distance calculations assume road multiplier in non-Google mode

### Driver Portal
- Requires custom `x_driver_id` field on sale orders
- Mobile-optimized but not a native app

### Dashboard
- Weather only works for US locations (National Weather Service API)
- Production targets may need customization
- Commission calculations may need adjustment

### Scheduler (Flask)
- Separate application requiring additional server setup
- Not as tightly integrated as native Odoo module
- Requires XML-RPC enabled on Odoo

### Quo Integration (Flask)
- Only works with Quo/OpenPhone (not generic SIP/VoIP)
- Requires publicly accessible webhook endpoint
- SMS features require Quo API access

---

## Troubleshooting

### Module Won't Install
- Check Odoo logs: `/var/log/odoo/odoo.log`
- Verify all dependencies installed
- Ensure module in addons path
- Try: Apps → Update Apps List

### Flask App Won't Start
- Check Python version (3.8+ required)
- Verify all dependencies: `pip install -r requirements.txt`
- Review `.env` configuration
- Check port not already in use
- Review application logs

### Odoo Connection Failed
- Verify Odoo URL is accessible
- Check database name is correct
- Confirm username/password are valid
- Ensure XML-RPC enabled on Odoo
- Test with: `python -c "import xmlrpc.client; print(xmlrpc.client.ServerProxy('http://localhost:8069').version())"`

### Webhook Not Working
- Verify webhook URL is publicly accessible
- Check HTTPS certificate is valid
- Review Quo webhook signing secret
- Check Flask app logs for errors
- Test with: `curl https://your-domain.com/health`

---

## Support & Documentation

### Each Repository Includes
- ✅ Comprehensive README
- ✅ CHANGELOG documenting all changes
- ✅ `.env.example` configuration template
- ✅ License file (LGPL-3)
- ✅ Troubleshooting guides

### Additional Resources
- Odoo Documentation: https://www.odoo.com/documentation/
- Flask Documentation: https://flask.palletsprojects.com/
- Google Maps API: https://developers.google.com/maps
- Quo API: https://docs.openphone.com/

---

## Migration History

These modules were migrated from a production Odoo 17 installation to Odoo 19 with comprehensive white-labeling and security hardening.

**Migration Process:**
1. **White-labeling:** All business-specific branding removed
2. **Security Hardening:** All credentials externalized to `.env` files
3. **Genericization:** Business-specific logic made configurable
4. **Odoo 19 Updates:** Manifest versions, API compatibility verified
5. **Documentation:** Comprehensive guides created

**Original Use Case:**
These modules powered a delivery business processing thousands of deliveries with 200% year-over-year growth. They have been fully genericized for reuse in any delivery, logistics, or service business.

---

## License

All modules and applications are licensed under **LGPL-3**.

See LICENSE file in each repository for details.

---

## Quick Reference

| Component | Type | Priority | Repository |
|-----------|------|----------|------------|
| Delivery Calculator | Odoo Module | Medium | [GitHub](https://github.com/Alex-Pennington/graybeard_odoo_delivery_calculator) |
| Driver Portal | Odoo Module | High | [GitHub](https://github.com/Alex-Pennington/graybeard_odoo_driver_portal) |
| Dashboard | Odoo Module | Low | [GitHub](https://github.com/Alex-Pennington/graybeard-odoo-dashboard) |
| Scheduler | Flask App | Medium | [GitHub](https://github.com/Alex-Pennington/graybeard-odoo-scheduler) |
| Quo Integration | Flask App | Low | [GitHub](https://github.com/Alex-Pennington/graybeard-odoo-quo-integration) |

---

**Document Version:** 1.0  
**Last Updated:** 2025-12-28  
**Status:** Production Ready ✅
