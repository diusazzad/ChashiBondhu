# 🗄️ ডাটাবেস ডিজাইন ও লজিক: চাষীবন্ধু (ChashiBondhu)

এই ডকুমেন্টে আমরা প্রজেক্টের "ডেটা ফাউন্ডেশন" নিয়ে কাজ করব। সিনিয়দের মতো আমরা সরাসরি কোড না করে প্রথমে **Raw SQL** এবং **ERD (Entity Relationship Diagram)** দিয়ে ডিজাইনের গভীর লজিকগুলো বুঝব।

---

## ১. এনটিটি রিলেশনশিপ ডায়াগ্রাম (ERD)

নিচের ডায়াগ্রামটি আমাদের টেবিলগুলোর মধ্যকার সম্পর্ক (Relationship) প্রকাশ করে:

```mermaid
erDiagram
    USERS ||--o{ SCANS : "performs"
    USERS ||--o{ ORDERS : "places"
    DEALERS ||--o{ MARKET_ITEMS : "owns"
    CROPS ||--o{ SCANS : "categorizes"
    MARKET_ITEMS ||--o{ ORDERS : "contains"

    USERS {
        bigint id PK
        string name
        string phone UNIQUE
        enum role "farmer, dealer, admin"
        string regional_dialect
    }

    SCANS {
        bigint id PK
        bigint user_id FK
        bigint crop_id FK
        string image_path
        string ai_result
        decimal confidence_score
        timestamp created_at
    }

    MARKET_ITEMS {
        bigint id PK
        bigint dealer_id FK
        string product_name
        decimal price
        integer stock_quantity
    }
```

---

## ২. Raw SQL Schema (The "Senior" Way)

আমরা এখানে **PostgreSQL** সিনট্যাক্স ব্যবহার করছি কারণ এটি জিও-লোকেশন এবং কমপ্লেক্স কুয়েরির জন্য অত্যন্ত শক্তিশালী।

### ক) ইউজার টেবিল (User Management)
এখানে `role` এনাম (Enum) ব্যবহার করা হয়েছে যাতে ডেটাবেস লেভেলেই সিকিউরিটি নিশ্চিত হয়।

```sql
CREATE TYPE user_role AS ENUM ('farmer', 'dealer', 'admin');

CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    phone VARCHAR(20) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role user_role DEFAULT 'farmer',
    regional_dialect VARCHAR(50), -- যেমন: 'noakhali', 'sylhet'
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### খ) এআই স্ক্যান টেবিল (AI Diagnosis)
এটি কৃষকদের প্রতিটি স্ক্যানের রেকর্ড রাখবে। লক্ষ্য করুন `user_id` এবং `crop_id` এর ওপর ফরেন কি (Foreign Key) ব্যবহার করা হয়েছে।

```sql
CREATE TABLE scans (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    crop_name VARCHAR(100), -- যেমন: ধান, আলু
    image_url TEXT NOT NULL,
    diagnosis_result TEXT, -- এআই যে রোগের নাম দিবে
    confidence DECIMAL(5, 2), -- এআই কতটা নিশ্চিত (০-১০০%)
    latitude DECIMAL(10, 8), -- কৃষকের লোকেশন (ম্যাপ করার জন্য)
    longitude DECIMAL(11, 8),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### গ) মার্কেটপ্লেস ও ইনভেন্টরি (Marketplace)
ডিলাররা তাদের স্টক এখানে মেনটেইন করবেন।

```sql
CREATE TABLE market_items (
    id BIGSERIAL PRIMARY KEY,
    dealer_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock_count INTEGER DEFAULT 0,
    is_available BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## ৩. রিলেশনশিপ কেন দরকার? (The Logic)

১. **One-to-Many (User to Scans):** একজন কৃষক অনেকগুলো ফসলের স্ক্যান করতে পারেন। তাই `users` এর সাথে `scans` এর সম্পর্ক ওয়ান-টু-মেনি।
২. **Foreign Key Constraints:** `ON DELETE CASCADE` ব্যবহার করা হয়েছে যাতে কোনো ইউজার ডিলিট হয়ে গেলে তার সাথে সম্পর্কিত স্ক্যান হিস্ট্রিও ডিলিট হয়ে যায় (ডেটা ক্লিনিং)।

---

## ৪. পরবর্তী চ্যালেঞ্জ

আমরা কি এই Raw SQL গুলো দিয়ে একটি **SQLite** বা **Postgres** টেস্ট ডাটাবেস সেটআপ করে রিয়েল ডাটা ইনসার্ট করে দেখব? এতে করে ল্যারাভেল মাইগ্রেশনে যাওয়ার আগে আমাদের লজিকগুলো ভ্যালিডেট হয়ে যাবে।

> [!TIP]
> **টিম লিড টিপস:** আপনি যখন জুনিয়রদের এই স্কিমাটা দেখাবেন, তাদের জিজ্ঞেস করবেন—"যদি আমরা কৃষকের মাটির স্বাস্থ্য (Soil Health) যোগ করতে চাই, তবে কোন টেবিলে ফিল্ড বাড়াবো নাকি নতুন টেবিল বানাবো?" (সঠিক উত্তর: নতুন টেবিল `soil_reports`)।
