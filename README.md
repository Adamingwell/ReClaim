# ReClaim 📦🔍
> **Intelligent IoT Asset Recovery & Chain-of-Custody Management System**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Status: Prototype / Capstone](https://img.shields.io/badge/Status-Course%20Project-success.svg)]()
[![Architecture: Edge%20%2B%20Cloud](https://img.shields.io/badge/Architecture-Edge%20%2B%20Cloud-orange.svg)]()

---

## 📌 Executive Summary

Traditional lost-and-found operations rely on unorganized cardboard bins, sticky notes, and paper logs. This leads to three major points of failure:
1. **Inefficient manual searches** across fragmented physical locations.
2. **"Guess-and-grab" fraudulent claims** on high-value items (laptops, jewelry, audio gear).
3. **Broken chain-of-custody** with unrecorded physical handoffs.

**ReClaim** modernizes property recovery by combining a public-facing searchable web catalog with an **IoT-enabled physical intake desk** and **asymmetric server-side claim verification**.

---

## ⚙️ Key Technical Features

* **Asymmetric Data Masking:** The public feed renders item category, location, and photos, but the API explicitly strips the private "secret verification detail" before sending JSON payloads to the browser.
* **IoT Hardware Integration:** Front-desk workstations integrate high-speed 2D barcode presentation scanners (Zebra DS9308) and direct thermal label printers (Zebra ZD421) via ESC/POS & ZPL commands.
* **Edge Offline Resilience:** Intake kiosks maintain a local SQLite buffer. If network connectivity drops, staff can continue logging physical property and printing tags locally; changes sync when connectivity restores.
* **PII Guardrails:** Automated client-side blurring and mandatory intake confirmation policies prevent personal identification (driver's licenses, student IDs, credit cards) from appearing publicly.

---

## 🏛️ System Architecture & Data Flow

```
[ FRONTEND CLIENTS ]            [ INGESTION & BALANCING ]         [ CORE SERVICES ]               [ PERSISTENCE & HARDWARE ]

[ Staff Intake Kiosk ] ──┐
                         ├──► [ Load Balancers A–D ] ───► [ REST API Gateway ] ──────┬──► [ PostgreSQL Database ]
[ Claimant Mobile Web ] ─┘                                                           │    (Items, Claims, Audit Logs)
                                                                                     ├──► [ AWS S3 Bucket ]
[ 2D Barcode Scanner ] ─────► [ Edge Gateway (Local SQLite) ] ─► [ IoT MQTT Service ]│    (Encrypted Photos)
                                                                                     └──► [ Zebra Thermal Printer ]
                                                                                          (ZPL / Code-128 Labels)
```

### Data Pipeline Walkthrough
1. **Intake:** Staff logs an item with public descriptors and a hidden secret clue (e.g., *"engraved initials on back"*).
2. **Tagging:** The backend commands the thermal printer over MQTT to spit out a machine-readable Code-128 sticker for the physical storage bin.
3. **Public Discovery:** Claimants browse the sanitized catalog (`GET /api/v1/items`). The server enforces zero client-side leakage.
4. **Claim Filing:** A user submits an ownership claim (`POST /api/v1/claims`) with their proposed matching detail.
5. **Human-in-the-Loop Review:** Staff compares the submitted claim against the stored secret clue, scans the physical barcode upon handoff, and marks the item `Claimed`.

---

## 👥 IT Program Roles & Responsibilities

| Role | Program Area | Core System Contribution |
| :--- | :--- | :--- |
| **Software Developer** | Software Engineering | React/Next.js UI, RESTful API endpoints, server-side data sanitization. |
| **Data Analyst** | Database Administration | Relational schema design (`FoundItems`, `Claims`), loss hotspot analytics. |
| **Cloud / Systems Admin**| Infrastructure & DevOps | AWS/Docker deployment, SAML/SSO role-based access control, automated backups. |
| **Network Admin** | Cybersecurity & Networking | Domain configuration, HTTPS/TLS certificates, isolated intake terminal VLAN. |
| **IT Support Tech** | Technical Support | Kiosk hardware setup (scanners, printers), staff SOP intake documentation. |
| **IoT Specialist** | Embedded Systems | Scanner HID listeners, thermal printer ZPL drivers, edge SQLite sync daemon. |

---

## 🛠️ Tech Stack

* **Frontend:** Next.js / React, Tailwind CSS, Lucide Icons
* **Backend:** Node.js / Express (or Python / FastAPI)
* **Database:** PostgreSQL (Cloud instance + local SQLite edge buffer)
* **Storage:** AWS S3 (Encrypted object storage for item photography)
* **IoT Protocols:** MQTT via AWS IoT Core / Mosquitto, HID Barcode Input, ZPL/ESC-POS
* **Security & Auth:** JWT / Session Auth, Server-Side Data Masking, TLS 1.3

---

## 📁 Repository Structure

```bash
reclaim/
├── api/                   # Backend REST services & business logic
│   ├── controllers/       # Item intake & claim review controllers
│   ├── middleware/        # Server-side data sanitization & auth
│   └── models/            # PostgreSQL schemas (Items, Claims, Logs)
├── edge/                  # IoT device drivers & local offline buffer
│   ├── printer/           # ZPL label template generators
│   └── scanner/           # Barcode HID reader listeners
├── web/                   # Public claimant portal & staff dashboard
│   ├── components/        # UI components (ItemCard, ClaimModal, etc.)
│   └── pages/             # Routing for catalog, intake, and review
├── docs/                  # System diagrams, hardware specs, presentation slides
└── README.md
```

---

## 🚀 Quick Start (Local Prototype Setup)

### Prerequisites
* Node.js (v18.0+)
* PostgreSQL running locally or via Docker
* Modern browser (Chrome / Edge / Firefox)

### 1. Clone the repository
```bash
git clone https://github.com/your-username/reclaim.git
cd reclaim
```

### 2. Configure Environment Variables
Create a `.env` file in the project root:
```env
PORT=5000
DATABASE_URL=postgresql://user:password@localhost:5432/reclaim_db
JWT_SECRET=supersecretkey
S3_BUCKET_NAME=reclaim-item-photos
```

### 3. Install Dependencies & Start Services
```bash
# Install backend and frontend dependencies
npm install

# Run database migrations
npm run db:migrate

# Start development servers
npm run dev
```

Visit `http://localhost:3000` to access the public item feed and `http://localhost:3000/staff` for the staff intake desk.

---

## 📊 Design Decisions & Tradeoffs

* **Server-Side Masking vs. Client-Side CSS Hiding:** We rejected `display: none` or JavaScript filtering on the browser. Keeping secret verification data off client payloads entirely prevents inspection-based theft via browser Developer Tools.
* **Human-in-the-Loop vs. Automated AI Matching:** We traded instant automated string matching for staff review to avoid false positives caused by typos or prompt manipulation.
* **Edge SQLite Buffering:** Physical intake stations remain functional even during network disconnects, preventing lost intake logs during building-wide Wi-Fi outages.
