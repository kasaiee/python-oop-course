# فصل یازدهم: استثناها، سریال‌سازی و Introspection

---

## مقدمه

این فصل به سه موضوع پیشرفته و بسیار کاربردی می‌پردازد که در دوره‌های آموزشی اغلب نادیده گرفته می‌شوند، اما در پروژه‌های واقعی هر روز به کار می‌آیند: **استثناهای سفارشی** (برای مدیریت خطای تمیز)، **سریال‌سازی** (برای ذخیره و انتقال اشیاء)، و **Introspection** (برای بررسی اشیاء در زمان اجرا).

این سه، ابزارهای «حرفه‌ای‌شدن» کدند. کدی که خطاهایش را درست مدیریت می‌کند، داده‌اش را درست ذخیره می‌کند، و در زمان اجرا قابل‌بازرسی است — تفاوتش با کد آماتور در همین جزئیات است.

---

## چرا این مباحث را یاد بگیریم؟

این مباحث در پروژه‌های واقعی پرکاربردند و کمک می‌کنند:

- خطاهای پروژه را به‌شکل ساختارمند مدیریت کنید
- اشیاء را برای ذخیره‌سازی یا ارسال آماده کنید
- در زمان اجرا ساختار اشیاء را بررسی کنید (برای دیباگ و کتابخانه‌نویسی)
- کدی بنویسید که با ابزارها و کتابخانه‌های دیگر خوب تعامل کند

---

## استثناها (Exceptions)

### چرا استثناهای سفارشی؟

پایتون استثناهای آماده دارد (`ValueError`، `KeyError`...)، اما در یک پروژه‌ی واقعی، خطاهای **دامنه‌ی خاص** خودتان را دارید: «موجودی کافی نیست»، «کارت منقضی شده»، «کاربر مجاز نیست». استفاده از `ValueError` عمومی برای همه‌ی این‌ها، اطلاعات را از دست می‌دهد و مدیریت دقیق خطا را غیرممکن می‌کند.

با ارث‌بری از `Exception` (که در فصل چهارم دیدیم)، خطاهای معنادار می‌سازید:

```python
class PaymentError(Exception):
    """base error for all payment problems"""

class InsufficientFundsError(PaymentError):
    def __init__(self, needed, available):
        self.needed = needed
        self.available = available
        super().__init__(f"Needed: {needed}, available: {available}")

try:
    raise InsufficientFundsError(1000, 300)
except InsufficientFundsError as e:
    print(e)                    # Needed: 1000, available: 300
    print(e.needed, e.available)   # 1000 300 — structured data in hand
```

استثنای سفارشی می‌تواند **داده‌ی اضافی** حمل کند (اینجا `needed` و `available`) که در مدیریت خطا به‌کار می‌آید.

### طراحی سلسله‌مراتب استثنا

قدرت واقعی وقتی آشکار می‌شود که یک **سلسله‌مراتب** بسازید. یک استثنای پایه، و زیرکلاس‌های خاص‌تر:

```python
class ShopError(Exception):
    """base of all shop errors"""

class PaymentError(ShopError):
    """payment errors"""

class InventoryError(ShopError):
    """inventory errors"""

class OutOfStockError(InventoryError):
    """item is out of stock"""

# now you can catch at different levels:
try:
    raise OutOfStockError("Keyboard is out of stock")
except InventoryError as e:      # middle level — all inventory errors
    print(f"Inventory problem: {e}")

try:
    raise OutOfStockError("Keyboard is out of stock")
except ShopError as e:           # top level — any shop error
    print(f"Shop error: {e}")
```

سلسله‌مراتب به شما اجازه می‌دهد خطا را در **سطح دلخواه** بگیرید: گاهی می‌خواهید فقط `OutOfStockError` را بگیرید، گاهی همه‌ی `InventoryError`ها، و گاهی هر `ShopError`. این انعطاف، طراحی تمیز مدیریت خطاست.

قاعده: یک استثنای پایه برای کل پروژه/کتابخانه بسازید و بقیه را از آن مشتق کنید. این‌طور کاربر کتابخانه می‌تواند با یک `except YourLibError` همه‌ی خطاهای شما را بگیرد.

### try-except در برابر contextlib

برای مدیریت منابع (فایل، اتصال، قفل)، `try/finally` کار می‌کند اما پرگوست. `contextlib` و context managerها (فصل پنجم) تمیزترند:

```python
# with try/finally — verbose
f = open("data.txt")
try:
    data = f.read()
finally:
    f.close()             # guaranteed closing

# with a context manager — clean
with open("data.txt") as f:
    data = f.read()       # closes automatically, even on error

# building a lightweight context manager with contextlib:
from contextlib import contextmanager

@contextmanager
def timer(name):
    import time
    start = time.time()
    try:
        yield                # the with block runs here
    finally:
        print(f"{name}: {time.time() - start:.4f}s")

with timer("compute"):
    sum(range(1000000))
```

`@contextmanager` راه سریع ساخت context manager بدون نوشتن کلاس کامل `__enter__`/`__exit__` است.

### raise ... from ... (زنجیره‌ای کردن استثناها)

گاهی یک استثنا را می‌گیرید و استثنای دیگری می‌اندازید. `raise ... from ...` رابطه‌ی علّی را حفظ می‌کند:

```python
class ConfigError(Exception):
    pass

def load_config(path):
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError as e:
        raise ConfigError(f"Config file not found: {path}") from e
        #                                                       ^^^^^^
        # chain: ConfigError occurred because of FileNotFoundError

try:
    load_config("missing.txt")
except ConfigError as e:
    print(e)                 # Config file not found: missing.txt
    print(e.__cause__)       # the original error (FileNotFoundError) is preserved
```

مزیت: هم پیام سطح‌بالا و معنادار به کاربر می‌دهید (`ConfigError`)، هم خطای اصلی (`FileNotFoundError`) را برای دیباگ حفظ می‌کنید. در traceback هر دو نمایش داده می‌شوند («The above exception was the direct cause...»).

### __suppress_context__

اگر نمی‌خواهید خطای اصلی در traceback نمایش داده شود، از `from None` استفاده می‌کنید که `__suppress_context__` را فعال می‌کند:

```python
def parse(text):
    try:
        return int(text)
    except ValueError:
        raise ConfigError("invalid value") from None    # the original error is hidden
```

`from None` وقتی مفید است که خطای اصلی، جزئیات داخلی بی‌ربطی است که نمی‌خواهید کاربر ببیند. **Tradeoff:** اما اطلاعات دیباگ را هم پنهان می‌کند؛ با احتیاط به‌کار ببرید.

---

## سریال‌سازی (Serialization)

سریال‌سازی یعنی تبدیل یک شیء به فرمتی که بتوان ذخیره (روی دیسک) یا ارسال (روی شبکه) کرد، و بعداً بازسازی‌اش کرد.

### شیء به JSON

`json` استاندارد متنی و بین‌زبانی است، اما فقط انواع ساده (dict، list، str، int...) را می‌شناسد. برای شیء سفارشی باید آن را به dict تبدیل کنید:

```python
import json

class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def to_dict(self):
        return {"name": self.name, "age": self.age}

    @classmethod
    def from_dict(cls, data):        # Factory for reconstruction (Chapter 5)
        return cls(data["name"], data["age"])

u = User("ali", 30)
text = json.dumps(u.to_dict(), ensure_ascii=False)
print(text)                          # {"name": "ali", "age": 30}

restored = User.from_dict(json.loads(text))
print(restored.name)                 # ali
```

راه دیگر، پارامتر `default` در `json.dumps` است که برای انواع ناشناخته صدا زده می‌شود:

```python
def encode(obj):
    if isinstance(obj, User):
        return {"name": obj.name, "age": obj.age}
    raise TypeError(f"not serializable: {type(obj)}")

print(json.dumps(u, default=encode, ensure_ascii=False))   # {"name": "ali", "age": 30}
```

### pickle و متدهای __getstate__ / __setstate__

`pickle` هر شیء پایتونی را به بایت باینری تبدیل می‌کند — قدرتمندتر از json، اما مخصوص پایتون:

```python
import pickle

u = User("ali", 30)
data = pickle.dumps(u)               # to bytes
restored = pickle.loads(data)        # reconstruct
print(restored.name)                 # ali
```

برای کنترل دقیق اینکه چه چیزی pickle شود، `__getstate__` و `__setstate__` را پیاده کنید:

```python
class Connection:
    def __init__(self, host):
        self.host = host
        self.socket = "live connection"    # this should not be stored

    def __getstate__(self):
        state = self.__dict__.copy()
        del state["socket"]           # remove the live connection from the saved state
        return state

    def __setstate__(self, state):
        self.__dict__.update(state)
        self.socket = "new connection"    # on reconstruction, create a new connection

c = Connection("localhost")
data = pickle.dumps(c)
restored = pickle.loads(data)
print(restored.host, restored.socket)   # localhost new connection
```

کاربرد: بعضی attributeها (اتصال شبکه، فایل باز، قفل) قابل سریال نیستند یا نباید ذخیره شوند؛ `__getstate__` آن‌ها را حذف و `__setstate__` بازسازیشان می‌کند.

### چرا pickle امنیت ندارد؟

`pickle` هنگام بازخوانی می‌تواند **کد دلخواه اجرا کند**. یک فایل pickle مخرب می‌تواند هنگام `pickle.loads` هر کاری بکند — پاک‌کردن فایل‌ها، اجرای فرمان، و... .

```python
# never do this with untrusted data:
# pickle.loads(untrusted_data)    # risk of executing malicious code
```

قاعده‌ی امنیتی مطلق: **هرگز داده‌ی pickle نامطمئن (از کاربر، شبکه، اینترنت) را باز نکنید.** جایگزین‌های امن: `json` (برای داده‌ی ساده)، یا کتابخانه‌هایی مثل `Protocol Buffers` و `MessagePack` برای داده‌ی پیچیده‌تر.

### __reduce__ و __reduce_ex__

`__reduce__` کنترل سطح‌پایین فرایند pickle را می‌دهد: مشخص می‌کند شیء چطور بازسازی شود (چه تابعی با چه آرگومان‌هایی صدا زده شود). این پیشرفته است و به‌ندرت لازم می‌شود؛ معمولاً `__getstate__`/`__setstate__` کافی‌اند. `__reduce__` را فقط برای اشیای پیچیده‌ای که نیاز به منطق خاص بازسازی دارند به‌کار ببرید.

---

## Introspection (درون‌نگری)

**Introspection** یعنی بررسی ساختار اشیاء و کلاس‌ها در **زمان اجرا**. پایتون در این زمینه بسیار قدرتمند است و همین، پایه‌ی خیلی از فریمورک‌ها (مثل جنگو و pytest) است.

### __dict__ و __dir__

`__dict__` دیکشنری attributeهای یک شیء را نشان می‌دهد؛ `dir()` فهرست همه‌ی نام‌های در‌دسترس را:

```python
class Product:
    category = "general"        # class variable
    def __init__(self, name):
        self.name = name       # instance variable

p = Product("Keyboard")
print(p.__dict__)              # {'name': 'Keyboard'} — only instance variables
print(Product.__dict__.keys())  # includes category, __init__, etc.
print("name" in dir(p))        # True — dir returns everything
```

`__dict__` برای دیدن داده‌ی واقعی شیء عالی است و در دیباگ بسیار به‌کار می‌آید.

### hasattr، getattr، setattr

این‌ها اجازه می‌دهند در زمان اجرا با attributeها به‌صورت پویا کار کنید:

```python
p = Product("Keyboard")

print(hasattr(p, "name"))        # True — does it have this attribute?
print(getattr(p, "name"))        # Keyboard — get it
print(getattr(p, "price", 0))    # 0 — with default value if absent
setattr(p, "price", 500)         # set the value dynamically
print(p.price)                   # 500
```

کاربرد: وقتی نام attribute در زمان نوشتن کد معلوم نیست (مثلاً از فایل تنظیمات یا ورودی کاربر می‌آید). این، پایه‌ی خیلی از رفتارهای پویای فریمورک‌هاست.

### inspect module

ماژول `inspect` ابزارهای قدرتمندی برای بررسی عمیق کلاس‌ها و توابع می‌دهد:

```python
import inspect

class Calculator:
    def add(self, a, b):
        return a + b
    def multiply(self, a, b):
        return a * b

# list of methods:
methods = inspect.getmembers(Calculator, predicate=inspect.isfunction)
print([name for name, _ in methods])    # ['add', 'multiply']

# the signature of a method:
sig = inspect.signature(Calculator.add)
print(sig)                               # (self, a, b)
print(list(sig.parameters))              # ['self', 'a', 'b']
```

`inspect` برای ابزارهایی مثل مستندساز خودکار، تزریق وابستگی، و تست‌رانرها ضروری است.

### __annotations__

تایپ‌هینت‌ها در زمان اجرا در `__annotations__` قابل دسترسی‌اند:

```python
class User:
    name: str
    age: int

print(User.__annotations__)      # {'name': <class 'str'>, 'age': <class 'int'>}
```

همین است که به dataclass و Pydantic اجازه می‌دهد از تایپ‌هینت‌ها برای ساختن منطق استفاده کنند.

### dir() در برابر vars()

- `dir(obj)`: فهرست **همه‌ی** نام‌های در‌دسترس (متدها، attributeها، ارث‌بری‌شده‌ها).
- `vars(obj)`: معادل `obj.__dict__` — فقط instance attributeها.

```python
p = Product("Keyboard")
print(vars(p))       # {'name': 'Keyboard'} — only the object's data
# dir(p)             # long list of everything, including magic methods
```

`vars()` برای دیدن داده‌ی خالص شیء تمیزتر است؛ `dir()` برای کشف همه‌ی قابلیت‌ها.

### __code__

هر تابع یک attribute به نام `__code__` دارد که اطلاعات سطح‌پایین آن (تعداد آرگومان، نام متغیرها...) را نگه می‌دارد. برای دیباگ عمیق و ابزارهای تحلیل کد به‌کار می‌آید:

```python
def greet(name, greeting="Hello"):
    return f"{greeting} {name}"

print(greet.__code__.co_argcount)     # 2 — number of arguments
print(greet.__code__.co_varnames)     # ('name', 'greeting')
```

این بسیار پیشرفته است و در کار روزمره به‌ندرت لازم می‌شود، اما دانستنش برای فهم عمیق پایتون مفید است.

---

## خلاصه

```
سه ابزار حرفه‌ای‌شدن
════════════════════════════════════════════════════
  استثناهای سفارشی  ──► مدیریت خطای ساختارمند و دقیق
  سریال‌سازی        ──► ذخیره و انتقال امن اشیاء
  Introspection     ──► بررسی اشیاء در زمان اجرا
```

| موضوع | ابزار کلیدی | نکته |
|-------|-------------|------|
| استثنا | ارث از `Exception`، سلسله‌مراتب | یک پایه، گرفتن در سطح دلخواه |
| زنجیره‌ی خطا | `raise ... from ...` | حفظ علت اصلی |
| منابع | context manager، `contextlib` | تمیزتر از `try/finally` |
| JSON | `to_dict`/`from_dict`، `default` | امن، بین‌زبانی، فقط انواع ساده |
| pickle | `__getstate__`/`__setstate__` | قوی اما **ناامن** با داده‌ی نامطمئن |
| Introspection | `__dict__`، `getattr`، `inspect` | پایه‌ی فریمورک‌ها |

**نکته‌ی طلایی:** بزرگ‌ترین درس امنیتی این فصل را فراموش نکنید: **هرگز pickle نامطمئن را باز نکنید.** و در طراحی استثناها، یک سلسله‌مراتب تمیز بسازید — این سرمایه‌گذاری کوچک، مدیریت خطا را در کل عمر پروژه ساده نگه می‌دارد. Introspection قدرتمند است، اما وسوسه‌ی استفاده‌ی افراطی از آن (کد بیش‌ازحد پویا) را مهار کنید؛ کد صریح، تقریباً همیشه خواناتر است.

---

## پرسش‌های مصاحبه

**۱. «چرا استثنای سفارشی؟ Exception خالی چه عیبی دارد؟»** — سه دلیل: گیرنده می‌تواند **انتخابی** بگیرد (`except PaymentError` بدون بلعیدن باگ‌های بی‌ربط)؛ سلسله‌مراتب، سطح‌بندی مدیریت خطا می‌دهد (پایه‌ی دامنه → شاخه‌های ریز)؛ و نام استثنا خودش مستندات است. `except Exception` در لایه‌های میانی یعنی «هر باگی را قورت بده» — همان که ساعت‌ها دیباگ را می‌سوزاند.

**۲. «raise ... from ... چه فرقی با raise خالی دارد؟»** — `from e` زنجیره‌ی علت را صریح نگه می‌دارد («این خطای دامنه، به‌خاطر آن خطای فنی رخ داد») و در traceback با «direct cause» می‌آید؛ `from None` زنجیره‌ی بی‌ربط را قطع می‌کند. بدون این‌ها traceback یا گمراه‌کننده است یا نشت‌کننده‌ی جزئیات پیاده‌سازی.

**۳. «چرا pickle برای داده‌ی نامطمئن ممنوع است؟»** — بازکردن pickle می‌تواند کد دلخواه اجرا کند (سازوکار `__reduce__`)؛ یعنی «فایل داده» عملاً «برنامه‌ی مهمان» است. برای تبادل با بیرون: JSON (یا فرمت‌های schema-دار)؛ pickle فقط برای داده‌ی داخلی خودی و هم‌نسخه.

**۴. «با getattr/hasattr چه کار واقعی‌ای کرده‌ای؟»** — انتظار مصاحبه‌گر یک مورد عملی است، مثلاً: dispatch پویا (`getattr(handler, f"handle_{event}")`)، پلاگین‌هایی که وجود متد اختیاری‌شان چک می‌شود، یا سریال‌سازی عمومی با پیمایش `__dict__`. اشاره به خطرش هم بلوغ است: introspection زیاد، کد را «جادویی» و رهگیری‌اش را سخت می‌کند.

---

**در فصل بعد:** همه‌ی ابزارها را داریم؛ حالا وقت چیدنشان در قالب‌های آزموده است: **الگوهای طراحی** — راه‌حل‌های تکرارشونده‌ای که مهندسان طی دهه‌ها برای مسئله‌های آشنا پیدا کرده‌اند، این‌بار به سبک پایتونی.
