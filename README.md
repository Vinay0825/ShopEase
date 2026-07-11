<div align="center">

<img src="https://img.shields.io/badge/Platform-Android-3DDC84?style=for-the-badge&logo=android&logoColor=white"/>
<img src="https://img.shields.io/badge/Language-Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white"/>
<img src="https://img.shields.io/badge/Firebase-Firestore%20%2B%20Auth-FFCA28?style=for-the-badge&logo=firebase&logoColor=black"/>
<img src="https://img.shields.io/badge/Architecture-MVVM-6200EE?style=for-the-badge"/>
<img src="https://img.shields.io/badge/License-MIT-blue?style=for-the-badge"/>

<br/><br/>

# 🛒 ShopEase
### Retail Analytics & Inventory Management for Indian Kirana Shops

**ShopEase** is a full-featured Android app that brings modern inventory management and retail analytics to small Indian kirana and retail shop owners — built entirely in Java as a college project.

[📲 Features](#-features) · [🏗️ Architecture](#️-architecture) · [🗄️ Firebase Schema](#️-firebase-schema) · [🚀 Getting Started](#-getting-started) · [📸 Screenshots](#-screenshots)

</div>

---

## ✨ Features

### 📦 Product Management
- Add, edit, and view products with full details (name, category, price, stock, barcode)
- **Barcode scanning** via CameraX + ML Kit — point and scan instantly
- **Barcode generation** using ZXing for printed shelf labels
- Product images stored as Base64 JPEG directly in Firestore

### 🛍️ Selling Flow
- Fast, intuitive point-of-sale interface
- **Offline support** with optimistic UI — sales work even without internet
- Stock decrements in real time with Firestore transactions

### 🔄 Restocking Flow
- Log incoming stock quickly with barcode scan or manual search
- **Offline-first** — restock records sync when connectivity is restored

### 📋 History & Filters
- View full sales and restock history
- Filter by **Today / This Week / This Month / All Time**
- Clean timeline UI with transaction details

### 📊 Analytics Dashboard
- 🏆 **Best Sellers** — top-performing products by revenue and quantity
- 🐢 **Slow Movers** — identify dead stock before it becomes a problem
- 🗂️ **Category Breakdown** — see which categories drive the most revenue
- 📈 **Line Chart** — sales trend over time

### 🔔 Alerts & Reminders
- **Low stock alerts** — get notified when products fall below threshold
- **Daily reminders** via WorkManager for end-of-day stock checks

### 🎨 Customization
- **Theme switching** — light and dark mode with persistent preference
- **Shop settings** — configure shop name and shop type stored in Firebase

### 📤 Excel Export
- Export sales and restock records to `.xlsx` using Apache POI
- Ready to share with accountants or for record-keeping

---

## 🏗️ Architecture

ShopEase follows a clean **MVVM** architecture with a unidirectional data flow:

```
Fragment (UI)
    │
    ▼
ViewModel  ←────────────── LiveData / StateFlow
    │
    ▼
Repository
    │
    ├──▶ Firebase Firestore  (remote source of truth)
    └──▶ Local State         (optimistic UI for offline support)
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
| **Barcode Scanning** | CameraX + ML Kit Barcode Scanning |
| **Barcode Generation** | ZXing |
| **Image Storage** | Base64 JPEG in Firestore (no Firebase Storage) |
| **Excel Export** | Apache POI |
| **Background Tasks** | WorkManager |
| **Min SDK** | 24 (Android 7.0) |

---

## 📁 Project Structure

```
ShopEase/
├── app/
│   └── src/main/
│       ├── java/com/yourname/shopease/
│       │   ├── data/
│       │   │   ├── model/          # Product, Sale, Restock, Analytics POJOs
│       │   │   └── repository/     # FirestoreRepository, AuthRepository
│       │   ├── ui/
│       │   │   ├── auth/           # Login, Register fragments
│       │   │   ├── sell/           # SellFragment + SellViewModel
│       │   │   ├── restock/        # RestockFragment + RestockViewModel
│       │   │   ├── history/        # SalesHistoryFragment, RestockHistoryFragment
│       │   │   ├── analytics/      # AnalyticsFragment + AnalyticsViewModel
│       │   │   ├── products/       # ProductListFragment, AddEditProductFragment
│       │   │   └── settings/       # ShopSettingsFragment, ThemeSettingsFragment
│       │   ├── workers/            # WorkManager Workers (LowStockWorker, etc.)
│       │   └── utils/              # BarcodeUtils, ExcelExporter, ImageUtils
│       └── res/
│           ├── layout/             # XML layouts for all fragments & items
│           ├── navigation/         # nav_graph.xml
│           ├── values/             # themes, colors, strings
│           └── drawable/
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
        │   └── {productId}           # { name, category, price, stock,
        │                             #   barcode, imageBase64, lowStockThreshold }
        ├── sales/
        │   └── {saleId}              # { productId, productName, qty, totalPrice,
        │                             #   category, timestamp }
        ├── restocks/
        │   └── {restockId}           # { productId, productName, qty, costPrice,
        │                             #   timestamp }
        └── analytics/
            └── {dateKey}             # { totalRevenue, totalSales, topProducts,
                                      #   categoryBreakdown }
```

> **Note:** Product images are stored as Base64-encoded JPEG strings directly in the `products` collection. Firebase Storage is intentionally **not** used.

---

## 🚀 Getting Started

### Prerequisites

- Android Studio **Hedgehog** or later
- JDK 17+
- A Firebase project with **Firestore** and **Authentication** enabled
- A physical Android device or emulator (API 24+)

### 1. Clone the Repository

```bash
git clone https://github.com/Vinay0825/ShopEase.git
cd ShopEase
```

### 2. Set Up Firebase

1. Go to the [Firebase Console](https://console.firebase.google.com/) and create a new project.
2. Add an **Android app** with your package name (e.g., `com.yourname.shopease`).
3. Download the `google-services.json` file.
4. Place it in the `app/` directory:
   ```
   ShopEase/app/google-services.json
   ```
5. In Firebase Console, enable:
   - **Authentication** → Email/Password sign-in
   - **Cloud Firestore** → Start in test mode (or apply the rules below)

### 3. Firestore Security Rules (Recommended)

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

Open the project in **Android Studio**, let Gradle sync, then run on your device or emulator.

```bash
./gradlew assembleDebug
```

> ⚠️ Barcode scanning requires a **physical device** with a rear camera. CameraX does not work on most emulators.

---

## 📸 Screenshots

> _Screenshots coming soon. Stay tuned!_

| Home / Products | Sell Flow | Analytics |
|:-:|:-:|:-:|
| `placeholder` | `placeholder` | `placeholder` |

| Restock | History | Settings |
|:-:|:-:|:-:|
| `placeholder` | `placeholder` | `placeholder` |

---

## 🤝 Contributing

Contributions are welcome! If you'd like to improve ShopEase:

1. **Fork** the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m 'Add some feature'`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a **Pull Request**

Please ensure your code follows the existing MVVM structure and naming conventions.

---

## 📄 License

```
MIT License

Copyright (c) 2025 Vinay

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

<div align="center">


⭐ **Star this repo if you found it useful!**

</div>
