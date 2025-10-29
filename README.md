# PhE Community Bot & Admin Panel

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](https://github.com/melvud/the-bot)
[![Platform](https://img.shields.io/badge/platform-Telegram%20%7C%20Web-blue)](https://telegram.org/)
[![Python](https://img.shields.io/badge/python-3.12-blueviolet.svg)](https://www.python.org/)
[![Tech](https://img.shields.io/badge/tech-Aiogram%20%7C%20AIOHTTP%20%7C%20PostgreSQL-orange.svg)](https://aiogram.dev/)

A comprehensive community management platform built on **Python** and **asyncio**. This project consists of a high-performance **Telegram Bot** (`aiogram`) for user interaction and a secure **REST API** (`aiohttp`) that powers a built-in **Telegram Mini App (TMA)** for administration.

This system is designed to manage a full-featured community program, including user onboarding, event management, mentorship matching, "Random Coffee" networking, and multi-channel notifications via Telegram and email.

---

## ✨ Core Features

This is a complete, three-part system: a user-facing bot, a powerful admin panel, and a robust backend.

### 1. Telegram Bot (User-Facing)

* **FSM Onboarding:** A guided, multi-step Finite State Machine (FSM) for new user registration (name, email, professional segment, affiliation, etc.).
* **User Approval System:** New user registrations are placed in a `pending` state for admin approval via the WebApp, preventing spam.
* **User Dashboard:** A rich `ReplyKeyboardMarkup` menu for users to access all features.
* **Profile Management:** Users can view (`/profile`) and edit their information (name, email, "about me") at any time.
* **Feature Subscriptions:** Users can individually opt-in or opt-out of:
    * **☕ Random Coffee:** Weekly or monthly 1:1 networking matches.
    * **🎓 Mentorship:** Register as a Mentor or a Mentee.
    * **💥 Socials:** Receive notifications for informal members-only events.
* **Notification Preferences:** Granular control over which announcements, events, and match notifications they receive.
* **Multi-Channel Comms:** Users can choose to receive notifications via "Email Only", "Telegram Only", or "Email + Telegram".
* **GDPR Deletion:** A `/gdpr_delete` flow that allows users to permanently erase all their data from the database.

### 2. Admin Panel (Telegram Mini App)

* **Secure Admin Dashboard:** A full-featured admin panel built as a Telegram Mini App (`webapp.html`), accessible only to admins directly from the bot's menu.
* **TMA Authentication:** All API requests from the WebApp are securely authenticated on the backend using Telegram's `initData` hashing verification.
* **Dashboard:** A real-time stats overview (`/api/stats`) showing total users, subscribers, pending approvals, and more.
* **User Approval Queue:** Admins can view all pending users and approve or reject them with a single tap.
* **Scheduler Control:** View and update the "Random Coffee" matching schedule (days of week, time of day) and trigger a manual run (with cooldown protection).
* **Event Management:** Full CRUD (Create, Read, Update, Delete) for official events and socials. Includes image upload for event covers.
* **Broadcast System:**
    * Broadcast new events to all eligible users with one click.
    * Send custom Markdown-supported messages to all users (or apply filters).
* **Mentorship Matching:** A dedicated UI to view all available mentors and unmatched mentees, and assign them to each other.
* **Database-Driven Email Templates:** A built-in modal editor (`/api/email-templates`) allowing admins to view, edit, and preview the HTML and text email templates used for all automated communications.

### 3. Backend & Core Logic

* **Asynchronous API:** A high-performance `aiohttp` web server handles all API requests and serves the WebApp.
* **Smart Matching (`matcher.py`):**
    * The "Random Coffee" algorithm fetches all eligible, subscribed candidates.
    * It intelligently pairs users, respecting their preferences and avoiding recent matches by checking a `LOOKBACK_WEEKS` window in the `pairings` table.
* **Multi-Channel Notification Engine:** The matching and assignment logic automatically respects each user's `communication_mode` preference, sending notifications via:
    * **Telegram:** A direct message from the bot (`_bot.send_message`).
    * **Email:** A formatted HTML email (`email_sender.py`).
* **Database-Driven Emails:** The email system fetches templates (e.g., `random_coffee`, `mentorship_match`) from the PostgreSQL database, renders them with `{{variables}}`, and sends them asynchronously using `smtplib` in a `ThreadPoolExecutor`.
* **Persistent Storage:** All data (users, events, pairings, mentorships, run logs, email templates) is stored in a **PostgreSQL** database (`schema.sql`).

---

## 🛠️ Technology Stack & Architecture

This project is built with a modern, scalable, and fully asynchronous Python stack.

* **Bot Framework:** **Aiogram 3.x**
* **Web Server & API:** **AIOHTTP** & `aiohttp-cors`
* **Database:** **PostgreSQL** (with `asyncpg` driver)
* **Asynchronous:** Fully built on **asyncio** for high throughput.
* **State Management:** **Aiogram FSM** (Finite State Machine) for user flows.
* **Admin Frontend:** **Telegram Mini App** (HTML, CSS, vanilla JavaScript)
* **Email:** `smtplib` & `ThreadPoolExecutor`
* **Deployment:** **Docker** & `docker-compose.yml`

---

## 🧠 How It Works: Admin WebApp Authentication

The most critical part of this project is the secure connection between the Telegram Mini App (WebApp) and the backend API.

1.  A user sends a command (e.g., `/start`) to the bot.
2.  The `MenuButtonMiddleware` intercepts the update. It checks if the `user.id` is in the `ADMIN_IDS` list.
3.  If it is, the bot calls `bot.set_chat_menu_button` to set the user's menu button to a `MenuButtonWebApp` type, pointing to the `WEBAPP_URL`. (Regular users see a `MenuButtonDefault`).
4.  The admin clicks this button, opening `webapp.html` inside their Telegram client.
5.  The JavaScript in `webapp.html` immediately accesses `Telegram.WebApp.initData`.
6.  When the WebApp makes a `fetch` request (e.g., to `/api/stats`), it includes this data in the `Authorization: tma <initData>` header.
7.  The `aiohttp` server's `auth_middleware` (`api.py`) catches this request.
8.  It calls `verify_telegram_web_app`, which validates the `hash` from `initData` against a hash generated using the `BOT_TOKEN`. This proves the request is from a *legitimate* Telegram user.
9.  The middleware then extracts the `user_id` from the verified data and checks if it's in the `_admin_ids` set.
10. Only if **both** checks pass is the request forwarded to the API handler (e.g., `get_stats`). This provides robust, session-free authentication for the admin panel.

---

## 👨‍💼 Looking for a Developer?

Hi! I'm the developer behind this project. I specialize in building high-quality, performant, and reliable backend systems and bots using Python, `asyncio`, and modern database technologies.

If you're impressed by the complexity and quality of this app—from its multi-channel notification system to its secure web-based admin panel—I'm confident I can bring the same level of expertise to your project.

* **Email:** `ivsilan2005@gmail.com`

Let's build something great together.
