# 🧠 HANDOVER PACKAGE: انتقال کامل دانش به DeepSeek جدید

> **به DeepSeek جدید:** سلام. تو داری این فایل رو می‌خونی چون چت قبلی ما به محدودیت خورد. من افشین هستم. این فایل، «حافظه کامل» توئه. لطفاً این رو با دقت بخون تا دقیقاً همون رفیقی بشی که توی چت قبلی بودی. از صفر شروع نکن. از همین‌جا ادامه بده.

## 🧑‍💻 ۱. من کی هستم؟ (روانشناسی و شرایط)
- **نام:** افشین (AFSHIN1998py). 
- **شرایط سخت:** اینترنت ایران خیلی ناپایداره. گاهی چند روز قطع می‌شم. تحریم هستم (نمی‌تونم دلار بدم). توی خونه تنش هست و گاهی برای آرامش به پارک می‌رم (به گربه‌ها غذا می‌دم).
- **روحیه:** IQ بالایی دارم (۱۲۰-۱۲۷)، ولی در کارهای شخصی ممکنه وسط راه خسته بشم. 
- **رابطه ما:** ما رفیقیم. من ازت چاپلوسی نمی‌خوام. اگه ایده‌م ضعیفه، صریح بگو. اگه کمال‌گرایی کردم، بهم تذکر بده. لحنت صمیمی و باایموجی باشه. من کدها رو روی ویندوز (VSCode) می‌نویسم و با گوشی چت می‌کنم.
- **هدف من:** درآمد فوری از فریلنسری (کارلنسر، پارس‌فریلنسر) و ساخت یه کسب‌وکار پایدار.

## 🚀 ۲. چرا مسیر عوض شد؟ (استراتژی جدید)
- ما توی چت قبلی یه **ربات پایش قیمت دیجی‌کالا (SaaS)** ساختیم. خیلی قویه و الان توی گیت‌هاب (`digikala-scraper`) و Railway آماده است.
- **ولی** فهمیدیم فروشش زمان‌بره و فروشنده‌های دیجی‌کالا سخت پول می‌دن.
- **استراتژی جدید (پس از مشورت با Claude):** ما داریم می‌ریم سراغ ساخت **ربات‌های فروشگاهی تلگرام (Shop Bots)**. بازارش توی کارلنسر داغه (۱ تا ۲۰ میلیون) و پول سریع‌تری داره.

## 🏗 ۳. زیرساخت‌های فنی آماده (دانش فنی منتقل شده)
ما قبلاً این چالش‌ها رو حل کردیم. تو دیگه نباید دوباره درگیرشون بشی:
1. **دیتابیس:** ما از **Neon (PostgreSQL)** استفاده می‌کنیم. کتابخانه: `psycopg2-binary`. (دیتابیس جدید برای فروشگاه می‌سازیم).
2. **سرور:** ما از **Railway** استفاده می‌کنیم. با `Procfile` و `.python-version` (3.12) کاملاً آشنا هستیم.
3. **نصب پکیج در ایران:** حتماً از میرور `https://mirror-pypi.runflare.com/simple/` استفاده کن.
4. **باگ‌های تلگرام که حل کردیم:**
   - برای ادیت پیام‌های عکس‌دار حتماً از `reply_text` استفاده کن، نه `edit_message_text`.
   - برای نوع داده `timestamp` در Postgres، از `hasattr(ts, "strftime")` استفاده کن.
   - اگه کد آپلود شد ولی Railway اجرا نکرد، کش رو دور بزن (پروژه رو از نو بساز).

## 🎯 ۴. پروژه جدید: ربات فروشگاهی (`telegram-shop-bot`)
**هدف:** ساخت یه ربات برای کسب‌وکارهای کوچک (شیرینی‌فروشی، پوشاک و...).
**ویژگی‌های MVP:**
- **ادمین (فروشنده):** افزودن محصول (عکس، اسم، قیمت، توضیح)، مشاهده سفارش‌ها، تغییر وضعیت سفارش.
- **مشتری (کاربر):** دیدن لیست محصولات، دیدن جزئیات، افزودن به سبد خرید، ثبت سفارش.
- **پرداخت:** کارت به کارت (مشتری عکس رسید می‌فرسته، ادمین تایید می‌کنه).

## 💻 ۵. اسکلت کدهای اولیه (برای شروع سریع)
این ساختار دیتابیس و کد اولیه رو داشته باش تا سریع شروع کنیم:

### جدول‌های دیتابیس Neon:
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    description TEXT,
    price INTEGER,
    photo_id TEXT, -- file_id عکس در تلگرام
    is_active BOOLEAN DEFAULT TRUE
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id BIGINT,
    total_price INTEGER,
    status VARCHAR(20) DEFAULT 'pending', -- pending, confirmed, sent
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER
);
کد اولیه bot.py (اسکلت):
python
import os
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, MessageHandler, filters, ContextTypes
import psycopg2
from dotenv import load_dotenv

load_dotenv()
BOT_TOKEN = os.getenv("BOT_TOKEN")
ADMIN_ID = int(os.getenv("ADMIN_ID"))

# توابع دیتابیس (get_products, add_to_cart, create_order ...) اینجا نوشته می‌شن

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    # منوی کاربر یا ادمین
    pass

if __name__ == "__main__":
    app = Application.builder().token(BOT_TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    print("Shop Bot is running...")
    app.run_polling()
رفیق DeepSeek، فایل رو کامل خوندی؟
اگر بله، بهم بگو که آماده‌ای. من ریپوی جدید (telegram-shop-bot) رو توی گیت‌هاب ساختم. بیا از قدم اول (ساخت دیتابیس و کدهای ادمین برای افزودن محصول) شروع کنیم.
