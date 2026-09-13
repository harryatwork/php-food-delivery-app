<div align="center">

# 🍕 PHP Food Delivery App

**A full-cycle food delivery platform — restaurant listings, menu ordering, live order status tracking, delivery partner management, and a kitchen admin dashboard. Built in PHP with a hybrid mobile-first approach.**

[![PHP](https://img.shields.io/badge/PHP-8.x-777bb4?style=flat-square&logo=php&logoColor=white)](https://php.net)
[![MySQL](https://img.shields.io/badge/MySQL-8.x-4479A1?style=flat-square&logo=mysql&logoColor=white)](https://mysql.com)
[![JavaScript](https://img.shields.io/badge/JavaScript-ES6-F7DF1E?style=flat-square&logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![License](https://img.shields.io/badge/license-MIT-a855f7?style=flat-square)](LICENSE)

[Features](#-features) · [Architecture](#-architecture) · [Quick Start](#-quick-start) · [Order Flow](#-order-flow) · [Configuration](#-configuration)

</div>

---

## The Problem

Food delivery apps like Zomato and Swiggy run a 20–30% restaurant commission that bleeds margins. Small restaurant chains and cloud kitchens need their own ordering system — one they control, with zero platform commission, direct customer relationships, and a kitchen dashboard built around how their operations actually work. This project is that system.

---

## ✨ Features

**Customer Side**
- 🏪 **Restaurant discovery** — browse nearby restaurants with cuisine filters, ratings, and delivery time estimates
- 📋 **Digital menus** — full menu with categories, item photos, prices, and customisations (extras, spice level, portion size)
- 🛒 **Cart with live price summary** — add/remove items, see total, taxes, and delivery fee in real time
- 🔐 **OTP-based authentication** — login with phone number, no password, OTP via SMS
- 💳 **Checkout** — select delivery address from saved addresses or add new, choose payment (online/COD)
- 📍 **Live order tracking** — status updates: Placed → Confirmed → Preparing → Out for Delivery → Delivered
- 📜 **Order history** — reorder any previous order in one tap

**Restaurant / Kitchen**
- 🧑‍🍳 **Kitchen dashboard** — incoming orders queue, order details with preparation timer
- ✅ **One-click status updates** — Confirm → Mark Ready → Assign Delivery Partner
- 🍽️ **Menu management** — toggle items available/unavailable in real time (sold-out handling)
- 📊 **Daily sales summary** — orders today, revenue today, average order value

**Delivery Partner**
- 📲 **Assignment panel** — see assigned orders, accept/reject, navigate to restaurant and customer
- 📍 **Status updates** — Picked Up → Out for Delivery → Delivered

**Admin**
- 🏢 **Restaurant onboarding** — add restaurants, manage menus, set delivery zones
- 👥 **Delivery partner management** — register partners, track active assignments
- 📊 **Platform analytics** — orders per day, revenue per restaurant, top-selling items
- 🎟️ **Promo codes** — flat or percentage discount, minimum order value, usage cap

---

## ⚡ Architecture

```
Customer Browser / Mobile WebApp
       │
       ├── GET /restaurants, /menu, /cart
       ├── POST /order/place
       └── GET /order/track/{id}  ←── polls every 10s for status
              │
    ┌─────────▼──────────────────────────────────────┐
    │              PHP Backend (MVC)                  │
    │  Router → Controllers → Models (PDO + MySQL)   │
    ├────────────┬───────────────┬───────────────────┤
    │ AuthCtrl   │ OrderCtrl     │ TrackingCtrl       │
    │ (OTP/SMS)  │ (Cart,Place,  │ (status poll,      │
    │            │  Confirm)     │  push-via-JS)      │
    └────────────┴───────┬───────┴───────────────────┘
                         │
              ┌──────────▼──────────┐
              │       MySQL          │
              │  restaurants, menus  │
              │  orders, order_items │
              │  delivery_partners   │
              │  users, addresses    │
              └──────────┬──────────┘
                         │
         ┌───────────────┼──────────────────────┐
         ▼               ▼                      ▼
    SMS Gateway    Payment Gateway        Kitchen Dashboard
  (Twilio/MSG91)  (Razorpay/Stripe)    (WebSocket or polling)
```

---

## 🔄 Order Flow

```
Customer places order
      │
      ▼
[PLACED] ──── Restaurant confirms ──► [CONFIRMED]
                                           │
                                 Kitchen prepares food
                                           │
                                           ▼
                                      [PREPARING]
                                           │
                            Delivery partner assigned
                                           │
                                           ▼
                              [OUT FOR DELIVERY] ──► Customer tracks live
                                           │
                                    Delivered
                                           │
                                           ▼
                                      [DELIVERED] ──► Rating prompt
```

---

## 🚀 Quick Start

### Prerequisites

- PHP 8.x + Composer
- MySQL 8.x
- SMS gateway API (Twilio / MSG91) for OTP
- Razorpay or Stripe for online payments

### 1 — Clone and install

```bash
git clone https://github.com/harryatwork/php-food-delivery-app
cd php-food-delivery-app
composer install
```

### 2 — Database

```bash
mysql -u root -p
CREATE DATABASE food_delivery;
USE food_delivery;
SOURCE database/schema.sql;
SOURCE database/seed.sql;   # Sample restaurants, menus, delivery partners
```

### 3 — Configure

```bash
cp config/config.example.php config/config.php
```

Set DB credentials, SMS API keys, and payment gateway credentials.

### 4 — Run

```bash
php -S localhost:8000 -t public
```

- **Customer app:** `http://localhost:8000`
- **Kitchen dashboard:** `http://localhost:8000/kitchen` (restaurant login)
- **Admin panel:** `http://localhost:8000/admin`

---

## 📁 Project Structure

```
php-food-delivery-app/
├── config/
│   ├── config.php
│   └── config.example.php
├── controllers/
│   ├── AuthController.php      # OTP login flow
│   ├── RestaurantController.php
│   ├── MenuController.php
│   ├── CartController.php
│   ├── OrderController.php     # Place order, status updates
│   └── TrackingController.php  # Live status polling endpoint
├── kitchen/
│   ├── DashboardController.php # Kitchen order queue
│   └── views/
├── admin/
│   ├── RestaurantController.php
│   ├── PartnerController.php
│   └── AnalyticsController.php
├── models/
│   ├── Restaurant.php
│   ├── Menu.php
│   ├── Order.php
│   ├── DeliveryPartner.php
│   └── User.php
├── views/
│   ├── home/                   # Restaurant listing
│   ├── menu/                   # Menu + cart
│   ├── checkout/
│   └── tracking/               # Live order status page
├── public/
│   ├── css/
│   └── js/
│       └── tracking.js         # Polls /order/track/{id} every 10s
├── database/
│   ├── schema.sql
│   └── seed.sql
└── index.php
```

---

## ⚙️ Configuration

| Key | Description |
|---|---|
| `DB_HOST` | MySQL host |
| `SMS_PROVIDER` | `twilio` or `msg91` |
| `SMS_API_KEY` | SMS gateway API key |
| `SMS_SENDER_ID` | Sender ID / phone number |
| `PAYMENT_GATEWAY` | `razorpay` or `stripe` |
| `RAZORPAY_KEY_ID` | Razorpay key |
| `RAZORPAY_SECRET` | Razorpay secret |
| `DELIVERY_FEE_BASE` | Base delivery fee (e.g. `49`) |
| `FREE_DELIVERY_ABOVE` | Order amount for free delivery (e.g. `300`) |
| `OTP_EXPIRY_SECONDS` | OTP validity window (default: `300`) |

---

<details>
<summary><strong>Common issues and fixes</strong></summary>

| Issue | Fix |
|---|---|
| OTP not received | Verify SMS API key and sender ID; check if number is in DLT-registered format (India) |
| Order status not updating | Confirm kitchen dashboard is hitting the `/order/update-status` endpoint with correct `order_id` |
| Payment not confirming | Set up payment gateway webhook to `POST /webhooks/payment` |
| Cart resets on login | Cart is session-based; `AuthController::login()` merges guest cart into user cart — check that logic |
| Menu item shows as available when sold out | Kitchen dashboard "Mark Unavailable" updates `menu_items.available = 0`; refresh menu cache |

</details>

---

<div align="center">

Built by [Harish K](https://github.com/harryatwork) · Full-stack PHP engineer

</div>