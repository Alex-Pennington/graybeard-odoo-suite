# Getting Started

**Quick start guide for deploying Graybeard Odoo Suite**

---

## Prerequisites

Before you begin, ensure you have:

- ✅ **Odoo 19+** installed and running (Community or Enterprise)
- ✅ **Python 3.8+** on your server
- ✅ **PostgreSQL** database
- ✅ **Git** for cloning repositories
- ✅ **Nginx** (recommended for Flask apps)
- ✅ **sudo/root access** to the server

**Optional:**
- Google Maps API key (for enhanced features)
- Quo/OpenPhone account (for phone integration)

---

## Step 1: Choose Your Components

Not all components are required. Choose based on your needs:

### Essential for Delivery Operations
- ✅ **Driver Portal** - Mobile interface for drivers
- ✅ **Delivery Calculator** - GPS-based pricing

### Nice to Have
- **Dashboard** - Management metrics
- **Scheduler** - Advanced scheduling (Flask app)

### Specialized
- **Quo Integration** - Only if using Quo/OpenPhone

---

## Step 2: Install Odoo Modules

### 2.1 Clone Repositories

```bash
# Navigate to your Odoo addons directory
cd /path/to/odoo/addons

# Clone the modules you need
git clone https://github.com/Alex-Pennington/graybeard_odoo_delivery_calculator
git clone https://github.com/Alex-Pennington/graybeard_odoo_driver_portal
git clone https://github.com/Alex-Pennington/graybeard-odoo-dashboard

# Verify
ls -la graybeard*
```

### 2.2 Set Permissions

```bash
# Set appropriate ownership (adjust user as needed)
sudo chown -R odoo:odoo graybeard*

# Ensure directories are readable
sudo chmod -R 755 graybeard*
```

### 2.3 Restart Odoo

```bash
# Using systemd
sudo systemctl restart odoo

# Or using supervisord
sudo supervisorctl restart odoo

# Or if running directly
# Stop Odoo, then start again with:
# /path/to/odoo-bin -c /path/to/odoo.conf
```

### 2.4 Install Modules

1. Open Odoo in your browser
2. Go to **Apps**
3. Click **Update Apps List**
4. Remove the "Apps" filter
5. Search for "Graybeard"
6. Install the modules you need

---

## Step 3: Configure Delivery Calculator

**Only if you installed Delivery Calculator**

### 3.1 Create Configuration

```bash
cd /path/to/odoo/addons/graybeard_odoo_delivery_calculator
cp .env.example .env
nano .env
```

### 3.2 Set Required Values

```bash
# Your business location (latitude/longitude)
# Find yours at: https://www.latlong.net/
DELIVERY_ORIGIN_LATITUDE=38.1234
DELIVERY_ORIGIN_LONGITUDE=-82.5678

# Pricing
DELIVERY_RATE_PER_MILE=2.50
DELIVERY_MAX_DISTANCE=75.0
DELIVERY_ROAD_MULTIPLIER=1.3

# Optional: Google Maps for enhanced accuracy
DELIVERY_USE_GOOGLE_MAPS=false
DELIVERY_GOOGLE_API_KEY=
```

### 3.3 Secure the File

```bash
chmod 600 .env
```

### 3.4 Restart Odoo

```bash
sudo systemctl restart odoo
```

---

## Step 4: Configure Driver Portal

**Only if you installed Driver Portal**

### 4.1 Create Security Group

The module creates `group_delivery_driver` automatically.

### 4.2 Add Drivers

1. Go to **Settings → Users & Companies → Users**
2. Select a driver user (or create new)
3. Edit user
4. Under **Other** tab, check **Delivery Driver**
5. Save

### 4.3 Test Access

1. Login as driver user
2. Navigate to: `https://your-odoo.com/driver/deliveries`
3. Verify deliveries appear

**Note:** Deliveries must have `x_driver_id` field set to show up.

---

## Step 5: Configure Dashboard

**Only if you installed Dashboard**

### 5.1 Optional Weather Setup

```bash
cd /path/to/odoo/addons/graybeard_dashboard
cp .env.example .env
nano .env
```

```bash
# Your location coordinates
# Find yours at: https://www.latlong.net/
DASHBOARD_WEATHER_LAT=38.1234
DASHBOARD_WEATHER_LON=-82.5678
```

```bash
chmod 600 .env
sudo systemctl restart odoo
```

### 5.2 Access Dashboard

1. Navigate to **Dashboard** menu in Odoo
2. View real-time metrics
3. Dashboard auto-refreshes every 5 minutes

**Note:** Weather only works for US locations (National Weather Service API).

---

## Step 6: Deploy Flask Apps (Optional)

### Option A: Scheduler

```bash
# Clone repository
cd /opt
sudo git clone https://github.com/Alex-Pennington/graybeard-odoo-scheduler
cd graybeard-odoo-scheduler

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure
cp .env.example .env
nano .env
```

**Set these values:**
```bash
ODOO_URL=http://localhost:8069
ODOO_DB=your_database_name
ODOO_USERNAME=admin
ODOO_PASSWORD=your_secure_password

FLASK_HOST=127.0.0.1
FLASK_PORT=5001
```

```bash
# Secure configuration
chmod 600 .env

# Test run
python app.py
# Should start on http://127.0.0.1:5001
```

**Setup Systemd Service:**

```bash
sudo nano /etc/systemd/system/graybeard-scheduler.service
```

```ini
[Unit]
Description=Graybeard Scheduler
After=network.target odoo.service

[Service]
Type=simple
User=odoo
WorkingDirectory=/opt/graybeard-odoo-scheduler
Environment="PATH=/opt/graybeard-odoo-scheduler/venv/bin"
ExecStart=/opt/graybeard-odoo-scheduler/venv/bin/waitress-serve --port=5001 app:application
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable graybeard-scheduler
sudo systemctl start graybeard-scheduler
sudo systemctl status graybeard-scheduler
```

### Option B: Quo Integration

```bash
# Clone repository
cd /opt
sudo git clone https://github.com/Alex-Pennington/graybeard-odoo-quo-integration
cd graybeard-odoo-quo-integration

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure
cp .env.example .env
nano .env
```

**Set these values:**
```bash
# Odoo connection
ODOO_URL=http://localhost:8069
ODOO_DB=your_database_name
ODOO_USERNAME=admin
ODOO_PASSWORD=your_secure_password

# Quo configuration (get from Quo dashboard)
QUO_SIGNING_SECRET=your_signing_secret_here
QUO_API_KEY=your_api_key_here
QUO_DEFAULT_FROM_NUMBER=+15555555555

# Business settings
BUSINESS_PHONES=+15555555555
DEFAULT_LEAD_OWNER_ID=3  # Odoo user ID
DEFAULT_FROM_EMAIL=info@yourcompany.com

# Server
FLASK_HOST=127.0.0.1
FLASK_PORT=5000
DEBUG_MODE=False
```

```bash
# Secure configuration
chmod 600 .env

# Test run
python quo_odoo_lead_integration.py
```

**Setup Systemd Service:** (similar to scheduler, adjust paths and port)

---

## Step 7: Configure Nginx (For Flask Apps)

```bash
sudo nano /etc/nginx/sites-available/graybeard
```

```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;
    
    # SSL configuration
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # Scheduler
    location /scheduler/ {
        proxy_pass http://127.0.0.1:5001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # Quo Integration Webhook
    location /webhook/quo {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # Quo Integration Director Dashboard
    location /director {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/graybeard /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload
sudo systemctl reload nginx
```

---

## Step 8: Configure Quo Webhook (If Using)

1. Login to Quo dashboard: https://app.openphone.com/
2. Go to **Settings → Webhooks**
3. Click **Create Webhook**
4. Set URL: `https://your-domain.com/webhook/quo`
5. Subscribe to events:
   - `call.completed`
   - `message.received`
6. Copy the **Signing Secret**
7. Add to `.env` file: `QUO_SIGNING_SECRET=your_secret_here`
8. Restart the Quo integration service

---

## Step 9: Testing

### Test Odoo Modules

**Delivery Calculator:**
1. Create a sale order
2. Add a customer with an address
3. Add a delivery method
4. Verify delivery cost is calculated

**Driver Portal:**
1. Login as driver user
2. Go to `/driver/deliveries`
3. Verify deliveries appear
4. Test marking complete

**Dashboard:**
1. Navigate to Dashboard menu
2. Verify all metrics load
3. Check weather (if US location)

### Test Flask Apps

**Scheduler:**
- Visit `https://your-domain.com/scheduler/`
- Verify calendar loads
- Test drag-and-drop

**Quo Integration:**
- Make a test call to your business number
- Check Odoo CRM for new lead
- Check Director dashboard: `https://your-domain.com/director`

---

## Troubleshooting

### Module Won't Install
```bash
# Check Odoo logs
sudo tail -f /var/log/odoo/odoo.log

# Update apps list
# In Odoo: Apps → Update Apps List

# Verify module in addons path
ls -la /path/to/odoo/addons/graybeard*
```

### Flask App Won't Start
```bash
# Check if port is available
sudo netstat -tlnp | grep 5001

# Check logs
journalctl -u graybeard-scheduler -f

# Verify virtual environment
source venv/bin/activate
python --version  # Should be 3.8+
```

### Delivery Costs Not Calculating
```bash
# Check .env file exists
ls -la /path/to/odoo/addons/graybeard_odoo_delivery_calculator/.env

# Verify values are set
cat .env | grep DELIVERY_ORIGIN

# Check Odoo logs for errors
sudo tail -f /var/log/odoo/odoo.log
```

### Webhook Not Working
```bash
# Test public accessibility
curl https://your-domain.com/webhook/quo

# Check Flask app logs
journalctl -u graybeard-quo -f

# Verify signing secret matches Quo dashboard
cat .env | grep QUO_SIGNING_SECRET
```

---

## Next Steps

1. ✅ **Security Hardening** - Review [SECURITY.md](./SECURITY.md)
2. ✅ **Production Setup** - Configure backups, monitoring
3. ✅ **Customization** - Adjust settings for your business
4. ✅ **Training** - Train staff on new interfaces
5. ✅ **Monitoring** - Set up logging and alerting

---

## Getting Help

- 📚 **Documentation** - Each repository has detailed README
- 🐛 **Issues** - Report bugs in specific repository
- 💬 **Discussions** - Ask questions on GitHub
- 📧 **Support** - Check DEPLOYMENT_GUIDE.md for troubleshooting

---

**Ready to Deploy?** Follow this guide step-by-step and you'll be up and running! 🚀
