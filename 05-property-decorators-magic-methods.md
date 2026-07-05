# فصل پنجم: Property، دکوراتورهای کلاسی و متدهای جادویی

---

## مقدمه

تا اینجا اصول بنیادین شیءگرایی را ساختیم. حالا سراغ ابزارهایی می‌رویم که پایتون را **پایتون** می‌کنند — قابلیت‌هایی که در بسیاری از زبان‌های دیگر وجود ندارند و کد شما را از «درست» به «حرفه‌ای» ارتقا می‌دهند.

سه دسته ابزار در این فصل: **Property** (کنترل هوشمندِ دسترسی به attribute)، **دکوراتورهای کلاسی** (`@classmethod` و `@staticmethod`)، و **متدهای جادویی** (که به کلاس شما اجازه می‌دهند با عملگرها و ساختارهای داخلی پایتون یکپارچه شود). این‌ها همان چیزهایی‌اند که وقتی کد جنگو یا SQLAlchemy را می‌خوانید، همه‌جا می‌بینید.

---

## چرا این ابزارها را یاد بگیریم؟

بدون این‌ها هم می‌شود کد نوشت، اما کدِ ناشیانه‌تر و پرتکرارتر. با آن‌ها:

- می‌توانید دسترسی به داده را کنترل کنید بی‌آنکه کدِ کاربر تغییر کند (Property)
- اشیاء را با منطق‌های مختلف بسازید (Factory با `@classmethod`)
- کلاس‌هایتان با حلقه‌ی `for`، عملگر `==`، عبارت `with` و... یکپارچه شوند (متدهای جادویی)
- کدی بنویسید که با فریمورک‌های مدرن پایتون هماهنگ باشد

---

## Property — کنترل هوشمند دسترسی

### مشکل: Getter و Setter کلاسیک

در جاوا رسم است که attributeها را private کنید و برایشان `getX()` و `setX()` بنویسید. یک تازه‌واردِ از جاوا آمده، در پایتون هم همین می‌کند:

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius

    def get_celsius(self):
        return self._celsius

    def set_celsius(self, value):
        self._celsius = value

t = Temperature(25)
print(t.get_celsius())    # 25 — verbose and non-Pythonic
t.set_celsius(30)
```

این پایتونی نیست. کاربر باید `t.get_celsius()` بنویسد به‌جای `t.celsius`. پایتون راه بهتری دارد.

### راه‌حل: @property

`@property` به شما اجازه می‌دهد یک متد را **مثل یک attribute ساده** در دسترس بگذارید، اما پشتش منطق داشته باشید:

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius

    @property
    def celsius(self):               # read like an attribute
        return self._celsius

    @celsius.setter
    def celsius(self, value):        # assigned like an attribute
        if value < -273.15:
            raise ValueError("temperature below absolute zero is impossible")
        self._celsius = value

t = Temperature(25)
print(t.celsius)      # 25 — no parentheses! like an attribute
t.celsius = 30        # the setter is called automatically
# t.celsius = -300    # ValueError — validation works
```

زیبایی کار: کاربر با `t.celsius` کار می‌کند انگار یک attribute ساده است، اما شما در پشت‌صحنه کنترل کامل دارید. این همان **انتزاع** فصل قبل است، به شکل عملی.

### مزیت بزرگ: تغییر بدون شکستن کد کاربر

فرض کنید ابتدا `celsius` یک attribute ساده بود:

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius        # simple attribute

# the user's code everywhere:  t.celsius = 30
```

بعداً می‌فهمید باید اعتبارسنجی اضافه کنید. در جاوا باید همه‌جا `t.celsius` را به `t.setCelsius()` تغییر می‌دادید. در پایتون فقط `celsius` را به property تبدیل می‌کنید و **هیچ خطی از کدِ کاربر تغییر نمی‌کند**. این چراییِ اصلیِ ترجیح `@property` بر getter/setter در پایتون است: تا وقتی نیازش نیست، attribute ساده بگذارید؛ هر وقت لازم شد، بدون شکستن چیزی، به property ارتقا دهید.

### Property فقط‌خواندنی (Read-Only)

اگر فقط getter تعریف کنید و setter نگذارید، attribute فقط‌خواندنی می‌شود — کاربردی برای مقدارهای محاسبه‌شونده:

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):                  # read-only, computed
        return 3.14159 * self.radius ** 2

c = Circle(5)
print(c.area)      # 78.53975 — always up to date
c.radius = 10
print(c.area)      # 314.159 — recomputed automatically
# c.area = 100     # AttributeError: can't set attribute
```

`area` داده‌ی ذخیره‌شده نیست؛ هر بار از `radius` محاسبه می‌شود. پس همیشه سازگار است و نمی‌شود به‌اشتباه مقدار نامعتبر برایش گذاشت.

### Lazy Loading و cached_property

گاهی محاسبه‌ی یک مقدار **گران** است و نمی‌خواهید مگر لازم شود انجامش دهید. با `functools.cached_property` مقدار در اولین دسترسی محاسبه و سپس **ذخیره** می‌شود:

```python
from functools import cached_property

class Dataset:
    def __init__(self, path):
        self.path = path

    @cached_property
    def records(self):
        print("Reading large file...")   # runs only once
        # assume this operation is heavy
        return [1, 2, 3]

d = Dataset("data.csv")
print(d.records)   # Reading large file... \n [1, 2, 3]
print(d.records)   # [1, 2, 3] — the second time the file wasn't read, it came from cache
```

تفاوت با `@property`: `@property` **هر بار** بازمحاسبه می‌کند؛ `@cached_property` فقط بار اول، بعد نتیجه را نگه می‌دارد. **Tradeoff:** `cached_property` حافظه مصرف می‌کند و اگر داده‌ی زیربنایی تغییر کند، کش کهنه می‌شود. برای مقدارهای گران و ثابت مناسب است، نه مقدارهایی که مدام عوض می‌شوند.

### چه زمانی property و چه زمانی attribute ساده؟

قاعده‌ی پایتونی: **با attribute ساده شروع کنید.** فقط وقتی به اعتبارسنجی، محاسبه، یا کنترلِ دسترسی نیاز پیدا کردید، به property ارتقا دهید. property نساختن پیش از نیاز، خودش یک اصل است (YAGNI — «لازمش نخواهی داشت»).

---

## دکوراتورهای کلاسی: classmethod و staticmethod

هر متدی که تا حالا دیدیم یک **instance method** بود — `self` می‌گرفت و روی یک شیء کار می‌کرد. اما دو نوع متد دیگر هم هست.

### تفاوت سه نوع متد

```python
class Example:
    class_var = "shared"

    def instance_method(self):       # has access to the object (self)
        return self

    @classmethod
    def class_method(cls):           # has access to the class (cls), not a specific object
        return cls

    @staticmethod
    def static_method():             # neither self nor cls — just a function in the class namespace
        return "independent"
```

- **Instance method** (`self`): روی یک شیءِ خاص کار می‌کند و به داده‌اش دسترسی دارد.
- **Class method** (`cls`): روی خودِ کلاس کار می‌کند، نه یک شیء. `cls` همان کلاس است.
- **Static method** (هیچ‌کدام): فقط یک تابعِ کمکی است که منطقاً به کلاس تعلق دارد اما نه به داده‌ی شیء نیاز دارد نه به داده‌ی کلاس.

### classmethod به‌عنوان Factory Method

پرکاربردترین استفاده‌ی `@classmethod`، ساختِ شیء با منطق‌های جایگزین است. سازنده‌ی `__init__` یکی است، اما گاهی می‌خواهید از راه‌های مختلف شیء بسازید:

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, text):          # Factory: build from a string
        year, month, day = map(int, text.split("-"))
        return cls(year, month, day)     # cls means Date (or its subclass)

    @classmethod
    def today(cls):                      # Factory: today's date
        import datetime
        n = datetime.date.today()
        return cls(n.year, n.month, n.day)

    def __repr__(self):
        return f"Date({self.year}, {self.month}, {self.day})"

d1 = Date(2026, 7, 1)                    # ordinary constructor
d2 = Date.from_string("2026-07-01")      # Factory from a string
d3 = Date.today()                        # Factory from today
print(d1, d2)                            # Date(2026, 7, 1) Date(2026, 7, 1)
```

چرا `cls` به‌جای نوشتن مستقیم `Date`؟ چون اگر کسی از `Date` ارث بگیرد، `from_string` روی زیرکلاس هم درست کار می‌کند و نمونه‌ی زیرکلاس می‌سازد، نه `Date`. این انعطاف، مزیتِ `cls` است.

### staticmethod: توابع کمکیِ مرتبط

`@staticmethod` برای وقتی است که تابعی منطقاً به کلاس مربوط است اما نه به `self` نیاز دارد نه به `cls`:

```python
class Validator:
    @staticmethod
    def is_valid_email(text):
        return "@" in text and "." in text

# usable without creating an object
print(Validator.is_valid_email("a@b.com"))   # True
```

می‌شد این را یک تابع مستقل هم نوشت. پس چرا static method؟ برای **سازمان‌دهی**: وقتی این تابع به‌شدت با `Validator` مرتبط است، قراردادنش داخل کلاس، خواناتر و یکپارچه‌تر است. **Tradeoff:** اگر رابطه‌ی قوی‌ای با کلاس ندارد، تابع مستقلِ ساده اغلب بهتر است.

---

## متدهای جادویی (Dunder Methods)

متدهای جادویی — با نام‌های `__something__` — قلاب‌هایی هستند که پایتون در موقعیت‌های خاص خودکار صدایشان می‌زند. با پیاده‌سازیِ آن‌ها، کلاس شما با زبان یکپارچه می‌شود.

### __str__، __repr__ و __format__

`__str__` و `__repr__` را در فصل قبل دیدیم. `__format__` کنترل می‌کند وقتی شیء در f-string با مشخصه‌ی قالب می‌آید چه شود:

```python
class Money:
    def __init__(self, amount):
        self.amount = amount

    def __format__(self, spec):
        if spec == "long":
            return f"{self.amount:,} Toman"
        return str(self.amount)

m = Money(1500000)
print(f"{m:long}")   # 1,500,000 Toman
print(f"{m}")        # 1500000
```

### __eq__ و __hash__

به‌طور پیش‌فرض، دو شیء فقط وقتی برابرند که **یک شیء واحد** باشند (هویت). `__eq__` اجازه می‌دهد برابری را بر اساس **مقدار** تعریف کنید:

```python
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __hash__(self):
        return hash((self.x, self.y))

a = Point(1, 2)
b = Point(1, 2)
print(a == b)      # True — because of __eq__ (same value)
print(a is b)      # False — two separate objects in memory
print(a is a)      # True
```

تفاوت `==` و `is`: `==` برابریِ **مقدار** را می‌سنجد (که `__eq__` تعریفش می‌کند)؛ `is` برابریِ **هویت** را (آیا دقیقاً همان شیء در حافظه است).

چرا `__hash__` هم لازم شد؟ چون به‌محضِ تعریف `__eq__`، پایتون شیء را غیرقابل‌hash می‌کند و نمی‌توانید در `set` یا کلید `dict` استفاده‌اش کنید. اگر می‌خواهید شیء در مجموعه یا دیکشنری برود، باید `__hash__` سازگار با `__eq__` تعریف کنید (دو شیء برابر باید hash یکسان داشته باشند):

```python
points = {Point(1, 2), Point(1, 2)}
print(len(points))   # 1 — because they are equal and have the same hash
```

### __bool__

تعیین می‌کند شیء در شرط‌ها True است یا False:

```python
class Cart:
    def __init__(self):
        self.items = []

    def __bool__(self):
        return len(self.items) > 0

cart = Cart()
if not cart:                     # __bool__ is called
    print("cart is empty")       # is printed
```

### کلاس به‌مثابه‌ی Container

مجموعه‌ای از متدهای جادویی، کلاس شما را به یک **کانتینر** (قابل ایندکس، پیمایش، و پشتیبانی از `in`) تبدیل می‌کنند:

- `__getitem__` / `__setitem__`: دسترسی با `[]`
- `__len__`: پشتیبانی از `len()`
- `__contains__`: پشتیبانی از عملگر `in`
- `__iter__`: پشتیبانی از حلقه‌ی `for`

```python
class Playlist:
    def __init__(self):
        self._songs = []

    def add(self, song):
        self._songs.append(song)

    def __getitem__(self, index):
        return self._songs[index]

    def __len__(self):
        return len(self._songs)

    def __contains__(self, song):
        return song in self._songs

    def __iter__(self):
        return iter(self._songs)

p = Playlist()
p.add("Song 1")
p.add("Song 2")

print(len(p))              # 2         ← __len__
print(p[0])                # Song 1     ← __getitem__
print("Song 2" in p)       # True      ← __contains__
for song in p:             # ← __iter__
    print(song)
```

با این چند متد، `Playlist` طوری رفتار می‌کند که انگار یک لیستِ داخلیِ پایتون است — این اوجِ یکپارچگی با زبان است.

### پروتکل Iterator: __iter__ و __next__

`__iter__` به‌تنهایی می‌تواند یک iterator داخلی برگرداند (مثل بالا). اما اگر بخواهید منطق پیمایشِ سفارشی داشته باشید، از `__next__` هم استفاده می‌کنید:

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self                  # the object itself is the iterator

    def __next__(self):
        if self.current <= 0:
            raise StopIteration      # end of iteration
        self.current -= 1
        return self.current + 1

for n in Countdown(3):
    print(n)     # 3, 2, 1
```

تفaوت با `__getitem__`: `__getitem__` برای دسترسیِ ایندکسی است؛ پروتکل iterator (`__iter__`/`__next__`) برای پیمایشِ کنترل‌شده و بالقوه بی‌پایان یا محاسبه‌شونده مناسب‌تر است.

### Context Manager: __enter__ و __exit__

این دو متد اجازه می‌دهند کلاس شما با عبارت `with` کار کند — الگوی استانداردِ مدیریت منابع (فایل، اتصال دیتابیس، قفل):

```python
class DatabaseConnection:
    def __enter__(self):
        print("connection opened")
        return self                  # what is given to `as`

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("connection closed")       # runs even if an error occurs
        return False                 # False means do not suppress the exception

    def query(self, sql):
        return f"Result: {sql}"

with DatabaseConnection() as db:
    print(db.query("SELECT * FROM users"))
# Output:
# connection opened
# Result: SELECT * FROM users
# connection closed   ← guaranteed, even on error
```

قدرتِ context manager این است که `__exit__` **همیشه** اجرا می‌شود — چه بلوک عادی تمام شود چه با استثنا. برای همین برای آزادکردنِ مطمئنِ منابع ایده‌آل است.

### __call__ در عمل

`__call__` را در فصل قبل دیدیم. یک کاربرد واقعی‌اش، ماشین حالت است:

```python
class RateLimiter:
    def __init__(self, max_calls):
        self.max_calls = max_calls
        self.calls = 0

    def __call__(self):
        self.calls += 1
        if self.calls > self.max_calls:
            return "blocked"
        return "allowed"

limit = RateLimiter(2)
print(limit())   # allowed
print(limit())   # allowed
print(limit())   # blocked — the object kept its state between calls
```

### __getattr__ و __setattr__ (پیشرفته)

- `__getattr__`: فقط وقتی صدا زده می‌شود که attribute به‌روشِ معمول **پیدا نشود**. برای مقادیر پیش‌فرض یا پروکسی مفید است.
- `__getattribute__`: برای **هر** دسترسی صدا زده می‌شود (خطرناک‌تر، به‌ندرت لازم).
- `__setattr__`: هنگام هر مقداردهیِ attribute. اینجا باید مراقب **حلقه‌ی بی‌نهایت** بود.

```python
class SafeConfig:
    def __getattr__(self, name):
        # only for missing attributes
        return f"<{name} not defined>"

    def __setattr__(self, name, value):
        # infinite-loop trap! you must not write self.name = value
        # because it would call __setattr__ again
        super().__setattr__(name, value)   # the right way

c = SafeConfig()
c.host = "localhost"
print(c.host)      # localhost
print(c.missing)   # <missing not defined>  ← __getattr__
```

نکته‌ی حیاتی: درون `__setattr__` هرگز مستقیم `self.x = ...` ننویسید؛ آن، دوباره `__setattr__` را صدا می‌زند و حلقه‌ی بی‌نهایت می‌سازد. همیشه از `super().__setattr__(...)` استفاده کنید.

### Descriptor Protocol (پیش‌درآمد)

`__get__`، `__set__` و `__delete__` هسته‌ی **Descriptor** هستند — مکانیزمی که خودِ `@property`، `@classmethod` و حتی متدها روی آن ساخته شده‌اند. Descriptor به شما اجازه می‌دهد منطقِ دسترسی به attribute را در یک کلاسِ جداگانه و **قابل‌استفاده‌ی مجدد** بگذارید:

```python
class Positive:
    def __set_name__(self, owner, name):
        self._name = "_" + name

    def __get__(self, obj, objtype=None):
        return getattr(obj, self._name)

    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError("must be positive")
        setattr(obj, self._name, value)

class Product:
    price = Positive()      # validation logic, reusable
    weight = Positive()

    def __init__(self, price, weight):
        self.price = price
        self.weight = weight

p = Product(100, 5)
# Product(-1, 5)   # ValueError: must be positive
```

مزیت نسبت به property: منطق `Positive` یک‌بار نوشته شد و روی `price` و `weight` (و هر تعدادِ دیگر) اعمال شد، بی‌تکرار. Descriptorها پیشرفته‌اند و در فصل سیزدهم (متاکلاس‌ها) دوباره سراغشان می‌رویم.

---

## خلاصه

| ابزار | کاربرد اصلی |
|-------|-------------|
| `@property` | دسترسی attribute-مانند با منطق پشت آن |
| `@cached_property` | محاسبه‌ی گران، فقط یک‌بار |
| `@classmethod` | Factory Method؛ ساخت شیء با منطق جایگزین (`cls`) |
| `@staticmethod` | تابع کمکیِ مرتبط با کلاس، بدون `self`/`cls` |
| `__str__`/`__repr__` | نمایش برای کاربر / برنامه‌نویس |
| `__eq__`/`__hash__` | برابری بر اساس مقدار، عضویت در set/dict |
| `__getitem__`/`__len__`/`__iter__` | تبدیل کلاس به کانتینر |
| `__enter__`/`__exit__` | پشتیبانی از `with` (مدیریت منابع) |
| `__call__` | شیءِ قابل‌فراخوانی مثل تابع |
| Descriptor | منطق دسترسیِ قابل‌استفاده‌ی مجدد |

```
متدهای جادویی = پل بین کلاس شما و زبان پایتون
═══════════════════════════════════════════════
  print(obj)   ──►  __str__
  obj == other ──►  __eq__
  len(obj)     ──►  __len__
  for x in obj ──►  __iter__
  obj[i]       ──►  __getitem__
  with obj     ──►  __enter__ / __exit__
  obj()        ──►  __call__
  x in obj     ──►  __contains__
```

**نکته‌ی طلایی:** متدهای جادویی وسوسه‌انگیزند، اما همه را لازم ندارید. فقط آن‌هایی را پیاده کنید که واقعاً به یکپارچگیِ کلاس با زبان کمک می‌کنند. یک کلاسِ ساده که فقط `__repr__` دارد، بهتر از کلاسی است که ده متد جادویی دارد که هیچ‌کدام لازم نبوده‌اند.

---

**در فصل بعد:** به سراغ ارث‌بریِ پیشرفته می‌رویم — وراثت چندگانه، الگوریتم C3 برای MRO، و Mixinها — تا بفهمیم پایتون چطور سلسله‌مراتب‌های پیچیده را مدیریت می‌کند.
