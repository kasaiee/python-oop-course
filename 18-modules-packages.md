# فصل هجدهم: ماژول‌ها، پکیج‌ها و شیءگرایی

---

## مقدمه

تا اینجا همه‌ی کد را در یک فایل تصور کردیم. اما پروژه‌ی واقعی، ده‌ها یا صدها کلاس دارد که نمی‌توان در یک فایل ریخت. این فصل به رابطِ بینِ شیءگرایی و **سازمان‌دهیِ فیزیکیِ کد** — ماژول‌ها و پکیج‌ها — می‌پردازد.

یاد می‌گیریم کلاس‌ها را چطور در ماژول‌ها بچینیم، پکیج بسازیم، `__init__.py` و `__all__` را درست به‌کار ببریم، و چرا خودِ **ماژول‌ها** یک شکلِ طبیعیِ الگوی Singleton‌اند. سازمان‌دهیِ خوب، تفاوتِ پروژه‌ای است که می‌توان توسعه‌اش داد با پروژه‌ای که هرکس واردش می‌شود گم می‌شود.

---

## چرا سازمان‌دهی ماژول‌ها و پکیج‌ها را یاد بگیریم؟

نوشتنِ چند کلاس در یک فایل آسان است. اما در پروژه‌ای با صدها کلاس، سازمان‌دهیِ مناسب حیاتی است. یادگیریِ این مباحث کمک می‌کند:

- کد را منظم و قابلِ‌توسعه بچینید
- از نام‌گذاری‌های تکراری و تصادمِ نام جلوگیری کنید
- پکیج‌های قابلِ‌استفاده‌ی مجدد بسازید
- کدی بنویسید که دیگران به‌راحتی از آن استفاده کنند

---

## چگونه سازمان‌دهی ماژول‌ها و پکیج‌ها را یاد بگیریم؟

### ماژول چیست و چرا مثل Singleton است؟

هر فایلِ `.py` یک **ماژول** است. وقتی ماژولی را `import` می‌کنید، پایتون آن را **فقط یک‌بار** اجرا می‌کند و نتیجه را کش می‌کند؛ importهای بعدی، همان شیءِ ماژولِ کش‌شده را برمی‌گردانند.

این دقیقاً رفتارِ **Singleton** است: یک نمونه‌ی واحد که در سراسرِ برنامه به اشتراک گذاشته می‌شود.

```python
# file: config.py
print("config loaded")
settings = {"debug": True}

# file: main.py
import config      # Output: config loaded
import config      # no output — it came from cache, didn't run again

config.settings["debug"] = False
# anywhere else that imports config sees this same change
```

پس اگر یک حالتِ سراسریِ مشترک می‌خواهید (مثلِ تنظیمات یا یک اتصالِ مشترک)، اغلب لازم نیست کلاسِ Singleton بنویسید؛ فقط از یک ماژول استفاده کنید. این، پایتونی‌ترین شکلِ Singleton است.

### پکیج چیست؟

یک **پکیج** پوشه‌ای است که چند ماژول را گروه‌بندی می‌کند. ساختارِ یک پکیجِ شیءگرا معمولاً این شکل را دارد:

```
shop/                       ← پکیج (پوشه)
├── __init__.py             ← این پوشه را پکیج می‌کند
├── models.py               ← کلاس‌های داده (User, Product, Order)
├── services.py             ← منطقِ کسب‌وکار (OrderService)
├── repositories.py         ← دسترسی به داده (UserRepository)
└── exceptions.py           ← استثناهای سفارشی (ShopError...)
```

هر ماژول یک مسئولیتِ منسجم دارد — همان اصلِ Single Responsibility، اما در سطحِ فایل. این چیدمان، لایه‌های معماریِ فصل هفتم را به ساختارِ فیزیکیِ پروژه ترجمه می‌کند.

### __init__.py

فایلِ `__init__.py` به پایتون می‌گوید این پوشه یک پکیج است. می‌تواند خالی باشد، یا کارِ مفیدی بکند: مثلاً یک **رابطِ عمومیِ تمیز** برای پکیج بسازد.

```python
# shop/__init__.py
from .models import User, Product, Order
from .services import OrderService
from .exceptions import ShopError

# now the user can import directly from the package:
#   from shop import User, OrderService
# instead of the long path:
#   from shop.models import User
#   from shop.services import OrderService
```

با این کار، جزئیاتِ داخلی (اینکه `User` در `models.py` است) پنهان می‌ماند و کاربر یک رابطِ ساده می‌بیند — همان انتزاعِ فصل چهارم، در سطحِ پکیج.

### __all__

`__all__` فهرستِ نام‌هایی را مشخص می‌کند که با `from module import *` وارد می‌شوند. این هم رابطِ عمومیِ ماژول را مشخص می‌کند و هم از واردشدنِ نام‌های داخلی جلوگیری:

```python
# shop/models.py

__all__ = ["User", "Product", "Order"]     # only these are public

class User: ...
class Product: ...
class Order: ...
class _InternalHelper: ...      # starts with _ and is not in __all__ → private

# elsewhere:
# from shop.models import *
# only User, Product, Order are imported, not _InternalHelper
```

`__all__` یک قرارداد است: «این‌ها API عمومیِ من‌اند؛ بقیه جزئیاتِ داخلی‌اند.» حتی اگر از `import *` استفاده نکنید، تعریفش به خواننده می‌گوید چه چیزی عمومی است.

**Tradeoff:** `import *` عموماً توصیه نمی‌شود (نام‌ها را نامرئی وارد می‌کند و خوانایی را کم می‌کند). ترجیحاً importِ صریح بنویسید. اما `__all__` حتی بدونِ `import *` هم برای مستندکردنِ رابطِ عمومی مفید است.

### import در برابر from ... import

دو سبکِ import، و اثرشان بر فضای‌نام:

```python
# style 1: import the module
import shop.models
user = shop.models.User()      # with the full path — it's clear where User comes from

# style 2: from ... import
from shop.models import User
user = User()                  # shorter, but the origin is less visible
```

**Tradeoff:** `import shop.models` روشن‌تر است (همیشه می‌دانید `User` از کجا آمده) اما پرگوتر. `from shop.models import User` مختصرتر است اما در فایلی با importهای زیاد، منشأِ نام‌ها گم می‌شود. قاعده‌ی رایج: برای نام‌های پرکاربرد `from import`، و وقتی چند ماژول نامِ مشابه دارند (`shop.models.User` و `auth.models.User`)، `import`ِ ماژول برای رفعِ ابهام.

### طراحی پکیج شیءگرا با Factory

الگوی Factory (فصل پنجم) در سطحِ پکیج هم مفید است: یک تابع یا کلاسِ کارخانه که بر اساسِ ورودی، شیءِ مناسب را می‌سازد و کاربر را از جزئیاتِ کلاس‌های داخلی بی‌نیاز می‌کند.

```python
# shop/notifications.py
class EmailNotifier:
    def send(self, msg): return f"Email: {msg}"

class SMSNotifier:
    def send(self, msg): return f"SMS: {msg}"

def create_notifier(kind: str):        # Factory at module level
    notifiers = {
        "email": EmailNotifier,
        "sms": SMSNotifier,
    }
    if kind not in notifiers:
        raise ValueError(f"unknown type: {kind}")
    return notifiers[kind]()

# the user only sees this:
#   from shop.notifications import create_notifier
#   notifier = create_notifier("email")
```

کاربر نیازی ندارد بداند `EmailNotifier` و `SMSNotifier` وجود دارند؛ فقط `create_notifier("email")` را صدا می‌زند. افزودنِ نوعِ جدید، فقط تغییرِ داخلیِ Factory است — رابطِ عمومی ثابت می‌ماند (Open/Closed).

### Singleton در سطح ماژول

اگر واقعاً به یک نمونه‌ی مشترکِ سراسری نیاز دارید، به‌جای کلاسِ Singleton (که در پایتون اغلب اضافی است)، از الگوی ماژولی استفاده کنید:

```python
# shop/database.py
class _Database:
    def __init__(self):
        self.connection = "single connection"
    def query(self, sql):
        return f"Result: {sql}"

# a single instance is created at module level:
db = _Database()

# anywhere in the program:
#   from shop.database import db
#   db.query("SELECT ...")
# all access the same single instance
```

چون ماژول فقط یک‌بار اجرا می‌شود، `db` یک‌بار ساخته می‌شود و همه‌ی importها همان نمونه را می‌گیرند. کلاسِ `_Database` با `_` خصوصی شده تا کسی مستقیم نمونه‌ی دومی نسازد. این، ساده‌ترین و پایتونی‌ترین Singleton است.

**Tradeoff:** Singletonِ سراسری (چه ماژولی چه کلاسی) حالتِ سراسریِ مشترک می‌سازد که تست را سخت‌تر و وابستگی‌ها را پنهان‌تر می‌کند. اغلب، **تزریقِ وابستگی** (فصل هشتم) — پاس‌دادنِ صریحِ شیء به جایی که لازمش دارد — انتخابِ تمیزتری است. Singleton را فقط برای منابعی که واقعاً باید واحد باشند (مثلِ استخرِ اتصالِ دیتابیس) نگه دارید.

---

## جمع‌بندی یک ساختار نمونه

بیایید همه‌ی این‌ها را در یک ساختارِ کاملِ پروژه ببینیم:

```
shop/
├── __init__.py          # رابطِ عمومی: from shop import User, OrderService
├── models.py            # @dataclass User, Product, Order  (فصل ۱۰)
├── exceptions.py        # سلسله‌مراتبِ ShopError  (فصل ۱۱)
├── repositories.py      # UserRepository, ProductRepository  (لایه‌ی داده، فصل ۷)
├── services.py          # OrderService  (لایه‌ی منطق، فصل ۷)
├── notifications.py     # create_notifier Factory  (فصل ۵)
└── database.py          # db  (Singletonِ ماژولی)
```

هر فایل یک مسئولیت، `__init__.py` رابطِ تمیز، و وابستگی‌ها از لایه‌ی بالا به پایین جاری. این همان معماریِ لایه‌ای فصل هفتم است که حالا به فایل‌ها و پوشه‌ها ترجمه شده. کسی که این پروژه را باز کند، از روی نامِ فایل‌ها می‌فهمد هر چیزی کجاست.

---

## خلاصه

```
سازمان‌دهی کد
════════════════════════════════════════════════
  فایلِ .py        =  ماژول  (رفتارِ Singleton)
  پوشه + __init__ =  پکیج
  __init__.py     →  رابطِ عمومیِ تمیز
  __all__         →  مشخص‌کردنِ API عمومی
```

| مفهوم | نکته‌ی کلیدی |
|-------|--------------|
| ماژول | فایلِ `.py`؛ یک‌بار اجرا، مثلِ Singleton |
| پکیج | پوشه با `__init__.py` |
| `__init__.py` | رابطِ عمومیِ پکیج، پنهان‌کردنِ ساختارِ داخلی |
| `__all__` | تعریفِ API عمومی؛ کنترلِ `import *` |
| `import` در برابر `from` | صراحت در برابر اختصار (Tradeoff) |
| Factory ماژولی | ساختِ شیء بدونِ افشای کلاس‌های داخلی |
| Singleton ماژولی | نمونه‌ی واحدِ سراسری، ساده‌ترین شکل |

**نکته‌ی طلایی:** سازمان‌دهیِ خوب، همان اصولِ شیءگرایی است که به سطحِ فایل‌ها بالا رفته: هر ماژول یک مسئولیت (SRP)، `__init__.py` یک انتزاع، و لایه‌بندیِ وابستگی‌ها. یک پروژه‌ی خوش‌ساختار، بدونِ خواندنِ حتی یک خط کد، از روی نامِ پوشه‌ها و فایل‌هایش خودش را توضیح می‌دهد.

---

## پرسش‌های مصاحبه

**۱. «چرا می‌گویند ماژول در پایتون Singleton است؟»** — چون `import` هر ماژول را فقط یک‌بار اجرا و در `sys.modules` کش می‌کند؛ importهای بعدی همان شیء را می‌گیرند. نتیجه‌ی عملی: حالتِ سطحِ ماژول، سراسری و مشترک است — هم Singletonِ مجانی، هم (اگر حواستان نباشد) حالتِ سراسریِ پنهانی که تست‌ها را به هم گره می‌زند.

**۲. «در __init__.py چه می‌گذاری و چه نمی‌گذاری؟»** — می‌گذارم: رابطِ عمومیِ پکیج (re-exportهای منتخب) و نهایتاً چند ثابت. نمی‌گذارم: منطقِ سنگین یا کارِ دارای عوارض (اتصال به دیتابیس!) — چون به‌محضِ import اجرا می‌شود و کندی/غافلگیری می‌سازد. `__init__.py` درِ ورودیِ خانه است، نه موتورخانه.

**۳. «ساختارِ ماژول‌بندیِ یک پروژه‌ی متوسط را چطور می‌چینی؟»** — بر اساسِ مسئولیت، آینه‌ی معماریِ لایه‌ای: `models`/`services`/`repositories`/`exceptions`. اصلِ راهنما: SRP در سطحِ فایل — «این تغییر، منطقاً کدام فایل را باید لمس کند؟» اگر جواب «سه فایلِ بی‌ربط» شد، چینش غلط است.

---

**در فصل بعد:** به هیجان‌انگیزترین بخشِ دوره می‌رسیم: **پروژه‌های عملیِ کامل** — از سیستمِ کتابخانه تا ساختنِ ORM و فریمورکِ تستِ خودتان — که در آن‌ها همه‌ی مفاهیمِ دوره را در قالبِ سیستم‌های واقعی به‌کار می‌بریم.
