<div align="center">

<img src="assets/icons/appIcon1.png" alt="HD Dental Clinic Logo" width="100"/>

# HD Dental Clinic — Management System

**A full-featured, cross-platform dental clinic management application**  
built with Flutter, Supabase, and Firebase — works online *and* offline.

[![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B?logo=flutter)](https://flutter.dev)
[![Dart](https://img.shields.io/badge/Dart-3.x-0175C2?logo=dart)](https://dart.dev)
[![Supabase](https://img.shields.io/badge/Supabase-Backend-3ECF8E?logo=supabase)](https://supabase.com)
[![Firebase](https://img.shields.io/badge/Firebase-FCM-FFCA28?logo=firebase)](https://firebase.google.com)
[![Version](https://img.shields.io/badge/Version-1.0.4-blue)](https://github.com/a7med2002/hd-dental-releases)
[![Platform](https://img.shields.io/badge/Platform-Android%20%7C%20Windows-green)](https://github.com/a7med2002/hd-dental-releases)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Features](#-features)
- [Folder Structure](#-folder-structure)
- [How to Run](#-how-to-run)
- [Testing](#-testing)
- [Screenshots](#-screenshots)
- [Future Improvements](#-future-improvements)
- [Contact](#-contact)

---

## 🦷 Overview

**HD Dental Clinic** is a complete clinic management system designed specifically for dental practices. It handles everything from patient records and appointment scheduling to financial management, lab order tracking, and inventory control — all in one unified application.

The app is built with an **offline-first** approach: every action (add, edit, delete) works without an internet connection and automatically syncs to the cloud when connectivity is restored. It runs natively on both **Android mobile** and **Windows desktop** from a single codebase.

> **Current Version:** `1.0.4` — Released June 2026

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | Flutter (Dart) — cross-platform UI |
| **State Management** | Provider (`ChangeNotifier` + Singleton pattern) |
| **Backend / Database** | Supabase (PostgreSQL + RLS + Auth + Storage) |
| **Offline Cache** | Hive (key-value local storage) |
| **Push Notifications** | Firebase Cloud Messaging (FCM) — Android/iOS |
| **PDF Generation** | `pdf` + `printing` packages (Cairo font, Arabic RTL) |
| **Charts** | `fl_chart` |
| **Connectivity** | `connectivity_plus` (Windows-compatible) |
| **File Handling** | `file_picker` + `path_provider` + `url_launcher` |
| **OTA Updates** | `upgrader` package + Sparkle appcast XML |
| **Edge Functions** | Supabase Edge Functions (Deno / TypeScript) |
| **Scheduled Jobs** | Supabase pg_cron (appointment reminders every 30 min) |

---

## 🏗 Architecture

The project follows a clean, scalable **Repository → Provider → UI** layered architecture, fully consistent across all modules.

```
UI Screen
   └── Provider (ChangeNotifier Singleton)
         └── Repository
               ├── Supabase (online)    ← primary source of truth
               ├── Hive (offline cache) ← immediate local read/write
               └── SyncQueueService     ← queues mutations when offline
```

### Core Patterns

**Repository pattern** — every feature has its own repository that transparently handles online/offline state:

```dart
Future<List<T>> getAll() async {
  if (await connectivity.isOnline()) {
    try {
      final data = await supabase.from('table').select();
      await hive.clearAndSave(data);        // refresh cache
      return data;
    } catch (_) { return _fromCache(); }   // graceful fallback
  }
  return _fromCache();                      // offline: read local
}
```

**Offline Sync Queue** — mutations while offline are enqueued and replayed on reconnect:

```
Offline action → Hive (instant UI update) → SyncQueue
   ↓ (on reconnect)
Provider.load() → processQueue() → Supabase → refresh
```

**Singleton Providers** — every provider is a singleton registered in `main.dart`, eliminating duplicate state and redundant network calls across screens.

**Role-based Permissions** — stored in `users.role` on Supabase, cached in Hive for offline use, and enforced via `PermissionService` + `PermissionGuard` widget. No hardcoded emails — role changes take effect instantly without a new release.

---

## ✨ Features

### 🏠 Dashboard
- Today's revenue and session count
- Total patients and outstanding debts summary
- Weekly revenue bar chart (Mon → Sun)
- Top 5 patients with unpaid balances
- Last 5 payment sessions with automatic status (Paid / Partial / Overdue)
- Live alerts pulled from the notifications system
- Quick "New Appointment" button

### 👥 Patients
- Full patient registry with instant search (name / code / phone)
- Status filters: Active / Follow-up / Late / Archived
- Outstanding debt filter — highlights patients with unpaid balance in red
- Patient card displays chronic diseases badge (Healthy ✓ / DM / HT / etc.)
- Sorted by most recent activity (last session date)
- Pagination: 20 cards per page with "Load More"
- Pull-to-refresh on mobile
- Direct contact buttons: 📞 Call · 💬 WhatsApp · ✉️ SMS

### 🦷 Patient Detail — 5 Tabs

| Tab | Description |
|---|---|
| **Dental Chart** | Interactive tooth diagram (upper + lower jaw). Tap any tooth to add a treatment. Auto-creates a payment session on save. |
| **Medical Record** | Full treatment history, newest first. Supports R.C.T canal lengths (B/P/MB/MB2/ML/D/DB/DL), root letters (A–E) for Pulpectomy / Pulpotomy / Exo, prep notes. |
| **Files & Images** | Upload, view, and download patient files and X-rays. Full-screen image viewer with pinch-to-zoom (up to 6×). |
| **Notes** | Free-text notes with author name and timestamp. |
| **Financial Record** | Per-session breakdown: treatment, doctor, date, total / paid / remaining, payment account. Full payment history log. Edit doctor per session. |

**Additional patient actions:** Book appointment · Record payment · Export full PDF report

### 📅 Appointments
- Weekly calendar view with day-by-day list
- New appointment wizard: patient search → doctor → date & time → treatment & notes
- Status management: Confirmed / Waiting / Pending / Done / Cancelled
- Automatic reminders dispatched by Supabase Cron (every 30 minutes)

### 💰 Finance
- Weekly view (Saturday → Friday) with backward/forward week navigation
- Per-doctor session list with earned amount and weekly salary (commission-based)
- Expenses log with payment account tracking
- Outstanding debts section: pay any session directly from this screen
- Weekly revenue breakdown by payment account with stacked color bar chart
- Drilldown dialog per account: which patient paid, which treatment, which doctor
- Payroll: doctor commission configs + staff weekly salaries (independent of user accounts)
- PDF export: full weekly financial report

### 🔬 Labs
- Manage multiple dental labs with contact details
- Order tracking through 5 stages:
  `Pending → Sent → Received → Delivered → Paid`
- Stage auto-calculated from timestamps — no manual stage field needed
- Undo any stage (cascades and clears all forward stages)
- Edit lab info and patient name per order

### 📦 Inventory
- Track all clinic supplies with real-time quantities
- Optimistic increment / decrement (instant UI update, then syncs)
- Low-stock alert 🟡 (quantity ≤ custom threshold)
- Out-of-stock alert 🔴 (quantity = 0)
- Edit item name inline
- Push notifications triggered automatically when stock drops

### 📊 Reports
- Total revenue, total expenses, and net profit (all-time)
- Percentage change vs. previous 30-day period
- Bar chart: last 8 months (revenue + expenses side by side)
- Donut chart: treatment type distribution with case counts
- Payment account distribution — who received how much
- All revenue calculated from actual payment date (`paid_at`), not session creation date

### 🔔 Notifications
- **In-app:** notification bell with unread badge in sidebar and top bar
- **Mobile push (FCM):** delivered even when the app is closed or terminated
- Notification types: new appointment · status change · cancellation · reminder · payment received · lab update · low inventory · out of stock
- Deep links: tap any notification to navigate directly to the related screen
- Mark all as read · Delete all with confirmation dialog

### ⚙️ Settings
- Clinic info: name, phone, email, address, currency
- Payment accounts: add / edit / soft-delete bank accounts and e-wallets
- Staff management: view users, update display name and role
- Theme: Dark mode 🌙 / Light mode ☀️ (stored locally)

### 🔐 Role-based Permissions

| Role | Access Level |
|---|---|
| Owner / Admin | Full access: add, edit, delete |
| Doctor / Receptionist / Assistant | Add only — no edit or delete |

Role changes on Supabase take effect instantly with no app update required.

### 🔄 OTA Updates
- App checks for new versions at startup via Sparkle appcast XML
- Shows a non-intrusive upgrade dialog automatically
- Releases hosted on a public GitHub releases repository

---

## 📁 Folder Structure

```
lib/
├── core/
│   ├── auth/                   # Auth provider, permission service & guard
│   ├── config/                 # Supabase config
│   ├── notifications/          # FCM service + token management
│   ├── services/               # Connectivity, Hive, SyncQueue
│   └── utils/                  # Shared widgets (HoverActionButtons, AnimStaggerItem)
│
├── features/
│   ├── auth/                   # Login, Signup, Reset Password screens
│   ├── patients/               # Patient list + detail (5 tabs) + PDF export
│   ├── appointments/           # Weekly calendar + CRUD + dialogs
│   ├── finance/                # Weekly payroll, expenses, sessions, PDF export
│   ├── labs/                   # Lab management + order tracking
│   ├── inventory/              # Stock management
│   ├── dashboard/              # Stats cards, charts, alerts
│   ├── notifications/          # In-app notification list
│   ├── reports/                # Analytics charts and summaries
│   └── settings/               # Clinic info, accounts, staff, theme
│
├── shared/
│   └── layout/                 # AppShell, Sidebar, TopBar
│
├── providers/
│   └── theme_provider.dart
│
├── app.dart                    # Routes, page transitions, UpgradeAlert
├── firebase_options.dart       # Generated by FlutterFire CLI
└── main.dart                   # Bootstrap: Firebase, Providers, FCM init
```

---

## 🚀 How to Run

### Prerequisites

- Flutter SDK `^3.x` with Dart `^3.11`
- A Supabase project with the schema applied (16 tables + Edge Functions)
- Firebase project (Android/iOS only) with `google-services.json` placed in `android/app/`

### 1. Clone & install

```bash
git clone https://github.com/a7med2002/dental_management_app.git
cd dental_management_app
flutter pub get
```

### 2. Configure Supabase

Update `lib/core/config/supabase_config.dart` with your project credentials:

```dart
static const supabaseUrl = 'https://YOUR_PROJECT.supabase.co';
static const supabaseKey = 'YOUR_ANON_KEY';
```

### 3. Configure Firebase (mobile only)

```bash
dart pub global activate flutterfire_cli
flutterfire configure
```

Place the generated `google-services.json` in `android/app/`.

### 4. Run

```bash
# Android
flutter run --release

# Windows Desktop
flutter run -d windows --release
```

### 5. Build for distribution

```bash
# Android APK
flutter build apk --release

# Windows executable
flutter build windows --release
```

---

## 🧪 Testing

The project is validated through real-device and real-database integration testing:

- **Offline / online sync** — tested by toggling airplane mode mid-session; mutations enqueue and replay correctly on reconnect with no data loss
- **Role permissions** — verified by switching `users.role` in Supabase Table Editor and confirming that `PermissionGuard` responds without an app restart
- **Payment flow** — end-to-end: add treatment → auto-session creation (paid = 0) → record payment → verify outstanding balance updates in patient sidebar, finance screen, and dashboard
- **PDF export** — generated and visually verified on both Android and Windows; Arabic names render correctly with RTL alignment via Cairo font
- **Push notifications** — verified in all three FCM states: foreground banner, background system notification, terminated app tap-to-navigate

> Automated unit and widget tests are planned for a future milestone — see [Future Improvements](#-future-improvements).

---

## 📸 Screenshots

> Screenshots captured from production build v1.0.4.

<p align="center">
  <img src="assets/images/1.png" width="45%" />
  <img src="assets/images/2.png" width="45%" />
</p>

<p align="center">
  <img src="assets/images/3.png" width="45%" />
  <img src="assets/images/4.png" width="45%" />
</p>

<p align="center">
  <img src="assets/images/5.png" width="45%" />
  <img src="assets/images/6.png" width="45%" />
</p>

<p align="center">
  <img src="assets/images/7.png" width="45%" />
  <img src="assets/images/8.png" width="45%" />
</p>

<p align="center">
  <img src="assets/images/9.png" width="45%" />
  <img src="assets/images/10.png" width="45%" />
</p>

<p align="center">
  <img src="assets/images/11.png" width="45%" />
  <img src="assets/images/12.png" width="45%" />
</p>

---

## 🔮 Future Improvements

- [ ] **Unit & widget tests** — full coverage for repositories, providers, and key UI components
- [ ] **iOS support** — FCM is already configured; needs Apple Developer account + TestFlight setup
- [ ] **macOS / Linux desktop** — Flutter supports both; minor platform-specific adjustments needed
- [ ] **Multi-clinic / multi-branch** — schema is single-tenant; RLS-based tenant isolation is feasible
- [ ] **Patient portal** — lightweight read-only view for patients to access their own records and invoices
- [ ] **SMS / WhatsApp automation** — automated appointment reminders via Twilio or similar API
- [ ] **Advanced reporting** — custom date range picker, CSV export, per-doctor revenue breakdown
- [ ] **Biometric login** — fingerprint / face unlock on mobile
- [ ] **X-ray annotation** — draw and annotate directly on patient images inside the app

---

## 👨‍💻 Contact

Designed and built by:

**Engineer Ahmed Jamil Maqdad**

[![GitHub](https://img.shields.io/badge/GitHub-a7med2002-181717?logo=github&logoColor=white)](https://github.com/a7med2002)
[![Email](https://img.shields.io/badge/Email-ahmd2002mqdad@gmail.com-EA4335?logo=gmail&logoColor=white)](mailto:ahmd2002mqdad@gmail.com)

---

<div align="center">

*If you find this project useful, consider giving it a ⭐ on GitHub.*

</div>
