---
name: navigate-platform
description: Navigate and explore the EV Connect Management Console staging environment. Login, browse networks/orgs/stations, verify feature behavior, investigate platform questions. Use when an agent needs to interact with the staging platform via OpenBrowser-AI.
allowed-tools:
  - mcp__openbrowser-ai__execute_code
  - Read
  - Glob
  - Grep
---

# Navigate EV Connect Staging Platform

You are navigating the EV Connect Management Console staging environment to investigate, verify, or explore platform features. This skill provides everything you need to log in and navigate autonomously.

**Important**: Zendesk Help Center documentation may be outdated. Always verify against what you actually see in the platform. If something doesn't match the docs, trust what you see.

## Prerequisites

Create a `.env` file in your project root with:

```
STAGING_URL=https://ops-stage.evconnect.com
STAGING_EMAIL=your-agent@evconnect.com
STAGING_PASSWORD=your-password-here
```

You also need the [OpenBrowser-AI](https://github.com/openbrowser-ai/openbrowser) MCP server configured.

---

## Login Flow

```python
from dotenv import load_dotenv
import os

load_dotenv('path/to/your/.env')
url = os.environ['STAGING_URL']        # https://ops-stage.evconnect.com
email = os.environ['STAGING_EMAIL']
password = os.environ['STAGING_PASSWORD']

# Navigate to any page — login form appears if session expired
await navigate(f'{url}/admin/platform/networks')
await wait(5)

# Check if login is needed
content = await evaluate('document.body.innerText.substring(0, 300)')
if 'Login' in content and 'Email' in content:
    state = await browser.get_browser_state_summary()
    for idx, el in state.dom_state.selector_map.items():
        if el.tag_name == 'input':
            input_id = el.attributes.get('id', '')
            if input_id == 'email':
                await input_text(idx, email)
            elif input_id == 'password':
                await input_text(idx, password)
        elif el.tag_name == 'button' and el.attributes.get('type') == 'submit':
            login_btn = idx
        elif el.attributes.get('id') == 'onetrust-accept-btn-handler':
            await click(idx)  # dismiss cookie banner
            await wait(1)
    await click(login_btn)
    await wait(5)
```

---

## Platform Hierarchy

```
Platform (EV Connect)
  └── Network (e.g., IONNA, bp pulse, EV Connect)
        └── Organization (e.g., site host company)
              └── Location (physical address with map coordinates)
                    └── Station (charge point with EVSEs and connectors)
```

- **Platform level**: Reserved for EV Connect internal admins. Manages all networks, manufacturers, system settings.
- **Network level**: A partner/CPO network. Has its own dashboard, stations, drivers, organizations, settings, and feature toggles.
- **Organization level**: A business entity within a network (site host). Has its own pricing, locations, stations, fleet, and feature toggles.
- **Station level**: Individual charge point with EVSEs (Electric Vehicle Supply Equipment), each having one or more connectors.

### User Roles

| Level | Roles |
|-------|-------|
| **Platform** | Admin (full access), Operations (no admin user mgmt), Staff (read-only) |
| **Network** | Network Admin (full), Network Operator (no user mgmt), Customer Service (driver support only) |
| **Organization** | Organization Operator (edit within org), Organization Viewer (read-only) |
| **Other** | Manufacturer roles (separate portal), Partner/Utility roles (restricted portal) |

---

## Navigation Rules

1. **ALWAYS navigate by URL** — never click sidebar elements. Element indices change on every interaction.
2. **Wait 5-8 seconds** after each navigation for the React SPA to render.
3. **Verify page loaded**: Check `document.body.innerText` — empty string means still loading or no permission.
4. **Use `evaluate()` for data extraction** — more reliable than element text for structured data.

---

## URL Structure

### Platform Level

| Page | URL |
|------|-----|
| Networks list | `/admin/platform/networks` |
| Create network | `/admin/platform/networks/create` |
| Manufacturers | `/admin/platform/manufacturers` |
| Platform Users | `/admin/platform/users` |
| Station Diagnostics / C-NOC | `/admin/platform/cnoc` |
| System: Vehicle Models | `/admin/platform/system/vehicle-models` |
| System: SSO Settings | `/admin/platform/system/sso-settings` |
| System: EV Charging Taxes | `/admin/platform/system/ev-charging-taxes` |

### Network Level

Pattern: `/admin/platform/{network-slug}/network/config/{page}`

| Page | Suffix | Description |
|------|--------|-------------|
| Dashboard | `/dashboard` | Connector status (live), utilization, sessions, energy, revenue. Filter by org/date/timezone. |
| Data | `/data` | Data exports and integrations (requires Data Export plugin) |
| Drivers | `/drivers` | Driver management — search, view profiles, subscriptions |
| Organizations | `/organizations` | List of organizations in the network |
| Stations | `/stations` | All stations. Filter by connector status, org, location. Bulk actions. |
| Users | `/users` | Network user management |
| Search Stations | `/station-search` | Search by Charge Box ID, QR Code, Serial Number, or Zip Code |
| Features | `/plugins` | Feature toggle management (enable/disable plugins) |
| Settings | `/tabs` | Network configuration sub-pages |
| Network Details | `/details` | Name, support contact, party ID, countries, currencies, OCPI code |
| Partner API | `/partner-api` | API client management (only visible if Partner API plugin enabled) |

#### Station Filters (query params on `/stations`)

`?connectorServiceStatus=AVAILABLE|IN_USE|ERROR|IN_MAINTENANCE|OFFLINE|ALL`

#### Settings Sub-pages (listed on `/tabs`)

- Payment Management (merchant account, Payter, Fiserv)
- Location Properties
- Station Properties
- Configure Reporting User
- Mobile and Notification Configuration
- Privacy Settings
- API Key Configuration
- Network Assets

### Organization Level

Pattern: `/admin/platform/{network-slug}/organizations/{org-uuid}/{page}`

| Page | Suffix | Description |
|------|--------|-------------|
| Dashboard | `/dashboard` | Same metrics as network but scoped to this org |
| Pricing and Plans | `/pricing` | Pricing policies and station assignments |
| Data | `/data` | Data exports scoped to org |
| Locations | `/locations` | Map + list of physical locations |
| Stations | `/stations` | Stations in this org |
| Users | `/users` | Org-level users |
| Search Stations | `/station-search` | Search within org |
| Fleet | `/fleet` | Fleet management (requires Fleet Plugin) |
| Features | `/plugins` | Org-level feature toggles |
| Organization Details | `/details` | Org info |

### Station Detail

Pattern: `/admin/platform/{network-slug}/organizations/{org-uuid}/stations/{station-uuid}`

Tabs: Event Log, Station Details, Payment, Configuration, Sessions (`/sessions`), Support

Shows: service uptime %, charge attempts, 7-day connector timeline, EVSE/connector details, manufacturer/model, firmware version, OCPP status.

---

## Feature / Plugin System

Features are toggled at TWO levels:
1. **Network level** — enables the capability for the entire network
2. **Organization level** — enables it for a specific org within that network

Some features have dependencies — they must be enabled in a specific order. API plugins are grayed out until their parent feature plugin is enabled at the network level.

### Feature Catalog

#### Access
- Fleet Plugin, Fleet Home Charging, Google Maps Plugin, Partner Management Plugin
- Plug and Charge Station Configuration, Roaming Plugin, Security Profile 3
- Single Sign On, Utility Partner Management Plugin

#### API (require parent plugin + Partner API enabled first)
- Channel Partner API, Data API, Driver Experience API, Energy Management API
- Fleet API, Loyalty Programs API, Network Operations and Maintenance API, Partner API

**API Module Bundles** (for Partner API clients):
- Driver Experience: Station Commands, Drivers, Locations, Networks, Payments, Tariffs, Plans, Vehicle Models, Support
- Operations & Maintenance: Organization Management, Location Management, Station Commands, Maintenance Status, Pricing
- Data: NEVI, Sessions, EVSE
- Energy Management: EVSE Properties, Energy Management Limits, Locations, Station Groups
- Fleet: Charge Windows, Organizations, Home Charging, Vehicles, Vehicle Integrations, Vehicle Fobs, Reporting

#### Customer Support
- C-NOC Zendesk Auto-Ticketing (requires Customer Support Plugin first)
- Customer Support Plugin (Zendesk integration)
- Station Diagnostics Tool

#### Data
- Dashboard Benchmark, Data Export And Integrations Plugin
- NEVI API and Reporting Access Plugin, Sustainability Reporting

#### Driver Experience
- Loyalty (EVoucher campaigns), Subscription (paid/free plans)
- Subscription Credits (dollar credit benefit)

#### Messaging
- Snitching Plugin (parking enforcement reporting)

#### Payment
- Bundled Payments Plugin, Dynamic Pricing Component (Norway/UK only)
- Payment Plugin (required for any paid charging)
- Payout Reporting, Per Minute Pricing
- SOC Pricing Modifier (State of Charge based)
- Tax Support Plugin, Time of Use Pricing Lock

#### Privacy
- Consent Management (cookie management)

#### Station
- Load Sharing Management Plugin, Remote Stop by User
- Reservations, Whole Site Load Management (Alpha)

#### Utility
- Demand Response Plugin (OpenADR VTN), Utility Data Plugin

---

## Station Management

### Finding Stations
- **Station list**: `/stations` — filter by connector status, org, location
- **Search**: `/station-search` — by Charge Box ID, QR Code, Serial Number, or Zip Code
- **Advanced filters** on station list: manufacturer, firmware version, access type, credit card reader, pricing policy, OCPP version, connectivity (filters use AND logic)

### Station Detail Page Tabs
- **Event Log**: Timeline of station events (status changes, maintenance)
- **Station Details**: Certification status, map pin, access settings (Public/Private, Hide on Map), warranty
- **Configuration**: Communication (Wi-Fi/LAN/Cellular), OCPP version (1.6 or 2.0.1), energy management
- **Payment**: Pricing policy assignment, Freevend mode toggle, credit card readers (Payter/Nayax/Ingenico)
- **Sessions**: Filter by date range or status (Paid, Invalid, Pending)
- **Support**: Zendesk ticket integration

### Station Actions
- Send OCPP Command, Edit Station, Set Station Message (shown in mobile app)
- Change Model, Change Network (Platform Admin only)
- Revert to Commissioning, Decommission (keeps data but removes from active use)
- Firmware Update (single or bulk)

### Station Lifecycle
Pre-Commissioning → Commissioning → Active → Decommission

---

## Pricing Configuration

### Pricing Policy Structure
A **Pricing Policy** (tariff) = one or more **Pricing Components** + optional **Restrictions**

### Pricing Components
| Component | Description |
|-----------|-------------|
| Flat Fee | One-time dollar amount per session |
| Metered by Charging Time | $/hour while actively charging (per minute or per hour) |
| Metered by Parking Time | $/hour while connected but not charging |
| Metered by Energy | $/kWh delivered |
| Metered by Energy (Market Price) | Dynamic pricing from utility rates (Norway/UK only) |

### Restrictions (stackable per component)
- Session Duration (after X min, range X-Y min)
- Energy Consumption (after X kWh, range X-Y kWh)
- Day of Week
- Time of Day (range)

### Pricing Setup Workflow
1. Org → Pricing and Plans → Add Pricing Policy
2. Name + Custom Description (shown on receipts)
3. Add components and restrictions
4. Save → Assign to stations via Payment Tab or Bulk Actions

**Time of Use Price Lock**: Locks time-based pricing at session start. Requires feature toggle. Applies to all session types.

---

## C-NOC (Station Diagnostics)

Requires **Station Diagnostics Tool** plugin enabled at both network and org level.

### C-NOC Metrics
| Metric | Description |
|--------|-------------|
| Service Uptime | Station availability percentage |
| Session Success | Successful vs failed sessions |
| In Maintenance | Stations in maintenance mode |
| Station Connectivity | Online/offline status |
| Faults | Station fault events |
| Failed Commands | OCPP command failures |
| Resets | Station reset events |
| Boot Notifications | Station boot events |
| Uncertified Stations | Non-certified configurations |

Each metric has a detail page with filtering by org, location, station, manufacturer, model, time period (up to 30 days, within last 2 years). Export to CSV available.

---

## Organization Management

- Organizations live within networks. No limit on count.
- **Cannot be deleted** after creation (locations can be deleted if no stations assigned).
- Organization name is shown in Mobile App and roaming.
- Selecting an org in the console reloads all data scoped to that org.
- Return to network view by clicking the Network name above the sidebar.

---

## Zendesk Help Center Article Reference

Key articles for deeper research when investigating specific questions:

| Topic | Article ID | Title |
|-------|-----------|-------|
| Console Overview | 29907994256795 | Logging in to the Management Console |
| Console Setup | 29907882011931 | Management Console Setup |
| Feature Toggles | 45953566440475 | Feature Toggles |
| Network Config | 29908033505051 | Network Configuration |
| Org Settings | 29908017973275 | Organization Settings |
| Org Console View | 29908013133851 | Using the Management Console as an Organization |
| User Roles | 47453823260571 | Understanding User Roles |
| Managing Stations | 29908041167131 | Managing Existing Stations |
| Bulk Actions | 42695554316699 | Using Bulk Actions |
| Firmware | 43337599916443 | Managing Station Firmware |
| Station FAQ | 29907987428635 | FAQ: Station management |
| Pricing | 29908072454427 | Configuring pricing policy |
| Time of Use Lock | 40469731718555 | Time of Use Price Lock |
| C-NOC | 38805872555035 | Station Diagnostics: C-NOC |
| Mobile Config | 41492162740891 | Mobile and Network Configuration |
| Credit Subscriptions | 46515905727771 | Credit Subscription |
| Subscriptions Guide | 31144848886683 | Subscriptions User Guide |
| Payouts | 41572146295835 | Payout - Site Hosts |
| Credit Card Readers | (section 36795748660507) | 8 articles |

Use `mcp__zendesk__get_help_center_article` with the article ID for full content. Note: docs may be outdated — verify against what you see in the platform.

---

## Common Investigation Patterns

### "Does the platform support X?"
1. Navigate to a network with all features enabled
2. Check `/plugins` to see if there's a feature toggle for it
3. Check the relevant section (Settings, Pricing, Stations, etc.)
4. Search Zendesk Help Center for documentation

### "How do you configure X?"
1. Navigate to the relevant settings page
2. Read the page content via `evaluate('document.body.innerText')`
3. Check for sub-tabs or expandable sections
4. Cross-reference with Zendesk article if available

### "What does X look like in the platform?"
1. Navigate to the relevant page
2. Extract content and interactive elements
3. Report what's visible

### "Is there a station with issue Y?"
1. Use Search Stations (`/station-search`) with Charge Box ID or serial number
2. Or browse stations list (`/stations`) filtered by connector status
3. Click into station detail for Event Log and Sessions
4. Check C-NOC metrics if diagnostics question

### "Verify a pricing configuration"
1. Navigate to org → Pricing and Plans (`/pricing`)
2. Check pricing policies — components and restrictions
3. Check station-level Payment tab for policy assignment

### "Check a driver account"
1. Navigate to network → Drivers (`/drivers`)
2. Search by email, name, or driver ID
3. View subscriptions, sessions, payment methods

---

## Troubleshooting

- **Blank page / empty content**: React SPA still loading. Wait 5-8 more seconds and retry.
- **Login form reappears**: Session expired. Re-run login flow.
- **Cookie banner blocking**: Click "Accept All Cookies" button (id: `onetrust-accept-btn-handler`).
- **"Incorrect email or password"**: Check `.env` credentials. Password has special chars — verify `input_text` entered it correctly.
- **Feature toggle bouncing**: May require dependency enabled first, or admin permissions. Check if it's grayed out (disabled attribute).
- **Page shows "Loading..."**: Some pages (Drivers, Data) take 10+ seconds. Wait and retry.
