# Graybeard Odoo Suite

**Professional Odoo modules and applications for delivery, logistics, and service businesses.**

[![License: LGPL-3](https://img.shields.io/badge/License-LGPL%20v3-blue.svg)](https://www.gnu.org/licenses/lgpl-3.0)
[![Odoo Version](https://img.shields.io/badge/Odoo-19.0-875A7B.svg)](https://www.odoo.com/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)

---

## 🚀 What is Graybeard Odoo Suite?

A comprehensive collection of **production-ready Odoo modules and Flask applications** designed for businesses that need sophisticated delivery, scheduling, and customer communication workflows.

Battle-tested in real-world operations managing thousands of deliveries, these modules provide enterprise-grade functionality for growing businesses.

### Perfect For:
- 🚚 Delivery and logistics companies
- 🏭 Manufacturing with delivery operations
- 📞 Service businesses using phone/SMS for customer acquisition
- 📅 Companies needing advanced scheduling beyond Odoo's built-in calendar
- 📊 Operations teams wanting real-time business metrics

---

## 📦 What's Included

### Odoo Modules (3)

| Module | Description | Status |
|--------|-------------|--------|
| [**Delivery Calculator**](https://github.com/Alex-Pennington/graybeard_odoo_delivery_calculator) | GPS-based delivery cost calculation with Google Maps integration | ✅ Public |
| [**Driver Portal**](https://github.com/Alex-Pennington/graybeard_odoo_driver_portal) | Mobile-friendly driver interface for deliveries and payments | ✅ Public |
| [**Dashboard**](https://github.com/Alex-Pennington/graybeard-odoo-dashboard) | Real-time operations dashboard with financial, production, and sales metrics | ✅ Public |

### Flask Applications (2)

| Application | Description | Status |
|-------------|-------------|--------|
| [**Scheduler**](https://github.com/Alex-Pennington/graybeard-odoo-scheduler) | Advanced scheduling system with calendar views, public booking widget, and Google Calendar sync | ✅ Public |
| [**Quo Integration**](https://github.com/Alex-Pennington/graybeard-odoo-quo-integration) | Webhook service for Quo (OpenPhone) that auto-creates CRM leads from calls and SMS | ✅ Public |

---

## ✨ Key Features

### 🗺️ **GPS-Based Delivery Pricing**
Automatically calculate delivery costs based on customer location using Google Maps Distance Matrix API or built-in Haversine formula. Configurable rates, max distances, and road multipliers.

### 📱 **Mobile Driver Experience**
Give drivers a simplified interface to view deliveries, mark complete, and collect payments on-site. No full Odoo access needed - just the essentials.

### 📊 **Real-Time Operations Dashboard**
Get at-a-glance metrics: bank balances, AR/AP, production output, deliveries, sales, low stock alerts, and even local weather (US only). Auto-refreshes every 5 minutes.

### 📅 **Professional Scheduling**
Drag-and-drop delivery scheduling with separate views for office, drivers, and manufacturing. Includes public booking widget for customer self-scheduling.

### 📞 **Automated Lead Capture**
Turn every phone call and text message into a CRM lead automatically. Includes call tracking, SMS history, consent management, and TCPA compliance.

---

## 🎯 Why Choose Graybeard?

### ✅ **Battle-Tested**
Not proof-of-concepts - these modules powered real operations processing thousands of deliveries with 200% year-over-year growth.

### ✅ **Security First**
All credentials externalized to `.env` files. No hardcoded API keys, passwords, or sensitive data. Production-ready security practices.

### ✅ **Well-Documented**
Each module includes comprehensive READMEs, CHANGELOGs, configuration guides, and troubleshooting sections. Get up and running quickly.

### ✅ **Professional Code Quality**
- Clean, maintainable Python code
- Follows Odoo development best practices
- Extensive inline documentation
- Proper error handling and logging

### ✅ **Odoo 19 Ready**
Built for Odoo 19+ with backwards compatibility considerations. Modern Owl framework for reactive dashboards.

---

## 🚀 Quick Start

### 1. Install Odoo Modules

```bash
# Clone the module you need
git clone https://github.com/Alex-Pennington/graybeard_odoo_delivery_calculator
git clone https://github.com/Alex-Pennington/graybeard_odoo_driver_portal
git clone https://github.com/Alex-Pennington/graybeard-odoo-dashboard

# Copy to Odoo addons directory
cp -r graybeard_* /path/to/odoo/addons/

# Restart Odoo
sudo systemctl restart odoo

# Install via Odoo Apps menu
```

### 2. Configure Environment

Each module includes `.env.example` - copy to `.env` and configure:

```bash
cd /path/to/module
cp .env.example .env
nano .env  # Add your settings
chmod 600 .env  # Secure the file
```

### 3. Deploy Flask Apps (Optional)

```bash
# Clone the app
git clone https://github.com/Alex-Pennington/graybeard-odoo-scheduler

# Install dependencies
cd graybeard-odoo-scheduler
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Configure
cp .env.example .env
nano .env

# Run
python app.py
```

📚 **See [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md) for detailed instructions**

---

## 🏗️ Architecture

### How It All Works Together

```
┌─────────────────────────────────────────────────────────────┐
│                        Odoo 19                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Delivery    │  │   Driver     │  │  Dashboard   │      │
│  │ Calculator   │  │   Portal     │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│         ▲                  ▲                  ▲              │
│         │                  │                  │              │
│         └──────────────────┴──────────────────┘              │
│                       XML-RPC API                            │
└─────────────────────────────┬───────────────────────────────┘
                              │
                ┌─────────────┴─────────────┐
                │                           │
      ┌─────────▼──────┐         ┌─────────▼──────┐
      │   Scheduler    │         │ Quo Integration │
      │   Flask App    │         │   Flask App     │
      └────────────────┘         └─────────────────┘
```

- **Odoo Modules** extend core Odoo functionality
- **Flask Apps** run separately, integrate via XML-RPC
- All components communicate through secure APIs
- Environment variables for all configuration

📚 **See [ARCHITECTURE.md](./ARCHITECTURE.md) for technical details**

---

## 📖 Documentation

| Document | Description |
|----------|-------------|
| [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md) | Comprehensive deployment guide for dev teams |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Technical architecture and integration patterns |
| [GETTING_STARTED.md](./GETTING_STARTED.md) | Step-by-step installation walkthrough |
| [SECURITY.md](./SECURITY.md) | Security best practices and configuration |

Each repository also includes:
- Detailed README with features and setup
- CHANGELOG documenting all changes
- `.env.example` configuration template
- Troubleshooting guides

---

## 🛠️ Technology Stack

**Odoo Modules:**
- Python 3.8+
- Odoo 19+ (Community or Enterprise)
- OWL Framework (for Dashboard)
- PostgreSQL

**Flask Applications:**
- Python 3.8+
- Flask + Waitress WSGI
- SQLAlchemy + SQLite
- Odoo XML-RPC Client

**External APIs:**
- Google Maps (Distance Matrix, Geocoding, Places)
- Quo/OpenPhone Webhooks
- National Weather Service (free, US only)

---

## 💡 Use Cases

### Delivery & Logistics
- Automatic delivery pricing based on GPS distance
- Driver mobile interface for on-the-go updates
- Real-time delivery scheduling with drag-and-drop
- Public booking widget for customer self-service

### Manufacturing + Delivery
- Production tracking integrated with delivery scheduling
- Manufacturing calendar view
- Commission calculations for production teams
- Inventory alerts and top product tracking

### Service Businesses
- Phone call tracking with automatic CRM lead creation
- SMS inquiry management
- Customer consent and compliance (TCPA)
- Google Maps address autocomplete

### Operations Management
- Real-time financial metrics (bank, AR/AP, revenue)
- Production output monitoring
- Low stock alerts
- Weather forecasting for planning

---

## 🔒 Security & Compliance

### Security Features
- ✅ All credentials in `.env` files (never committed)
- ✅ Webhook signature verification
- ✅ Odoo authentication required for sensitive endpoints
- ✅ API token protection
- ✅ TCPA compliance features (Quo Integration)
- ✅ Double opt-in verification for SMS

### Best Practices
- File permissions on `.env` (600 on Linux)
- HTTPS required for webhooks
- Regular API key rotation
- Odoo user permission reviews
- Audit logging

📚 **See [SECURITY.md](./SECURITY.md) for detailed security guidance**

---

## 🤝 Contributing

We welcome contributions! Each repository has its own contribution guidelines.

**General Process:**
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

All modules and applications are licensed under **LGPL-3**.

This means you can:
- ✅ Use commercially
- ✅ Modify the code
- ✅ Distribute
- ✅ Use privately

You must:
- 📋 Disclose source
- 📋 License and copyright notice
- 📋 State changes
- 📋 Use same license (LGPL-3)

See [LICENSE](./LICENSE) for details.

---

## 🆘 Support

### Documentation
- Each repository has comprehensive READMEs
- See [GETTING_STARTED.md](./GETTING_STARTED.md) for installation help
- Check [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md) for advanced configuration

### Issues
- Report bugs in the specific module's repository
- Feature requests welcome
- Security issues: Please report privately

### Community
- 💬 Discussions: Use GitHub Discussions on relevant repos
- 📧 Contact: Via GitHub issues

---

## 🗺️ Roadmap

### Planned Features
- [ ] Multi-language support
- [ ] Additional dashboard widgets
- [ ] Advanced reporting modules
- [ ] Integration templates for popular services
- [ ] Docker deployment guides
- [ ] Kubernetes manifests

### In Consideration
- Stripe payment integration module
- Route optimization engine
- Customer portal enhancements
- Additional webhook integrations

**Want to see something?** Open an issue and let us know!

---

## 🙏 Acknowledgments

These modules were originally developed for a delivery business operations managing significant growth. They've been white-labeled and genericized for the broader Odoo community.

Special thanks to the Odoo community and OCA for the excellent platform and ecosystem.

---

## 📊 Stats

- **5 Components:** 3 Odoo modules + 2 Flask apps
- **~20,000 Lines:** Professional Python code
- **Production-Proven:** Battle-tested in real operations
- **Well-Documented:** Comprehensive guides and examples
- **Actively Maintained:** Regular updates and improvements

---

**Built with ❤️ for the Odoo community**

**Questions? Open an issue in the relevant repository or start a discussion!**
