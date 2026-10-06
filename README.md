# QC Manager — Main App Overview

A desktop application for radiology departments that automates patient-information extraction and problem comparison from hospital PACS and generates print-ready PDF reports.

Built with **Python, PySide6 (Qt), and Selenium**, with **English/Persian (EN/FA)** support and **Jalali/Gregorian** calendars.

## Main Features

* **PACS Automation** — Automated login, modality filtering, date selection, data extraction, and PDF generation.
* **Patient Information** — Extracts patient information for active modalities.
* **Problem Comparison** — Compares PACS data with a provided problem list and highlights matches.
* **Multi-Site Support** — Supports multiple hospitals/sites with independent URLs, credentials, and modalities.
* **Daily Scheduler** — Automatically runs active modalities according to the configured schedule.
* **Licensing** — Ed25519-signed activation with public-key verification.
* **Security** — PACS credentials are protected using Windows DPAPI.
* **Dashboard & Logs** — System status, scheduled jobs, live logs, and execution summaries.
* **Responsive UI** — Heavy operations run in `QThread` workers to keep the interface responsive.
* **Automatic Updates** — Non-blocking version checking and update notifications.

## Activation & Licensing

The application requires a valid license to unlock its full functionality.

For **activation, licensing, or support**, please contact the developer.


---

<div dir="rtl">

# QC Manager — نمای کلی اپلیکیشن

اپلیکیشن دسکتاپ تخصصی برای بخش‌های رادیولوژی که استخراج اطلاعات بیمار و مقایسه مشکلات را از **PACS بیمارستان** خودکار کرده و گزارش‌های **PDF آماده چاپ** تولید می‌کند.

ساخته‌شده با **Python، PySide6 و Selenium** با پشتیبانی از **فارسی/انگلیسی** و تقویم‌های **جلالی/میلادی**.

## امکانات اصلی

* **اتوماسیون PACS** — ورود، فیلتر مودالیتی، انتخاب تاریخ، استخراج اطلاعات و تولید PDF
* **اطلاعات بیمار** — استخراج اطلاعات برای مودالیتی‌های فعال
* **مقایسه مشکلات** — مقایسه اطلاعات PACS با لیست مشکلات و هایلایت موارد مرتبط
* **پشتیبانی چند مرکز** — مدیریت چند بیمارستان و سایت با تنظیمات مستقل
* **زمان‌بندی روزانه** — اجرای خودکار مودالیتی‌های فعال
* **سیستم لایسنس** — فعال‌سازی با امضای Ed25519
* **امنیت** — محافظت از اطلاعات ورود PACS با Windows DPAPI
* **داشبورد و لاگ‌ها** — نمایش وضعیت سیستم و گزارش اجرای عملیات
* **رابط کاربری روان** — اجرای عملیات سنگین در Worker Thread
* **بروزرسانی خودکار** — بررسی نسخه جدید و نمایش اعلان بروزرسانی

فعال‌سازی و لایسنس

برای استفاده از تمامی قابلیت‌های برنامه، به یک لایسنس معتبر نیاز است.

برای فعال‌سازی، دریافت لایسنس یا پشتیبانی، لطفاً با توسعه‌دهنده در ارتباط باشید.

</div>
