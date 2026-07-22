# پاسخ تمرین‌های فصل پانزدهم: Descriptorها

---

## تمرین ۱ - پرسش‌های مفهومی

**۱.** descriptor کلاسی است که دست‌کم یکی از متدهای `__get__`، `__set__` یا `__delete__` را دارد. باید به‌صورت **attribute کلاس** تعریف شود (نه instance)، چون سازوکار جست‌وجوی attribute فقط برای چیزهایی که در کلاس (و MRO) پیدا می‌شوند پروتکل descriptor را فعال می‌کند؛ descriptorی که در `__dict__` یک instance گذاشته شود، یک شیء معمولی است و متدهایش هرگز خودکار صدا زده نمی‌شوند.

**۲.** `self` خود شیء descriptor است؛ `instance` شیئی که attribute رویش خوانده شده؛ `owner` کلاس صاحب. وقتی attribute از روی خود کلاس خوانده شود (`Product.price`)، `instance` برابر `None` است و پیاده‌سازی استاندارد در این حالت خود descriptor (`self`) را برمی‌گرداند — به همین دلیل `Product.price` شیء descriptor را نشان می‌دهد نه مقدار را (رفتاری که در Django هم می‌بینید).

**۳.** مشکل «descriptor نام خودش را نمی‌داند» را حل می‌کند: descriptor باید بداند داده را زیر چه کلیدی در instance ذخیره کند. پایتون هنگام **تعریف کلاس** (نه ساخت شیء) برای هر descriptor نشسته در بدنه‌ی کلاس، `__set_name__(owner, name)` را صدا می‌زند. بدون آن مجبور بودیم نام را دستی تکرار کنیم — `price = Positive("price")` — که هم زشت است و هم مستعد ناهماهنگی.

**۴.** data descriptor علاوه بر `__get__`، متد `__set__` (یا `__delete__`) هم دارد و در جست‌وجوی attribute بر `__dict__` instance **مقدم** است؛ non-data فقط `__get__` دارد و **مغلوب** `__dict__` instance است. نمونه‌های پایتونی: data → `property`، فیلد/slotها (`__slots__` هم با descriptor پیاده شده)؛ non-data → توابع/متدها، `cached_property`، `classmethod`/`staticmethod`.

**۵.** چون descriptor یک شیء **مشترک** بین همه‌ی نمونه‌های کلاس است؛ ذخیره‌ی مقدار روی `self` descriptor یعنی آخرین مقدار نوشته‌شده برای *همه‌ی* نمونه‌ها دیده شود. این دقیقاً همان دام **class variable تغییرپذیر مشترک** از فصل دوم است (لیست `Team.members`) در لباس جدید. راه درست: ذخیره در خود instance، زیر نامی که `__set_name__` ساخته.

**۶.** زنجیره: `obj.method` باعث جست‌وجوی `method` در کلاس می‌شود؛ تابع پیداشده descriptor است (تابع‌ها `__get__` دارند)، پس پایتون `function.__get__(obj, type(obj))` را صدا می‌زند که یک **bound method** برمی‌گرداند — بسته‌ای از تابع + instance. پرانتز آخر همان bound method را اجرا می‌کند و instance بسته‌شده به‌عنوان `self` به تابع می‌رود. پس `obj.method()` ≡ `type(obj).method.__get__(obj, type(obj))()`.

---

## تمرین ۲ - خواندن کد و پیش‌بینی

**۷.**

```
10
99
```

`Ten` فقط `__get__` دارد → non-data. خط اول از پله‌ی ۳ (کلاس) می‌آید: ۱۰. اما بعد از نوشتن مستقیم در `__dict__` instance، پله‌ی ۲ زودتر برنده می‌شود و ۹۹ برمی‌گردد — non-data descriptor سایه‌پذیر است.

**۸.** حالا خروجی `10` است. با افزودن `__set__`، `Ten` به data descriptor تبدیل شد و data descriptor **همیشه** بر `__dict__` instance مقدم است؛ مقدار ۹۹ که یواشکی در `__dict__` گذاشته شده هرگز دیده نمی‌شود. (دقت کنید `b.__dict__["value"] = 99` استثنا نمی‌دهد — چون مستقیم دیکشنری را دست‌کاری کردیم و از مسیر `__set__` رد نشدیم؛ فقط در *خواندن* نادیده گرفته می‌شود.)

**۹.** خروجی `999 999` است در حالی که انتظار `100 999` داریم: مقدار سفارش `a` گم شد. ریشه: `__set__` مقدار را روی `self._value` می‌نویسد — یعنی روی خود descriptor که بین `a` و `b` مشترک است. هر نوشتن جدید، قبلی را برای همه بازنویسی می‌کند. اصلاح: ذخیره روی instance با `__set_name__`:

```python
class Field:
    def __set_name__(self, owner, name):
        self.storage = "_" + name
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.storage)
    def __set__(self, instance, value):
        setattr(instance, self.storage, value)
```

**۱۰.** `print(o.pay)` چیزی شبیه `<bound method Order.pay of <...Order object...>>` چاپ می‌کند — نه رشته‌ی `"paid"` — چون بدون پرانتز فقط bound method را گرفته‌ایم. `print(o.pay())` رشته‌ی `paid` را چاپ می‌کند. `Order.pay` (از روی کلاس) یک **تابع ساده** است چون `__get__` با `instance=None` صدا شده؛ `o.pay` نسخه‌ی bound است که `self` در آن پر شده.

---

## تمرین ۳ - کدنویسی

**۱۱.**

```python
class NonEmpty:
    def __set_name__(self, owner, name):
        self.name = name
        self.storage = "_" + name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.storage)

    def __set__(self, instance, value):
        if not isinstance(value, str) or not value.strip():
            raise ValueError(f"{self.name} must be a non-empty string")
        setattr(instance, self.storage, value)

class Article:
    title = NonEmpty()
    author = NonEmpty()

    def __init__(self, title, author):
        self.title = title
        self.author = author

a = Article("Descriptors 101", "ali")
print(a.title)                  # Descriptors 101
# Article("   ", "ali")         # ValueError: title must be a non-empty string
```

**۱۲.**

```python
class InRange:
    def __init__(self, min_value, max_value):
        self.min_value = min_value
        self.max_value = max_value

    def __set_name__(self, owner, name):
        self.name = name
        self.storage = "_" + name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.storage)

    def __set__(self, instance, value):
        if not (self.min_value <= value <= self.max_value):
            raise ValueError(
                f"{self.name} must be between {self.min_value} and {self.max_value}"
            )
        setattr(instance, self.storage, value)

class Exam:
    score = InRange(0, 20)
    age = InRange(18, 99)

    def __init__(self, score, age):
        self.score = score
        self.age = age

e = Exam(18, 25)
print(e.score, e.age)     # 18 25
# e.score = 21            # ValueError: score must be between 0 and 20
```

**۱۳.**

```python
class my_cached_property:
    def __init__(self, func):
        self.func = func

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        value = self.func(instance)
        instance.__dict__[self.name] = value
        return value

class Stats:
    @my_cached_property
    def heavy(self):
        print("computing...")
        return 42

s = Stats()
print(s.heavy)    # computing...  42
print(s.heavy)    # 42  (silent — served from __dict__)
```

چرا `__set__` ممنوع؟ چون کل ترفند بر non-data بودن استوار است: بعد از نوشتن نتیجه در `__dict__` instance، پله‌ی ۲ی جست‌وجو زودتر از descriptor می‌رسد و کش «خودکار» می‌شود. اگر `__set__` اضافه کنید، به data descriptor تبدیل می‌شود که **همیشه** بر `__dict__` مقدم است — یعنی `__get__` هر بار اجرا می‌شود و هم کش از کار می‌افتد، هم محاسبه هر بار تکرار می‌شود.

**۱۴.**

```python
class Audited:
    def __set_name__(self, owner, name):
        self.name = name
        self.storage = "_" + name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.storage, None)

    def __set__(self, instance, value):
        old = getattr(instance, self.storage, None)
        if not hasattr(instance, "history"):
            instance.history = []
        instance.history.append((self.name, old, value))
        setattr(instance, self.storage, value)

class Settings:
    theme = Audited()
    language = Audited()

s = Settings()
s.theme = "dark"
s.theme = "light"
s.language = "fa"
print(s.history)
# [('theme', None, 'dark'), ('theme', 'dark', 'light'), ('language', None, 'fa')]
```

**راه دیگر:** ثبت history داخل خود descriptor با یک دیکشنری `WeakKeyDictionary` (فصل چهاردهم) — instance را کلید می‌کنید تا با مرگ شیء، تاریخچه‌اش هم آزاد شود. برای شروع، نسخه‌ی بالا خواناتر است.

---

## تمرین ۴ - تحلیل و مقایسه

**۱۵.**

| ابزار | data یا non-data؟ | چه چیزی را می‌بندد / چه رفتاری دارد؟ |
|-------|-------------------|----------------------------------------|
| تابع/متد | non-data (فقط `__get__`) | instance را می‌بندد → bound method با `self` پر |
| `property` | data (`__get__` و `__set__`/`__delete__`) | getter/setter شما را صدا می‌زند؛ سایه‌ناپذیر |
| `classmethod` | non-data | به‌جای instance، **کلاس** را می‌بندد → `cls` |
| `cached_property` | non-data (عمداً) | بار اول محاسبه و در `__dict__` می‌نویسد؛ بعد خودش دور زده می‌شود |

**۱۶.**

- الف) `@property` — یک فیلد محاسبه‌ای در یک کلاس؛ descriptor اسراف است.
- ب) descriptor سفارشی (`BoundedString`) — منطق واحد روی ده‌ها فیلد در دوازده کلاس؛ دقیقاً دردی که descriptor حل می‌کند.
- ج) `@property` — یک فیلد، یک کلاس. اگر بعداً کلاس‌های دیگر هم email خواستند، آن موقع ارتقا بدهید.
- د) descriptor + `__set_name__` — تعریف اعلانی (`StringField(...)`) زبان استاندارد کتابخانه‌سازی است؛ Django و SQLAlchemy همین‌اند.

---

## تمرین ۵ - پرسش تحلیلی

**۱۷.** حق نسبی‌اش: برای **یک** کلاس با اعتبارسنجی فقط-هنگام-ساخت، متد `validate()` در `__init__` ساده‌تر و برای خواننده‌ی تازه‌کار شفاف‌تر است — پیچیدگی descriptor آنجا خرج بی‌دلیل است. سوراخ‌های راه‌حلش: اول، **تغییر بعد از ساخت** را نمی‌پوشاند — `order.total = -50` بعد از `__init__` بدون هیچ مانعی رد می‌شود، مگر setter/property هم اضافه کند که همان مسیر descriptor است با تکرار بیشتر. دوم، **تکرار منطق**: با ده کلاس و چهل فیلد، `validate()`ها پر از کپی همان چند قانون می‌شوند؛ descriptor منطق را یک‌بار می‌نویسد و اعلانی وصل می‌کند. سوم، اصل «**همه‌ی مسیرها از یک در**»: اعتبارسنجی متمرکز در `__set__` تضمین می‌کند سازنده، تغییر مستقیم، و کد آینده‌ای که هنوز نوشته نشده، همه از همان قانون رد شوند — `validate()` فقط مسیرهایی را می‌پوشاند که نویسنده یادش بوده صدایش بزند. جمع‌بندی: برای مقیاس کوچک حق با اوست؛ برای منطق تکرارشونده یا کتابخانه، descriptor نه پیچیده‌بازی بلکه حذف تکرار و بستن درهای فراموش‌شده است.
