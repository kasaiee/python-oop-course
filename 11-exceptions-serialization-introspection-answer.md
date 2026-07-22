# پاسخ تمرین‌های فصل یازدهم: استثناها، سریال‌سازی و Introspection

> پاسخ‌ها به‌ترتیب پرسش‌های `11-exceptions-serialization-introspection-exercise.md`. هرجا راه دیگری هم بود، بخش **راه دیگر** آورده شده است.

---

## تمرین ۱ - استثناها

**۱.** در یک پروژه‌ی واقعی، خطاهای **دامنه‌ی خاص** خودتان را دارید («موجودی کافی نیست»، «کارت منقضی شده»). استفاده از `ValueError` عمومی برای همه‌ی این‌ها، **معنا و امکان مدیریت دقیق** را از دست می‌دهد — نمی‌توانید فقط «خطاهای پرداخت» را جدا بگیرید. استثنای سفارشی می‌تواند **داده‌ی اضافی** حمل کند:

```python
class InsufficientFundsError(Exception):
    def __init__(self, needed, available):
        self.needed = needed
        self.available = available
        super().__init__(f"Needed: {needed}, available: {available}")
```

حالا در `except` می‌توانید به `e.needed` و `e.available` دسترسی داشته باشید — داده‌ی ساختارمند برای تصمیم‌گیری.

**۲.**

```python
class ShopError(Exception):
    """base of all shop errors"""

class PaymentError(ShopError):
    pass

class InventoryError(ShopError):
    pass

class OutOfStockError(InventoryError):
    pass

# catching at the middle level:
try:
    raise OutOfStockError("Keyboard is out of stock")
except InventoryError as e:
    print(f"Inventory problem: {e}")

# catching at the top level:
try:
    raise OutOfStockError("Keyboard is out of stock")
except ShopError as e:
    print(f"Shop error: {e}")
```

چون `OutOfStockError` زیرکلاس هر دو `InventoryError` و `ShopError` است، هر دو `except` آن را می‌گیرند. این انعطاف اجازه می‌دهد خطا را در **سطح دلخواه** مدیریت کنید.

**۳.** `raise NewError(...) from original` رابطه‌ی **علّی** بین دو استثنا را حفظ می‌کند: می‌گوید «`NewError` به‌خاطر `original` رخ داد». `__cause__` به همان استثنای اصلی اشاره می‌کند (`e.__cause__`). مزیت در دیباگ: هم یک پیام سطح‌بالا و معنادار به کاربر می‌دهید (مثلاً `ConfigError`)، و هم خطای اصلی (مثلاً `FileNotFoundError`) را برای ردیابی حفظ می‌کنید؛ در traceback هر دو نمایش داده می‌شوند («The above exception was the direct cause...»).

**۴.** `raise ... from None` باعث می‌شود خطای اصلی در traceback **پنهان** شود (با فعال‌کردن `__suppress_context__`). مفید است وقتی خطای اصلی یک جزئیات داخلی بی‌ربط است که نمی‌خواهید کاربر ببیند و فقط پیام تمیز خودتان را می‌خواهید نشان دهید. **Tradeoff:** اطلاعات دیباگ را هم پنهان می‌کند؛ اگر بعداً بخواهید بفهمید ریشه‌ی خطا چه بود، آن اطلاعات از دست رفته. پس با احتیاط به‌کار ببرید.

---

## تمرین ۲ - سریال‌سازی

**۵.** سریال‌سازی یعنی تبدیل یک شیء به فرمتی که بتوان ذخیره (روی دیسک) یا ارسال (روی شبکه) کرد و بعداً بازسازی‌اش کرد. دو تفاوت `json` و `pickle`:
- `json` **متنی و بین‌زبانی** است (هر زبانی می‌تواند بخواند) اما فقط انواع ساده (dict/list/str/int...) را می‌شناسد؛ `pickle` **باینری و مخصوص پایتون** است اما هر شیء پایتونی را ذخیره می‌کند.
- `json` **امن** است؛ `pickle` **ناامن** است (می‌تواند هنگام بازخوانی کد دلخواه اجرا کند).

**۶.**

```python
import json

class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def to_dict(self):
        return {"name": self.name, "age": self.age}

    @classmethod
    def from_dict(cls, data):
        return cls(data["name"], data["age"])

u = User("ali", 30)
text = json.dumps(u.to_dict(), ensure_ascii=False)
print(text)                                # {"name": "ali", "age": 30}

restored = User.from_dict(json.loads(text))
print(restored.name)                       # ali
```

**راه دیگر (پارامتر `default`):** به‌جای `to_dict`، می‌توان یک تابع کدگذار به `json.dumps` داد که برای انواع ناشناخته صدا زده می‌شود:

```python
def encode(obj):
    if isinstance(obj, User):
        return {"name": obj.name, "age": obj.age}
    raise TypeError(f"not serializable: {type(obj)}")

json.dumps(u, default=encode, ensure_ascii=False)
```

**۷.** `__getstate__` تعیین می‌کند **چه چیزی** هنگام pickle ذخیره شود، و `__setstate__` تعیین می‌کند شیء **چطور** از آن حالت بازسازی شود. در کد نمونه، `socket` یک «اتصال زنده» است که **قابل سریال نیست** (نمی‌توان یک اتصال شبکه را به بایت تبدیل و بعداً همان را زنده کرد). پس `__getstate__` آن را از حالت ذخیره حذف می‌کند، و `__setstate__` هنگام بازسازی یک اتصال **تازه** می‌سازد. این الگوی رایج برای attributeهای غیرقابل‌سریال (اتصال، فایل باز، قفل) است.

**۸.** `pickle` هنگام بازخوانی (`pickle.loads`) می‌تواند **کد دلخواه اجرا کند**؛ یک فایل pickle مخرب می‌تواند فایل‌ها را پاک کند، فرمان اجرا کند و... . قاعده‌ی مطلق: **هرگز داده‌ی pickle نامطمئن (از کاربر، شبکه، اینترنت) را باز نکنید.** جایگزین‌های امن: `json` برای داده‌ی ساده، و کتابخانه‌هایی مثل Protocol Buffers یا MessagePack برای داده‌ی پیچیده‌تر.

---

## تمرین ۳ - Introspection

**۹.**
- `obj.__dict__`: دیکشنری **instance attributeها**ی شیء (فقط داده‌ای که روی خود شیء نشسته).
- `dir(obj)`: فهرست **همه‌ی** نام‌های در‌دسترس (متدها، attributeها، اعضای ارث‌بری‌شده، متدهای جادویی).
- `vars(obj)`: معادل `obj.__dict__` — فقط instance attributeها.

خلاصه: `__dict__`/`vars` داده‌ی خالص شیء را می‌دهند؛ `dir` همه‌ی قابلیت‌ها را.

**۱۰.** این‌ها اجازه می‌دهند در زمان اجرا با attributeها **به‌صورت پویا** کار کنید:
- `hasattr(obj, "x")`: آیا شیء این attribute را دارد؟
- `getattr(obj, "x")`: مقدارش را بگیر.
- `setattr(obj, "x", v)`: مقدار بده.

کاربرد اصلی: وقتی **نام** attribute در زمان نوشتن کد معلوم نیست (مثلاً از فایل تنظیمات یا ورودی کاربر می‌آید). در `getattr(p, "price", 0)`، آرگومان سوم یک **مقدار پیش‌فرض** است: اگر `price` وجود نداشت، به‌جای خطا مقدار `0` برگردانده می‌شود.

**۱۱.** `__annotations__` دیکشنری‌ای است که **تایپ‌هینت‌های** یک کلاس یا تابع را در زمان اجرا نگه می‌دارد (مثل `{'name': <class 'str'>, 'age': <class 'int'>}`). ارتباط با dataclass و Pydantic: این ابزارها همین `__annotations__` را می‌خوانند تا بفهمند کلاس چه فیلدهایی با چه انواعی دارد و بر اساس آن `__init__`، `__repr__` و اعتبارسنجی را بسازند. یعنی قدرت آن‌ها از دسترسی زمان اجرا به تایپ‌هینت‌ها می‌آید.

**۱۲.** `inspect` ابزارهای بررسی عمیق کلاس‌ها و توابع می‌دهد. دو کاربرد:
- **فهرست متدها:** `inspect.getmembers(Cls, predicate=inspect.isfunction)`.
- **امضای یک تابع:** `inspect.signature(func)` که پارامترها را می‌دهد (`['self', 'a', 'b']`).

این‌ها پایه‌ی ابزارهایی مثل مستندساز خودکار، تزریق وابستگی، و تست‌رانرها هستند.

---

## تمرین ۴ - کدنویسی و کدخوانی

**۱۳.** خروجی:

```python
p.__dict__   # {'name': 'Keyboard'}
vars(p)      # {'name': 'Keyboard'}
```

`category` در `p.__dict__` **نیست**، چون یک **Class Variable** است (روی خود کلاس نشسته، نه روی نمونه). `__dict__` نمونه فقط instance attributeها (اینجا `name`) را نشان می‌دهد. `category` را در `Product.__dict__` (دیکشنری کلاس) پیدا می‌کنید، نه در `p.__dict__`.

**۱۴.**

```python
from contextlib import contextmanager
import time

@contextmanager
def timer(name):
    start = time.perf_counter()
    try:
        yield
    finally:
        print(f"{name}: {time.perf_counter() - start:.4f}s")

with timer("compute"):
    sum(range(1_000_000))
# Output: compute: 0.0XXXs
```

`yield` جایی است که بلوک `with` اجرا می‌شود؛ کد قبل از آن نقش `__enter__` و کد بعد از آن (در `finally`) نقش `__exit__` را دارد. `finally` تضمین می‌کند حتی با خطا، زمان چاپ شود.

---

## تمرین ۵ - پرسش تحلیلی

**۱۵.** Introspection قدرتمند است اما استفاده‌ی افراطی از آن کد را **مبهم و سخت‌فهم** می‌کند. مثال: به‌جای نوشتن صریح `user.name`، اگر همه‌جا `getattr(user, field_name)` بنویسید که `field_name` از یک رشته‌ی متغیر می‌آید، خواننده‌ی کد دیگر نمی‌داند چه attributeهایی واقعاً استفاده می‌شوند؛ ابزارهای تحلیل کد و تکمیل خودکار هم کور می‌شوند، و خطاها به زمان اجرا موکول می‌شوند. استفاده‌ی موجه: وقتی نام attribute **واقعاً** در زمان نوشتن کد معلوم نیست (پیکربندی پویا، فریمورک، سریال‌سازی عمومی). قاعده: کد **صریح** را ترجیح دهید؛ فقط وقتی پویایی **ذاتی** مسئله است سراغ `getattr`/`setattr` بروید.

**۱۶.** ساختن یک استثنای پایه برای کل پروژه، تلاش کمی می‌طلبد (چند خط کلاس خالی) اما سود بلندمدت دارد:
- **مدیریت خطای منعطف:** می‌توانید خطا را در هر سطحی بگیرید (خاص یا عمومی) بدون بازنویسی.
- **افزودن آسان خطاهای جدید:** خطای تازه فقط زیر همان سلسله‌مراتب می‌رود و کدهای موجود `except` همچنان کار می‌کنند.

نفع برای کاربر کتابخانه: او می‌تواند با یک `except YourLibError` **همه‌ی** خطاهای کتابخانه‌ی شما را یکجا بگیرد، بی‌آنکه مجبور باشد ده‌ها استثنای مختلف را جداگانه بشناسد و بگیرد. این همان چیزی است که تفاوت یک کتابخانه‌ی حرفه‌ای و آماتور را می‌سازد.
