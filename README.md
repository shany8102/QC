# QC Manager — Main App Overview

A desktop tool for radiology departments that automates patient-information extraction and problem comparison from the hospital PACS, then produces print-ready PDF reports.

Built with **Python + PySide6 (Qt)** and **Selenium**, with bilingual **English/Persian (EN/FA)** support and both **Jalali (Persian)** and **Gregorian** calendars.

---

## What the Main App Does

### 1. Startup Bootstrap (`main.py`)

On launch, the application prepares itself before any UI work:

* Loads and validates `config.yaml` (`hospital` and `sites` are required).
* Creates required output folders (`logs/`, `reports/`, `pdfs/`, etc.) on a fresh installation.
* Optionally clears the ChromeDriver cache.
* Detects the installed browser according to the configured priority:
  **Edge → Chrome → Firefox**
* Checks network connectivity.
* Checks for a newer version without blocking startup and shows the update dialog when available.

### 2. Activation Gate & Licensing

Before the main window becomes usable, the application verifies the **Ed25519-signed license** using the embedded **public key only**.

> The private signing key is never included in or distributed with the application.

With a valid activation:

* All application tabs become available.
* The daily scheduler can start.

Without activation:

* The application can still open.
* Application tabs remain locked.
* The daily scheduler does **not** start.

### 3. PACS Automation — Core Workflow

The main PACS automation is handled through `QCEngine` and `PACSBot` using Selenium.

Each workflow follows the same general pipeline:

```text
Login
  → Apply modality filter
  → Navigate to date
  → Re-apply filter
  → Extract all pages
  → Structure data
  → Generate PDF
  → Save state
  → Close browser
```

Date selection supports:

* **Today**
* **Yesterday**
* **Custom date range**

Both **Jalali** and **Gregorian** calendars are supported.

PACS credentials are protected using **Windows DPAPI** and are never stored as plaintext.

### 4. User Workflows — Controls Tab

| Workflow                | Trigger                                                               | Result                                   |
| ----------------------- | --------------------------------------------------------------------- | ---------------------------------------- |
| **Patient Information** | One button per modality                                               | `pdfs/..._patient_info_....pdf`          |
| **Comparison**          | Paste the problem list and click **Compare** for the desired modality | Comparison PDF with highlighted problems |

Supported modalities include:

**Graphy · CT · MRI · PET · SPECT · Mammography · Ultrasound**

Only modalities activated for the current site are displayed.

Inactive modalities are excluded from:

* Controls
* Reporting
* Scheduled jobs

### 5. Daily Scheduled Auto-Run

`SchedulerEngine` uses the **same engine instance and configuration used by the UI**.

Therefore, changes made through:

**Settings → Save**

are also used by scheduled executions.

The scheduled time is read from:

```yaml
schedule:
  run_time: "00:00"
```

Changing and saving the schedule restarts the scheduler.

Each scheduled run:

1. Executes all active modalities.
2. Skips the complete scheduled job if a manual run is already in progress.
3. Records the execution result in `state.json`.

Example:

```json
{
  "ok": 5,
  "failed": 1
}
```

The result is then available to the dashboard.

### 6. Multi-Site / Multi-Hospital Support

`HospitalManager` manages multiple sites through `config.yaml`.

When the active site is changed in the UI, its:

* PACS URL
* Credentials
* Enabled modalities
* Site-specific configuration

are switched accordingly.

Each center therefore sees only the modalities actually available at that site.

### 7. Dashboard, Logs & Settings

#### Dashboard

Provides:

* Browser information
* Network status
* Driver/cache information
* Quick-action cards
* Next scheduled run
* Last scheduled run
* Latest release notes

#### Logs

Provides structured, step-by-step logging streamed live into the UI, together with an execution summary.

#### Settings

Provides configuration for:

* Hospitals / sites
* Browser priority
* Scheduler
* Output paths
* Language (English / Persian)
* PACS credentials

Passwords entered through Settings are protected using **Windows DPAPI**.

### 8. Threading & UI Responsiveness

All heavy operations run inside a `WorkerThread` (`QThread`).

The worker communicates with the UI using:

```text
progress
finished
error
```

signals.

This prevents the UI from freezing while a PACS session or report-generation task is running.

---

## Quick Start

Install the required dependencies:

```bash
pip install -r requirements.txt
```

Start the application:

```bash
python main.py
```

Configure the PACS URL, username, password, and modalities through **Settings** or `config.yaml`.

For the complete configuration reference, see:

[`ed-apps/07-configuration.md`](ed-apps/07-configuration.md)

After configuration, enter the activation key once and use the **Controls** tab.

### Run the Offline Test Suite

```bash
python -m pytest tests -q
```

---

## Documentation

This README provides a high-level overview.

The canonical documentation is maintained under [`ed-apps/`](ed-apps/00-README.md).

| Topic                                     | Documentation                                                        |
| ----------------------------------------- | -------------------------------------------------------------------- |
| Main application architecture & workflows | [`ed-apps/02-main-app-guide.md`](ed-apps/02-main-app-guide.md)       |
| UI system, tabs & `WorkerThread`          | [`ed-apps/06-ui-system.md`](ed-apps/06-ui-system.md)                 |
| PACS Bot workflow & selectors             | [`ed-apps/05-pacs-bot-workflow.md`](ed-apps/05-pacs-bot-workflow.md) |
| Configuration reference                   | [`ed-apps/07-configuration.md`](ed-apps/07-configuration.md)         |
| Activation & licensing                    | [`ed-apps/04-activation-system.md`](ed-apps/04-activation-system.md) |
| Troubleshooting                           | [`ed-apps/10-troubleshooting.md`](ed-apps/10-troubleshooting.md)     |

---

<div dir="rtl">

# QC Manager — نمای کلی اپلیکیشن اصلی

ابزاری دسکتاپ برای بخش‌های رادیولوژی که استخراج اطلاعات بیمار و مقایسه مشکلات را به‌صورت خودکار از **PACS بیمارستان** انجام می‌دهد و در نهایت گزارش‌های **PDF آماده چاپ** تولید می‌کند.

این برنامه با **Python + PySide6 (Qt)** و **Selenium** ساخته شده و از رابط کاربری دوزبانه **فارسی / انگلیسی** و تقویم‌های **جلالی و میلادی** پشتیبانی می‌کند.

---

## کارهای اصلی اپلیکیشن

### ۱. راه‌اندازی اولیه (`main.py`)

هنگام اجرای برنامه، پیش از شروع عملیات رابط کاربری، مراحل زیر انجام می‌شود:

* بارگذاری و اعتبارسنجی `config.yaml`
* بررسی کلیدهای الزامی `hospital` و `sites`
* ایجاد پوشه‌های موردنیاز مانند `logs/`، `reports/` و `pdfs/` در نصب اولیه
* پاک‌سازی اختیاری کش ChromeDriver
* تشخیص مرورگر نصب‌شده بر اساس اولویت تنظیم‌شده:
  **Edge → Chrome → Firefox**
* بررسی اتصال شبکه
* بررسی وجود نسخه جدید بدون متوقف کردن روند شروع برنامه
* نمایش پنجره بروزرسانی در صورت وجود نسخه جدید

### ۲. دروازه فعال‌سازی و سیستم لایسنس

پیش از قابل استفاده شدن پنجره اصلی، اعتبار لایسنس امضاشده **Ed25519** با استفاده از **کلید عمومی** بررسی می‌شود.

> کلید خصوصی امضای لایسنس هرگز داخل برنامه قرار نمی‌گیرد و همراه آن توزیع نمی‌شود.

در صورت فعال بودن لایسنس:

* تمام تب‌های برنامه فعال می‌شوند.
* زمان‌بندی روزانه امکان اجرا پیدا می‌کند.

در صورت فعال نبودن:

* برنامه همچنان باز می‌شود.
* تب‌های برنامه قفل باقی می‌مانند.
* زمان‌بند روزانه **شروع نمی‌شود**.

### ۳. اتوماسیون PACS — هسته اصلی برنامه

عملیات اصلی PACS توسط `QCEngine` و `PACSBot` با استفاده از Selenium انجام می‌شود.

هر Workflow مسیر زیر را طی می‌کند:

```text
ورود
  → اعمال فیلتر مودالیتی
  → انتخاب تاریخ
  → اعمال مجدد فیلتر
  → استخراج تمام صفحات
  → ساختاربندی اطلاعات
  → تولید PDF
  → ذخیره وضعیت
  → بستن مرورگر
```

انتخاب تاریخ شامل موارد زیر است:

* **امروز**
* **دیروز**
* **بازه زمانی دلخواه**

هر دو تقویم **جلالی و میلادی** پشتیبانی می‌شوند.

اطلاعات ورود به PACS با استفاده از **Windows DPAPI** محافظت شده و به‌صورت متن ساده ذخیره نمی‌شوند.

### ۴. Workflowهای اصلی — تب Controls

| Workflow          | نحوه اجرا                                                        | خروجی                               |
| ----------------- | ---------------------------------------------------------------- | ----------------------------------- |
| **اطلاعات بیمار** | یک دکمه برای هر مودالیتی                                         | `pdfs/..._patient_info_....pdf`     |
| **مقایسه**        | قرار دادن لیست مشکلات و انتخاب **Compare** برای مودالیتی موردنظر | PDF مقایسه‌ای با مشکلات هایلایت‌شده |

مودالیتی‌های پشتیبانی‌شده:

**Graphy · CT · MRI · PET · SPECT · Mammography · Ultrasound**

فقط مودالیتی‌هایی که برای سایت فعال فعلی تعریف شده‌اند نمایش داده می‌شوند.

مودالیتی‌های غیرفعال در موارد زیر حضور ندارند:

* Controls
* گزارش‌گیری
* اجرای زمان‌بندی‌شده

### ۵. اجرای خودکار روزانه

`SchedulerEngine` از **همان موتور و تنظیماتی** استفاده می‌کند که رابط کاربری برنامه استفاده می‌کند.

بنابراین تغییراتی که از مسیر زیر ذخیره می‌شوند:

**Settings → Save**

در اجرای زمان‌بندی‌شده نیز اعمال خواهند شد.

زمان اجرا از مقدار زیر خوانده می‌شود:

```yaml
schedule:
  run_time: "00:00"
```

با تغییر ساعت و ذخیره تنظیمات، زمان‌بند مجدداً راه‌اندازی می‌شود.

در هر اجرای زمان‌بندی‌شده:

۱. تمام مودالیتی‌های فعال اجرا می‌شوند.

۲. اگر یک اجرای دستی در حال انجام باشد، کل اجرای زمان‌بندی‌شده رد می‌شود.

۳. نتیجه اجرا در `state.json` ذخیره می‌شود.

برای مثال:

```json
{
  "ok": 5,
  "failed": 1
}
```

این اطلاعات برای نمایش در داشبورد استفاده می‌شود.

### ۶. پشتیبانی از چند مرکز / چند بیمارستان

`HospitalManager` امکان مدیریت چند سایت را از طریق `config.yaml` فراهم می‌کند.

با تغییر سایت فعال در رابط کاربری، موارد زیر متناسب با مرکز انتخاب‌شده تغییر می‌کنند:

* آدرس PACS
* اطلاعات ورود
* مودالیتی‌های فعال
* تنظیمات اختصاصی سایت

بنابراین هر مرکز فقط مودالیتی‌هایی را مشاهده می‌کند که واقعاً در آن مرکز فعال هستند.

### ۷. داشبورد، لاگ‌ها و تنظیمات

#### داشبورد

شامل موارد زیر است:

* اطلاعات مرورگر
* وضعیت شبکه
* اطلاعات Cache / Driver
* کارت‌های دسترسی سریع
* زمان اجرای بعدی
* زمان آخرین اجرای زمان‌بندی‌شده
* آخرین Release Notes

#### لاگ‌ها

لاگ‌های ساختاریافته و مرحله‌به‌مرحله به‌صورت زنده در رابط کاربری نمایش داده می‌شوند و در پایان، خلاصه اجرای عملیات نیز ارائه می‌شود.

#### تنظیمات

امکان تنظیم موارد زیر را فراهم می‌کند:

* بیمارستان‌ها و سایت‌ها
* اولویت مرورگر
* زمان‌بندی
* مسیرهای خروجی
* زبان فارسی / انگلیسی
* اطلاعات ورود PACS

رمزهای واردشده در بخش Settings با **Windows DPAPI** محافظت می‌شوند.

### ۸. مدیریت Thread و روان بودن رابط کاربری

تمام عملیات سنگین داخل `WorkerThread` از نوع `QThread` اجرا می‌شوند.

ارتباط Worker با رابط کاربری از طریق Signalهای زیر انجام می‌شود:

```text
progress
finished
error
```

به این ترتیب هنگام اجرای نشست PACS یا تولید گزارش، رابط کاربری دچار هنگ یا Freeze نمی‌شود.

---

## شروع سریع

نصب وابستگی‌ها:

```bash
pip install -r requirements.txt
```

اجرای برنامه:

```bash
python main.py
```

آدرس PACS، نام کاربری، رمز عبور و مودالیتی‌ها را از طریق **Settings** یا فایل `config.yaml` تنظیم کنید.

مرجع کامل تنظیمات:

[`ed-apps/07-configuration.md`](ed-apps/07-configuration.md)

پس از انجام تنظیمات، کلید فعال‌سازی را یک بار وارد کرده و از تب **Controls** استفاده کنید.

### اجرای تست‌های آفلاین

```bash
python -m pytest tests -q
```

---

## مستندات

این README یک نمای کلی از برنامه ارائه می‌دهد.

مستندات اصلی و تخصصی هر بخش در پوشه [`ed-apps/`](ed-apps/00-README.md) قرار دارند.

| موضوع                              | مستندات                                                              |
| ---------------------------------- | -------------------------------------------------------------------- |
| معماری و Workflowهای اپلیکیشن اصلی | [`ed-apps/02-main-app-guide.md`](ed-apps/02-main-app-guide.md)       |
| سیستم UI، تب‌ها و `WorkerThread`   | [`ed-apps/06-ui-system.md`](ed-apps/06-ui-system.md)                 |
| جریان PACS Bot و Selectorها        | [`ed-apps/05-pacs-bot-workflow.md`](ed-apps/05-pacs-bot-workflow.md) |
| مرجع پیکربندی                      | [`ed-apps/07-configuration.md`](ed-apps/07-configuration.md)         |
| فعال‌سازی و سیستم لایسنس           | [`ed-apps/04-activation-system.md`](ed-apps/04-activation-system.md) |
| عیب‌یابی                           | [`ed-apps/10-troubleshooting.md`](ed-apps/10-troubleshooting.md)     |

</div>
