# 📡 API স্পেসিফিকেশন: চাষীবন্ধু (ChashiBondhu)

এই ডকুমেন্টটি ল্যারাভেল ব্যাকএন্ড এবং ফ্লাটার মোবাইলের মধ্যে ডেটা বিনিময়ের নিয়মাবলী বর্ণনা করে। সিনিয়রা এপিআই ডিজাইনের সময় **RESTful** স্ট্যান্ডার্ড এবং **JSON** ফরমেট ফলো করেন।

---

## ১. জেনারেল এপিআই স্টাইল (Standard Response)

প্রতিটি এপিআই নিচের ফরম্যাটে ডেটা রিটার্ন করবে:

```json
{
    "success": true,
    "message": "অপারেশন সফল হয়েছে",
    "data": { ... },
    "errors": null
}
```

---

## ২. অথেন্টিকেশন (Authentication Flow)

আমরা **Laravel Sanctum** ব্যবহার করব টোকেন ভিত্তিক অথেন্টিকেশনের জন্য।

*   **Endpoint:** `POST /api/v1/login`
*   **Request:** `phone`, `password`
*   **Response:** `access_token`, `user_info`

---

## ৩. গুরুত্বপূর্ণ এন্ডপয়েন্ট সমূহ (Key Endpoints)

### ক) এআই স্ক্যান সাবমিশন (Crop Diagnosis)
*   **Endpoint:** `POST /api/v1/scans`
*   **Method:** Multipart/Form-Data
*   **Payload:** `image` (file), `crop_type` (string)
*   **Logic:** এই এন্ডপয়েন্টে ছবি এলে ল্যারাভেল তা ইন্টারনালি **FastAPI** সার্ভারে পাঠাবে এবং রেজাল্ট ইউজারকে ফেরত দিবে।

### খ) মার্কেটপ্লেস ব্রাউজিং (Product List)
*   **Endpoint:** `GET /api/v1/market/items`
*   **Query Params:** `latitude`, `longitude` (কাছের ডিলার খোঁজার জন্য)
*   **Logic:** এটি ইউজারের লোকেশন অনুযায়ী সর্ট করে ডিলারদের প্রোডাক্ট দেখাবে।

---

## ৪. এরর হ্যান্ডলিং (Error Codes)

*   `401 Unauthorized`: টোকেন এক্সপায়ার হয়ে গেলে।
*   `422 Unprocessable Entity`: ইনপুট ভ্যালিডেশন ফেল করলে।
*   `500 Server Error`: ইন্টারনাল কোনো কারিগরি ত্রুটি।

---

> [!TIP]
> **টিম লিড টিপস:** আপনি যখন ফ্লাটার ডেভেলপারদের সাথে কথা বলবেন, তখন তাদের এই `success` এবং `data` ফরমেটটি আগে বুঝিয়ে দিবেন। তাহলে তাদের অ্যাপে ডাটা রেন্ডার করা অনেক সহজ হবে।
