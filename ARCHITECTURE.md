# Architecture

**Graybeard Odoo Suite - Technical Architecture**

---

## System Overview

The Graybeard Odoo Suite consists of two types of components that work together to extend Odoo's capabilities:

1. **Odoo Modules** - Native Odoo extensions (Python + XML)
2. **Flask Applications** - Standalone web services that integrate via XML-RPC

```
┌───────────────────────────────────────────────────────────────┐
│                         Odoo 19 Core                           │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐              │
│  │  Delivery  │  │   Driver   │  │ Dashboard  │              │
│  │ Calculator │  │   Portal   │  │            │              │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘              │
│        │                │                │                     │
│        └────────────────┴────────────────┘                     │
│                    Odoo ORM & Views                            │
│  ┌─────────────────────────────────────────────────┐          │
│  │              XML-RPC API                         │          │
│  └──────────────┬─────────────────┬─────────────────┘         │
└─────────────────┼─────────────────┼───────────────────────────┘
                  │                 │
         ┌────────▼───────┐  ┌─────▼──────────┐
         │   Scheduler    │  │ Quo Integration │
         │   Flask App    │  │   Flask App     │
         │   Port 5001    │  │   Port 5000     │
         └────────┬───────┘  └─────┬───────────┘
                  │                │
         ┌────────▼────────────────▼─────────┐
         │      Nginx Reverse Proxy          │
         │      HTTPS Termination             │
         └────────────────────────────────────┘
```

---

## Component Architecture

### Odoo Modules

#### 1. Delivery Calculator
**Type:** Odoo Module  
**Purpose:** GPS-based delivery cost calculation

**Components:**
- `models/res_partner.py` - Partner address geocoding
- `models/delivery_carrier.py` - Delivery cost calculation logic
- `views/` - Partner and carrier views
- External API: Google Maps Distance Matrix (optional)

**Data Flow:**
```
Customer Address → Geocoding → Distance Calculation → Cost Calculation → Sale Order
                                     ↓
                            Google Maps API (optional)
                                     ↓
                            Haversine Formula (fallback)
```

**Configuration:**
- Environment variables (`.env`)
- No database configuration

---

#### 2. Driver Portal
**Type:** Odoo Module  
**Purpose:** Mobile-friendly driver interface

**Components:**
- `controllers/driver_portal.py` - Web routes (`/driver/deliveries`)
- `views/driver_portal_templates.xml` - Mobile-optimized views
- `security/` - Driver security groups

**Data Flow:**
```
Driver Login → Odoo Auth → Filter by x_driver_id → Display Deliveries
                                                           ↓
                                              Mark Complete / Record Payment
                                                           ↓
                                               Update Sale Order / Create Payment
```

**Security:**
- Uses Odoo's built-in authentication
- Security group: `group_delivery_driver`
- Filters data by assigned driver

---

#### 3. Dashboard
**Type:** Odoo Module  
**Purpose:** Real-time operations metrics

**Components:**
- `models/graybeard_dashboard.py` - Data aggregation model
- `static/src/components/` - Owl framework components
- `static/src/scss/` - Dashboard styling
- External API: National Weather Service (optional, US only)

**Data Flow:**
```
Dashboard Load → JavaScript Component → RPC Call → Python Model → Query Odoo Data
                                                           ↓
                                                   Weather API (optional)
                                                           ↓
                                                    Return JSON → Render
```

**Technology Stack:**
- Backend: Python (Odoo ORM queries)
- Frontend: Owl Framework (reactive JavaScript)
- Styling: Bootstrap 5 + Custom SCSS
- Auto-refresh: 5-minute interval

---

### Flask Applications

#### 4. Scheduler
**Type:** Flask Application  
**Purpose:** Advanced delivery and manufacturing scheduling

**Components:**
- `app.py` - Main Flask application
- `routes/` - Office, driver, manufacturing views
- `static/` - JavaScript (drag-and-drop)
- `templates/` - Jinja2 templates
- SQLite database - Scheduler state

**Data Flow:**
```
User Action → Flask Route → XML-RPC to Odoo → Fetch Data → Render View
                                    ↓
                        Update Sale Order / MRP Order
                                    ↓
                        SQLite (cache scheduler state)
```

**Integration Points:**
- Odoo XML-RPC API
- Google Calendar API (optional)
- SMS Gateway (optional)

**Architecture Pattern:**
```
Flask App (Waitress WSGI)
     ↓
SQLAlchemy (ORM)
     ↓
SQLite (scheduler state)
     ↓
XML-RPC Client
     ↓
Odoo (sale.order, mrp.production, fleet.vehicle, hr.employee)
```

---

#### 5. Quo Integration
**Type:** Flask Webhook Service  
**Purpose:** Automated CRM lead creation from phone/SMS

**Components:**
- `quo_odoo_lead_integration.py` - Main Flask application (~20k lines)
- Webhook endpoints: `/webhook/quo`
- Director dashboard: `/director`
- Multiple SQLite databases:
  - `consent_ledger.db` - Opt-in/opt-out tracking
  - `sms_messages.db` - SMS history
  - `call_metadata.db` - Call tracking
  - `error_logs.db` - Error history

**Data Flow:**
```
Quo Webhook → Verify Signature → Parse Event → Check Existing Customer
                                                       ↓
                                            Create CRM Lead (XML-RPC)
                                                       ↓
                                          Update Consent DB / SMS History
```

**Integration Points:**
- Quo Webhooks (inbound)
- Quo API (outbound SMS)
- Odoo XML-RPC (CRM leads, partners)
- Google Maps API (optional, address autocomplete)

**Security:**
- Webhook signature verification (HMAC-SHA256)
- Odoo authentication for Director dashboard
- API token for automation endpoints

---

## Data Flow Patterns

### Pattern 1: Odoo Module Direct Integration
```
User Interaction → Odoo View → Odoo Controller → Odoo Model → Database
```
- **Used by:** Delivery Calculator, Driver Portal, Dashboard
- **Advantages:** Tight integration, no additional infrastructure
- **Disadvantages:** Limited to Odoo's request/response cycle

### Pattern 2: External Service via XML-RPC
```
External Event → Flask App → XML-RPC → Odoo Model → Database
```
- **Used by:** Scheduler, Quo Integration
- **Advantages:** Independent scaling, advanced UI, webhook support
- **Disadvantages:** Additional server setup, separate deployment

---

## API Integration

### Odoo XML-RPC API

All Flask applications use Odoo's XML-RPC API for integration:

```python
# Authentication
common = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/common')
uid = common.authenticate(db, username, password, {})

# Object API
models = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/object')
result = models.execute_kw(db, uid, password,
    'crm.lead', 'create',
    [{'name': 'New Lead', 'phone': '+1234567890'}])
```

**Operations:**
- `search` - Find records matching criteria
- `read` - Read field values from records
- `create` - Create new records
- `write` - Update existing records
- `unlink` - Delete records
- `search_read` - Combined search + read

### External APIs

**Google Maps:**
- Distance Matrix API - Road distance calculation
- Geocoding API - Address to coordinates
- Places API - Address autocomplete

**Quo/OpenPhone:**
- Webhook Events - `call.completed`, `message.received`
- REST API - Outbound SMS, contact management

**National Weather Service:**
- Free API - US locations only
- No API key required
- 3-day forecast and alerts

---

## Database Schema

### Odoo Modules
Use standard Odoo ORM models. No custom database tables created directly.

**Delivery Calculator:**
- Extends: `res.partner`, `delivery.carrier`
- No new tables

**Driver Portal:**
- Uses: `sale.order`, `account.payment`, `res.partner`
- No new tables

**Dashboard:**
- Read-only access to all standard Odoo tables
- No new tables

### Flask Applications

**Scheduler SQLite:**
```sql
-- Simplified schema example
CREATE TABLE scheduler_state (
    id INTEGER PRIMARY KEY,
    sale_order_id INTEGER,
    scheduled_date TEXT,
    driver_id INTEGER,
    truck_id INTEGER
);
```

**Quo Integration SQLite:**
```sql
-- Consent ledger
CREATE TABLE consent_ledger (
    phone TEXT PRIMARY KEY,
    consent_status TEXT,
    verification_sent_at TEXT,
    opted_in_at TEXT
);

-- SMS history
CREATE TABLE sms_messages (
    id INTEGER PRIMARY KEY,
    phone TEXT,
    direction TEXT,
    message_text TEXT,
    created_at TEXT
);

-- Call metadata
CREATE TABLE call_metadata (
    id INTEGER PRIMARY KEY,
    phone TEXT,
    duration INTEGER,
    answered BOOLEAN,
    created_at TEXT
);
```

---

## Security Architecture

### Authentication & Authorization

**Odoo Modules:**
- Uses Odoo's built-in authentication
- Security groups control access
- Row-level security via record rules

**Flask Applications:**
- Odoo username/password for API calls
- Webhook signature verification (HMAC-SHA256)
- API tokens for automation endpoints
- Session-based auth for Director dashboard

### Data Protection

**In Transit:**
- HTTPS for all production deployments
- TLS 1.2+ required
- Certificate pinning for webhooks

**At Rest:**
- `.env` files with 600 permissions
- SQLite databases with appropriate file permissions
- No plaintext passwords in code

**Credential Management:**
- All credentials in `.env` files
- Never committed to version control
- `.gitignore` enforces this

---

## Deployment Architecture

### Recommended Production Setup

```
┌─────────────────────────────────────────────────────┐
│                  Internet                            │
└──────────────────────┬──────────────────────────────┘
                       │ HTTPS (443)
              ┌────────▼────────┐
              │  Nginx (Reverse │
              │     Proxy)       │
              └────┬─────┬───────┘
                   │     │
        ┌──────────┘     └──────────┐
        │                           │
   Port 8069                   Port 5000/5001
        │                           │
┌───────▼───────┐          ┌────────▼────────┐
│  Odoo 19      │          │  Flask Apps     │
│  (Modules)    │◄─────────┤  (XML-RPC)      │
└───────┬───────┘          └─────────────────┘
        │
┌───────▼───────┐
│  PostgreSQL   │
└───────────────┘
```

**Components:**
- **Nginx** - HTTPS termination, reverse proxy, load balancing
- **Odoo** - Core application + Graybeard modules
- **PostgreSQL** - Main database
- **Flask Apps** - Scheduler & Quo Integration (separate processes)
- **SQLite** - Flask app state (could use PostgreSQL instead)

---

## Scalability Considerations

### Odoo Modules
- Scale with Odoo's worker processes
- Consider read replicas for Dashboard queries
- Use Odoo's built-in caching

### Flask Applications
- Can run multiple instances behind load balancer
- SQLite limits concurrent writes - consider PostgreSQL for high traffic
- Horizontal scaling possible with shared database

---

## Monitoring & Observability

### Logs

**Odoo:**
- `/var/log/odoo/odoo.log`
- Log level configurable in `odoo.conf`

**Flask Apps:**
- Application logs to stdout/stderr
- Configurable log levels
- Structured logging recommended

### Health Checks

**Odoo:**
- HTTP health endpoint: `/web/health`

**Flask Apps:**
- HTTP health endpoint: `/health`
- Returns JSON with service status

### Metrics

Recommended tools:
- Prometheus for metrics collection
- Grafana for visualization
- AlertManager for alerting

---

## Performance Optimization

### Odoo Modules

**Delivery Calculator:**
- Cache geocoding results
- Batch distance calculations
- Use indexed fields for queries

**Dashboard:**
- Minimize ORM queries
- Use `search_read` instead of separate `search` + `read`
- Cache weather data (5-minute TTL)

### Flask Applications

**Scheduler:**
- Use connection pooling for XML-RPC
- Cache Odoo data (short TTL)
- Optimize SQLite queries with indexes

**Quo Integration:**
- Use async processing for heavy operations
- Queue webhook processing
- Batch database writes

---

## Disaster Recovery

### Backups

**Odoo:**
- Database: PostgreSQL dumps (daily)
- Filestore: Rsync or similar (daily)

**Flask Apps:**
- SQLite databases (daily)
- `.env` files (secure storage)

### Recovery Procedures

1. Restore PostgreSQL database
2. Restore Odoo filestore
3. Restore Flask SQLite databases
4. Restore `.env` configurations
5. Restart all services
6. Verify integrations

---

## Development Workflow

### Local Development

1. Clone repositories
2. Set up Python virtual environments
3. Configure `.env` files with local settings
4. Run Odoo in development mode
5. Run Flask apps with debug mode enabled
6. Use ngrok for webhook testing

### Testing

**Odoo Modules:**
- Use Odoo's unittest framework
- Test with sample data
- Verify UI in multiple browsers

**Flask Apps:**
- Use pytest for unit tests
- Mock XML-RPC calls
- Test webhooks with tools like Postman

---

## Technology Stack Summary

| Component | Technologies |
|-----------|-------------|
| **Backend** | Python 3.8+, Odoo 19, Flask |
| **Frontend** | Owl Framework, Bootstrap 5, vanilla JavaScript |
| **Database** | PostgreSQL (Odoo), SQLite (Flask) |
| **Web Server** | Nginx, Waitress (WSGI) |
| **APIs** | XML-RPC, REST, Webhooks |
| **External Services** | Google Maps, Quo/OpenPhone, National Weather Service |

---

**Document Version:** 1.0  
**Last Updated:** 2025-12-28
