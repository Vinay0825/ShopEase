<div align="center">

<img src="https://img.shields.io/badge/Platform-Android-3DDC84?style=for-the-badge&logo=android&logoColor=white"/>
<img src="https://img.shields.io/badge/Language-Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white"/>
<img src="https://img.shields.io/badge/Firebase-Firestore%20%2B%20Auth-FFCA28?style=for-the-badge&logo=firebase&logoColor=black"/>
<img src="https://img.shields.io/badge/Architecture-MVVM-6200EE?style=for-the-badge"/>
<img src="https://img.shields.io/badge/License-MIT-blue?style=for-the-badge"/>

<br/><br/>

# 🛒 ShopEase
### Inventory & Retail Analytics, Built for the Indian Kirana Shop

Most inventory apps assume every product comes in a barcoded package. Real Indian retail doesn't work that way — a huge share of daily sales is loose rice, grain, sugar, and other commodities sold by weight, with no barcode to scan.

ShopEase handles both: packaged products via standard barcode scanning, and **loose/unpackaged items via shop-generated barcodes with decimal-quantity tracking** — so a shopkeeper can run their entire inventory from one phone.

🏆 **1st Prize, LTCE Mumbai Mini Project Competition (2024)**

[📲 Features](#-features) · [🏗️ Architecture](#️-architecture) · [🗄️ Firebase Schema](#️-firebase-schema) · [🚀 Getting Started](#-getting-started) · [📸 Screenshots](#-screenshots)

</div>

---

## ✨ Features

### 📦 Product & Loose Item Management
- Add, edit, and view products with full details (name, category, price, stock, barcode)
- **Loose item support** — shopkeepers generate their own barcodes for unpackaged goods (grain, rice, sugar, pulses) and track stock in decimal weight, not just whole units
- Barcode scanning via CameraX + ML Kit — instant, on-device
- Barcode generation (ZXing) for both packaged products and shop-created loose item labels
- Product images stored as compressed Base64 — no separate storage service needed

### 🛍️ Sell & Restock
- Fast point-of-sale flow for both packaged and loose items
- **Offline-first** — sales and restocks queue locally and sync automatically when connectivity returns
- Stock updates in real time across staff devices
- Void incorrect sales or restocks directly from history

### 📋 History & Filters
- Full sales and restock history with Today / This Week / This Month / All Time filters
- Clean, scannable transaction timeline

### 📊 Analytics Dashboard
- 🏆 **Best Sellers** — top-performing products by revenue and quantity
- 🐢 **Slow Movers** — surfaces dead stock before it ties up capital
- 🗂️ **Category Breakdown** — revenue by category
- 📈 **Trend Chart** — sales over time, filterable by range
- Insights exportable straight to Excel

### 🔔 Alerts & Reminders
- Low-stock alerts, tuned per product
- Daily end-of-day stock check reminders

### ⚙️ Account & Settings
- Change password or log out from a dedicated settings screen
- Light/dark theme with persistent preference
- Configurable shop profile (name, type)

### 📤 Excel Export
- Sales and restock records export to `.xlsx` (Apache POI) — a format shop owners and accountants already trust

---

## 🏗️ Architecture

MVVM with a unidirectional data flow, offline-first by design:

```
Fragment (UI)
    │
    ▼
ViewModel  ←────────────── LiveData
    │
    ▼
Repository
    │
    ├──▶ Firebase Firestore  (remote source of truth)
    └──▶ Local optimistic state (works offline, syncs when online)
```

### 🧱 Tech Stack

| Layer | Technology |
|---|---|
| **Language** | Java |
| **UI** | XML Layouts + ViewBinding |
| **Navigation** | Android Navigation Component |
| **Architecture** | MVVM (Fragment → ViewModel → Repository) |
| **Database** | Firebase Firestore |
| **Authentication** | Firebase Auth |
| **Barcode Scanning** | CameraX + ML Kit |
| **Barcode Generation** | ZXing (CODE_128) |
| **Image Storage** | Compressed Base64 in Firestore |
| **Excel Export** | Apache POI |
| **Background Tasks** | WorkManager |
| **Min SDK** | 24 (Android 7.0) |

---

## 📁 Project Structure

```
ShopEase/
├── app/
│   └── src/main/
│       ├── java/com/example/inventory/
│       │   ├── model/          # Product, Sale, Restock, Analytics
│       │   ├── repository/     # ProductRepository, SalesRepository, AnalyticsRepository
│       │   ├── viewmodel/      # ProductViewModel, SalesViewModel, DashboardViewModel
│       │   ├── ui/
│       │   │   ├── auth/           # Login, Register, Splash
│       │   │   ├── dashboard/      # DashboardFragment
│       │   │   ├── product/        # Product list/detail/add/edit + loose item entry
│       │   │   ├── sales/          # Sell, Restock, Sales & Restock history
│       │   │   ├── analytics/      # AnalyticsFragment
│       │   │   ├── alerts/         # Low stock alerts
│       │   │   ├── settings/       # Account & app settings
│       │   │   └── scanner/        # Barcode scanning
│       │   ├── notifications/  # Low stock + daily reminder workers
│       │   └── utils/          # Category, date, stock-rules helpers
│       └── res/
│           ├── layout/ navigation/ values/ drawable/
├── google-services.json            # ⚠️ NOT committed — add your own
└── build.gradle
```

---

## 🗄️ Firebase Schema

```
Firestore
└── users/
    └── {userId}/
        ├── settings/
        │   └── shopInfo              # { shopName, shopType, theme }
        ├── products/
        │   └── {barcode}             # { name, category, price, stock,
        │                             #   unit, lowStockThreshold, image }
        ├── sales/
        │   └── {saleId}              # { barcode, productName, quantity,
        │                             #   totalAmount, timestamp, voided }
        ├── restocks/
        │   └── {restockId}           # { barcode, productName, quantity,
        │                             #   timestamp, voided }
        └── analytics/
            └── {barcode}             # { totalUnitsSold, totalRevenue,
                                      #   salesCount, dailySales }
```

> Product images are stored as compressed Base64 strings directly in Firestore — no separate storage service required.

---

## 🚀 Getting Started

### Prerequisites
- Android Studio Hedgehog or later
- JDK 17+
- A Firebase project with Firestore and Authentication enabled
- A physical Android device (barcode scanning needs a real rear camera — most emulators won't work)

### 1. Clone the Repository
```bash
git clone https://github.com/Vinay0825/ShopEase.git
cd ShopEase
```

### 2. Set Up Firebase
1. Create a project in the [Firebase Console](https://console.firebase.google.com/), add an Android app with package name `com.example.inventory`.
2. Download `google-services.json` and place it in `app/`.
3. Enable **Authentication (Email/Password)** and **Cloud Firestore**.

### 3. Firestore Security Rules
```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

### 4. Build & Run
```bash
./gradlew assembleDebug
```
> ⚠️ Barcode scanning requires a physical device with a rear camera.

---

## 📸 Screenshots

_Coming soon._

| Home / Products | Sell Flow | Analytics |
|:-:|:-:|:-:|
| `placeholder` | `placeholder` | `placeholder` |

| Restock | History | Settings |
|:-:|:-:|:-:|
| `placeholder` | `placeholder` | `placeholder` |

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes
4. Open a Pull Request, keeping the existing MVVM structure and conventions

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

<div align="center">

⭐ **Star this repo if you found it useful!**

</div>
