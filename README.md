# 🛡️ iTop Wrapper — سامانه مدیریت خدمات فناوری اطلاعات

پوشش‌دهنده حرفه‌ای موتور iTop با معماری MVC، توسعه‌یافته با **Spring Boot 3** + **Java 17** + **PostgreSQL**

---

## 📋 ویژگی‌های اصلی

| ماژول | توضیح |
|---|---|
| 🪟 پنجره واحد خدمات | کاتالوگ یکپارچه خدمات فناوری اطلاعات |
| 💰 موتور قیمت‌گذاری | مدل‌های FIXED، HOURLY، MONTHLY، PER_USER، TIERED، FORMULA |
| 👛 **کیف‌پول مشتری** | کسر خودکار اعتبار هنگام انتخاب خدمت (فوری یا پس از تأیید) + شارژ/کسر دستی توسط ادمین |
| 🔐 احراز هویت | JWT + Local + LDAP + Multi-LDAP |
| 📊 SLA/OLA | تعریف، پایش، و هشدار نقض SLA |
| 📁 مدیریت پروژه | گانت چارت، وظایف، پیشرفت |
| 💵 مرکز هزینه | بودجه‌بندی سلسله‌مراتبی |
| 🧩 ماژول‌پذیری | آپلود و نصب ماژول JAR سفارشی |
| 📝 لاگ جامع | ذخیره تمام رویدادها در PostgreSQL |
| 🔄 همگام‌سازی | همگام‌سازی خودکار دوره‌ای با iTop (Incident/Change/Problem/Project/CostCenter/CMDB) |

---

## 👛 سیستم کیف‌پول

هر مشتری دقیقاً یک کیف‌پول دارد که با اولین استفاده به‌صورت خودکار ساخته می‌شود.

**نحوه کسر اعتبار** (قابل تنظیم در سطح هر خدمت، از پنل مدیریت کاتالوگ):
- **فوری (IMMEDIATE):** هم‌زمان با ثبت درخواست، مبلغ بلافاصله از کیف‌پول کسر می‌شود.
- **پس از تأیید (ON_APPROVAL):** مبلغ ابتدا «مسدود» (Hold) می‌شود؛ با تأیید/رفع درخواست توسط اپراتور به‌صورت قطعی کسر (Capture) و با رد/لغو درخواست به‌طور خودکار آزاد (Release) می‌شود.

**مدیریت اعتبار توسط ادمین** (`/admin/wallets`):
- شارژ و کسر دستی موجودی با ثبت دلیل
- تعیین **سقف اعتباری** برای هر مشتری (امکان رفتن موجودی تا حدی منفی)
- مشاهده گردش‌حساب کامل (Immutable Ledger) با موجودی پیش/پس از هر تراکنش
- هشدار خودکار کیف‌پول‌های با موجودی کم

تمام عملیات تغییر موجودی با **قفل بدبینانه ردیف (Pessimistic Lock)** انجام می‌شود تا در صورت درخواست‌های همزمان، هیچ Race Condition رخ ندهد.

---

## 🏗️ معماری

```
┌─────────────────────────────────────────────────────────┐
│                    NGINX (Reverse Proxy)                 │
├──────────────────────┬──────────────────────────────────┤
│   پرتال مدیریتی      │       پرتال مشتریان              │
│   /admin/**          │       /portal/**                 │
├──────────────────────┴──────────────────────────────────┤
│              Spring Boot Application (8080)              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │
│  │Controller│ │ Service  │ │Repository│ │ Security │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘   │
├────────────────────────┬────────────────────────────────┤
│    PostgreSQL (5432)   │      iTop Engine (8081)        │
└────────────────────────┴────────────────────────────────┘
```

---

## 🚀 راه‌اندازی سریع

### پیش‌نیازها
- Docker 24+
- Docker Compose 2.20+

### مراحل

```bash
# ۱. کلون کردن
git clone https://github.com/your-org/itop-wrapper.git
cd itop-wrapper

# ۲. تنظیم متغیرهای محیطی
cp .env.example .env
nano .env   # مقادیر را تنظیم کنید

# ۳. راه‌اندازی
docker compose up -d

# ۴. بررسی وضعیت
docker compose ps
docker compose logs -f itop-wrapper
```

### دسترسی
| سرویس | آدرس | کاربر پیش‌فرض |
|---|---|---|
| پرتال مدیریتی | http://localhost/admin | admin / Admin@123 |
| مدیریت کیف‌پول مشتریان | http://localhost/admin/wallets | admin / Admin@123 |
| پرتال مشتریان | http://localhost/portal | — |
| کیف‌پول من (مشتری) | http://localhost/portal/wallet | — |
| موتور iTop | http://localhost/itop-engine | admin / Admin@123 |
| API Swagger | http://localhost/swagger-ui.html | — |

> ⚠️ **مهم:** رمز عبور پیش‌فرض `Admin@123` را بلافاصله پس از اولین ورود تغییر دهید!
> کیف‌پول کاربر `admin` به‌صورت خودکار با موجودی صفر ساخته می‌شود؛ از پنل `/admin/wallets` آن را شارژ کنید.

---

## 📁 ساختار پروژه

```
itop-wrapper/
├── src/main/java/com/itopwrapper/
│   ├── config/          # تنظیمات Spring Security، Beans
│   ├── controller/      # کنترلرهای MVC و REST API
│   │   ├── admin/       # پنل مدیریتی
│   │   └── portal/      # پرتال مشتریان
│   ├── model/
│   │   ├── entity/      # موجودیت‌های JPA/Hibernate
│   │   └── dto/         # اشیاء انتقال داده
│   ├── repository/      # Spring Data JPA Repositories
│   ├── service/impl/    # منطق کسب‌وکار
│   ├── security/        # JWT و Multi-LDAP
│   ├── itop/            # کلاینت REST API موتور iTop
│   ├── pricing/         # موتور قیمت‌گذاری
│   ├── module/          # مدیریت ماژول‌های پویا
│   └── audit/           # سرویس لاگ‌گیری
├── src/main/resources/
│   ├── templates/       # قالب‌های Thymeleaf (فارسی RTL)
│   │   ├── admin/       # قالب‌های پنل مدیریتی
│   │   └── portal/      # قالب‌های پرتال مشتریان
│   ├── static/          # CSS، JS، تصاویر
│   ├── db/changelog/    # Migration‌های Liquibase
│   └── i18n/            # پیام‌های فارسی
├── src/test/            # تست‌های واحد و یکپارچگی
├── docker/              # پیکربندی Docker (Nginx، Init SQL)
├── Dockerfile
├── docker-compose.yml
└── .env.example
```

---

## 🧩 توسعه ماژول سفارشی

### ساختار JAR
```
META-INF/MANIFEST.MF:
  iTop-Module-Id: my-custom-module
  iTop-Module-Name: ماژول سفارشی
  iTop-Module-Description: توضیح ماژول
  Implementation-Version: 1.0.0
  iTop-Module-Author: نام توسعه‌دهنده
  iTop-Entry-Point: com.example.MyModule
  iTop-Requires-Restart: false
```

### پیاده‌سازی Interface
```java
public class MyModule implements ITopModule {
    @Override public String getModuleId() { return "my-custom-module"; }
    @Override public String getModuleName() { return "ماژول سفارشی"; }
    @Override public String getVersion() { return "1.0.0"; }
    @Override public void onActivate() { /* کد فعال‌سازی */ }
    @Override public void onDeactivate() { /* کد غیرفعال‌سازی */ }
}
```

---

## 🔐 احراز هویت

### JWT
```bash
curl -X POST http://localhost/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"Admin@123","authType":"LOCAL"}'
```

### LDAP
```bash
curl -X POST http://localhost/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"john","password":"pass","authType":"LDAP"}'
```

---

## 📊 مدل‌های قیمت‌گذاری

| مدل | فرمول | مثال |
|---|---|---|
| FIXED | قیمت_پایه | ۵۰۰,۰۰۰ ریال |
| HOURLY | قیمت_پایه × ساعت | ۱۰۰,۰۰۰ × ۸ = ۸۰۰,۰۰۰ |
| MONTHLY | قیمت_پایه × ماه | ۳۰۰,۰۰۰ × ۱۲ = ۳,۶۰۰,۰۰۰ |
| PER_USER | قیمت_پایه × کاربر | ۵۰,۰۰۰ × ۲۵ = ۱,۲۵۰,۰۰۰ |
| TIERED | تخفیف پله‌ای تا ۳۰٪ | ۱۰۰+ واحد: ۷۰٪ |
| FORMULA | JavaScript | `basePrice * months * 0.9` |

تخفیف سطح مشتری: BRONZE=0٪ | SILVER=5٪ | GOLD=10٪ | PLATINUM=15٪

---

## 🔧 تنظیمات LDAP

تنظیمات LDAP از طریق پنل مدیریتی قابل مدیریت است:

`پنل مدیریتی → LDAP → افزودن سرور`

پشتیبانی از:
- OpenLDAP: `(uid={0})`
- Microsoft Active Directory: `(sAMAccountName={0})`
- Email login: `(mail={0})`

---

## 🐳 دستورات Docker مفید

```bash
# مشاهده لاگ‌ها
docker compose logs -f itop-wrapper

# ری‌استارت سرویس
docker compose restart itop-wrapper

# بک‌آپ پایگاه داده
docker compose exec postgres pg_dump -U itopwrapper itopwrapper > backup.sql

# ورود به shell PostgreSQL
docker compose exec postgres psql -U itopwrapper itopwrapper

# بروزرسانی
docker compose pull && docker compose up -d --build
```

---

## 📄 API Documentation

پس از راه‌اندازی، مستندات Swagger در آدرس زیر در دسترس است:
```
http://localhost/swagger-ui.html
```

---

## 🛠️ توسعه محلی

```bash
# ۱. راه‌اندازی پایگاه داده و iTop
docker compose up postgres itop itop-mysql -d

# ۲. اجرای اپلیکیشن با پروفایل dev
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# ۳. اجرای تست‌ها
./mvnw test
```

---

## 📝 License

این پروژه تحت مجوز MIT منتشر شده است.

---
