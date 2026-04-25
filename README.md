# 🛒 AWS E-Commerce Platform with OTP-Based Authentication

A production-style, full-stack e-commerce web application deployed on **Amazon Web Services**, featuring a two-step OTP email verification system for secure user sign-up and sign-in.

---

## 📌 Project Overview

This project demonstrates a secure, multi-tier cloud architecture built on AWS. End users can browse and shop products across categories — **Phones, Computers, Electronics, and Earphones** — and must verify their identity via a 6-digit One-Time Password (OTP) delivered to their email during both registration and login.

---

## 🏗️ Architecture

```
Internet
    │
    ▼
Internet-facing ALB  ◄──── ACM (SSL/TLS Certificate — HTTPS)
    │
    ▼
┌─────────────────────────────────────────────┐
│              Public Subnet (VPC)             │
│  ┌──────────────────┐                        │
│  │   Bastion Host   │  ← SSH (port 22 only)  │
│  │   (EC2)          │                        │
│  └──────────────────┘                        │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│                  Private Subnet (VPC)                    │
│                                                          │
│  ┌────────────────────┐     ┌──────────────────────┐    │
│  │  Frontend EC2       │────▶│    Internal ALB      │    │
│  │  HTML/CSS/JS        │     └──────────┬───────────┘    │
│  └────────────────────┘                │               │
│                                        ▼               │
│                              ┌─────────────────────┐   │
│                              │   Backend EC2        │   │
│                              │   Python Flask API   │   │
│                              └────────┬────────────┘   │
│                                       │                 │
│                    ┌──────────────────┴──────────┐      │
│                    ▼                             ▼      │
│          ┌──────────────────┐      ┌──────────────────┐ │
│          │   RDS MySQL      │      │   Gmail SMTP      │ │
│          │   (port 3306)    │      │   (Flask-Mail)    │ │
│          └──────────────────┘      └──────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

---

## ☁️ AWS Services Used

| Service | Purpose |
|---|---|
| **EC2** | Frontend and Backend application servers (private subnet) |
| **VPC** | Custom isolated network with public/private subnets |
| **Application Load Balancer** | Internet-facing ALB (frontend) + Internal ALB (backend) |
| **ACM** | SSL/TLS certificate for HTTPS on the domain |
| **RDS MySQL** | Managed relational database for user and OTP data |
| **Security Groups** | Least-privilege firewall rules per component |
| **Bastion Host** | Secure SSH jump server in the public subnet |

---

## 🔐 Security Design

- All application servers live in the **private subnet** — no direct internet access
- **Bastion Host** is the only SSH entry point (Security Group: port 22 only)
- **Frontend SG** — allows HTTP/HTTPS from ALB only
- **Backend SG** — allows traffic from Internal ALB only
- **RDS SG** — allows port 3306 exclusively from the Backend Security Group
- **ACM certificate** enforces HTTPS end-to-end on the public domain
- Passwords stored as **bcrypt hashes** (Werkzeug)

---

## 🔑 OTP Authentication Flow

### Sign-Up
```
User fills form → POST /api/signup/request
→ OTP generated (6-digit, 10 min expiry)
→ OTP stored in memory (pending_users dict)
→ Email sent via Flask-Mail (Gmail SMTP)
→ User enters OTP → POST /api/signup/verify
→ OTP validated → User inserted into RDS MySQL
→ Account created ✓
```

### Sign-In
```
User enters credentials → POST /api/login/request
→ Password verified against bcrypt hash in RDS
→ OTP generated (6-digit, 5 min expiry)
→ OTP stored in RDS (otp_code, otp_expiry columns)
→ Email sent via Flask-Mail (Gmail SMTP)
→ User enters OTP → POST /api/login/verify
→ OTP + expiry validated → Login successful ✓
```

---

## 🗂️ Project Structure

```
aws-ecommerce-otp-verification/
│
├── frontend/
│   ├── index.html          # Main landing / auth page
│   ├── style.css           # Glassmorphic dark/light theme
│   ├── script.js           # Cart, search, product catalog
│   ├── phones/
│   ├── computers/
│   ├── electronics/
│   ├── earphones/
│   └── googlemusic/        # Bonus: browser-based music player
│
├── backend/
│   ├── app.py              # Flask API — OTP, signup, login
│   ├── requirements.txt    # Python dependencies
│   └── test.sql            # Database schema
│
└── README.md
```

---

## 🛠️ Backend Setup

### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/aws-ecommerce-otp-verification.git
cd aws-ecommerce-otp-verification/backend
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Configure environment variables
Update the following in `app.py` (or use environment variables in production):
```python
db_config = {
    "host": "<your-rds-endpoint>",
    "user": "<db-username>",
    "password": "<db-password>",
    "database": "cloud"
}

MAIL_USERNAME = '<your-email>'
MAIL_PASSWORD = '<your-app-password>'
```

### 4. Set up the database
```bash
mysql -h <rds-endpoint> -u <username> -p < test.sql
```

### 5. Run the Flask server
```bash
python app.py
```
Server runs on `http://0.0.0.0:5000`

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/` | Health check |
| POST | `/api/signup/request` | Send OTP to email for registration |
| POST | `/api/signup/verify` | Verify OTP and create account |
| POST | `/api/login/request` | Validate credentials and send login OTP |
| POST | `/api/login/verify` | Verify OTP and authenticate user |

---

## 🗃️ Database Schema

```sql
CREATE TABLE users (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    username    VARCHAR(100) NOT NULL UNIQUE,
    email       VARCHAR(150) NOT NULL UNIQUE,
    password    VARCHAR(255) NOT NULL,       -- bcrypt hash
    otp_code    VARCHAR(6),                  -- login OTP
    otp_expiry  DATETIME,                    -- OTP validity
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🖥️ Frontend Features

- Multi-page product catalog — Phones, Computers, Electronics, Earphones
- Shopping cart with quantity management and live total
- Real-time debounced search and price/rating sort
- Dark / Light theme toggle
- Grid and Carousel (Swiper.js) view modes
- Quick-view product modal
- Scroll reveal and GSAP animations
- Embedded music player with HTML5 Audio API and playlist management

---

## 📦 Tech Stack

**Cloud:** AWS EC2 · VPC · ALB · ACM · RDS MySQL · Security Groups

**Backend:** Python · Flask · PyMySQL · Flask-Mail · Werkzeug

**Frontend:** HTML5 · CSS3 · Vanilla JavaScript · Swiper.js · GSAP

---

## ⚠️ Known Improvements (for production)

- Move credentials to **AWS Secrets Manager** or environment variables
- Replace in-memory `pending_users` with **Redis** for OTP persistence across restarts
- Add **JWT tokens** for session management after login
- Enable **CloudWatch** logging for ALB and EC2 instances
- Add **Auto Scaling Groups** for high availability

---

## 📄 License

This project is intended for educational and portfolio purposes.
