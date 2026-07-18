# تمرین‌های فصل پنجم: Property، دکوراتورهای کلاسی و متدهای جادویی

> این فصل ابزارهایی را پوشش می‌دهد که «پایتون را پایتون می‌کنند». ابتدا خودتان پاسخ دهید، سپس با `05-property-decorators-magic-methods-answer.md` مقایسه کنید.

---

## بخش الف: Property

**۱.** مشکل نوشتن getter/setter کلاسیک (به‌سبک جاوا) در پایتون چیست؟ چرا `t.get_celsius()` «غیرپایتونی» شمرده می‌شود؟

**۲.** مهم‌ترین مزیت `@property` نسبت به getter/setter چیست؟ سناریویی را توضیح دهید که در آن، ارتقای یک attribute ساده به property، **بدون تغییر حتی یک خط از کد کاربر** انجام می‌شود.

**۳.** چگونه یک property فقط‌خواندنی (read-only) می‌سازیم و چه زمانی مفید است؟ در مثال `Circle.area`، چرا بهتر است `area` یک property محاسبه‌شونده باشد تا یک attribute ذخیره‌شده؟

**۴.** تفاوت `@property` و `@cached_property` چیست؟ Tradeoff استفاده از `cached_property` کدام است و برای چه نوع مقادیری مناسب است؟

---

## بخش ب: classmethod و staticmethod

**۵.** تفاوت سه نوع متد (instance method، class method، static method) را از نظر پارامتر اول (`self`/`cls`/هیچ) و اینکه به چه چیزی دسترسی دارند، توضیح دهید.

**۶.** رایج‌ترین کاربرد `@classmethod` چیست؟ در یک Factory Method مثل `Date.from_string`، چرا از `cls(...)` استفاده می‌کنیم به‌جای نوشتن مستقیم `Date(...)`؟

**۷.** `@staticmethod` چه زمانی مناسب است؟ Tradeoff آن در برابر یک تابع مستقل ساده (بیرون کلاس) چیست؟

---

## بخش ج: متدهای جادویی

**۸.** تفاوت عملگر `==` و `is` چیست؟ متد `__eq__` کدام‌یک را تعریف می‌کند؟ در کد زیر خروجی هر سه `print` چیست؟

```python
a = Point(1, 2)   # a class with __eq__ that compares by value
b = Point(1, 2)
print(a == b)
print(a is b)
print(a is a)
```

**۹.** چرا وقتی `__eq__` را تعریف می‌کنیم، معمولاً باید `__hash__` را هم تعریف کنیم؟ قاعده‌ی سازگاری بین این دو چیست (دو شیء برابر باید چه ویژگی‌ای در hash داشته باشند)؟

**۱۰.** متدهای `__enter__` و `__exit__` چه چیزی را ممکن می‌کنند؟ context manager چه **تضمینی** درباره‌ی اجرا شدن `__exit__` می‌دهد و چرا این برای مدیریت منابع (فایل، اتصال، قفل) ایده‌آل است؟

**۱۱.** کدام متدهای جادویی یک کلاس را به یک «کانتینر» تبدیل می‌کنند؟ جدول زیر را کامل کنید:

| عملگر/کاربرد | متد جادویی |
|--------------|-------------|
| `len(obj)` | ؟ |
| `obj[i]` | ؟ |
| `x in obj` | ؟ |
| `for x in obj` | ؟ |

**۱۲.** دام «حلقه‌ی بی‌نهایت» در `__setattr__` چیست؟ چرا نوشتن `self.name = value` درون `__setattr__` خطرناک است و راه درست چیست؟

---

## بخش د: کدنویسی و کدخوانی

**۱۳. (کدخوانی)** در کد زیر، جمله‌ی «در حال خواندن...» چند بار چاپ می‌شود و چرا؟

```python
from functools import cached_property

class Dataset:
    def __init__(self, path):
        self.path = path

    @cached_property
    def records(self):
        print("Reading...")
        return [1, 2, 3]

d = Dataset("data.csv")
print(d.records)
print(d.records)
```

**۱۴. (کدنویسی)** یک کلاس `Circle` بنویسید که `radius` را در سازنده بگیرد و دو property فقط‌خواندنی داشته باشد: `area` (مساحت) و `diameter` (قطر). نشان دهید با تغییر `radius`، هر دو خودکار به‌روز می‌شوند و تلاش برای `c.area = 100` خطا می‌دهد.

**۱۵. (کدنویسی)** یک کلاس `Date` بنویسید که سازنده‌ی معمولی (`year, month, day`) داشته باشد و یک `@classmethod` به نام `from_string` که رشته‌ی `"2026-07-01"` را بگیرد و یک `Date` بسازد. یک `__repr__` مناسب هم اضافه کنید.

**۱۶. (کدنویسی)** یک کلاس `Playlist` بنویسید که آهنگ‌ها را نگه دارد و مثل یک لیست داخلی پایتون رفتار کند: از `len()`، دسترسی با `[]`، عملگر `in`، و حلقه‌ی `for` پشتیبانی کند.

**۱۷. (کدنویسی)** یک context manager به نام `Timer` بنویسید که با `with` کار کند و هنگام خروج، مدت‌زمان سپری‌شده در بلوک را چاپ کند. (راهنمایی: از `time.perf_counter()` در `__enter__` و `__exit__` استفاده کنید.)

---

## بخش ه: پرسش تحلیلی

**۱۸.** این جمله را نقد کنید: «هر کلاسی باید تا حد ممکن متدهای جادویی زیادی پیاده کند تا حرفه‌ای‌تر و کامل‌تر به‌نظر برسد.»

**۱۹.** نویسنده توصیه می‌کند «با attribute ساده شروع کن و فقط هنگام نیاز به property ارتقا بده» (اصل YAGNI). این توصیه چه تفاوتی با عادت «همیشه از ابتدا getter/setter بنویس» در جاوا دارد، و چرا این رویکرد در پایتون منطقی است؟

---

## بخش و: عملگرها و جنریتورها

**۲۰.** تفاوت `return NotImplemented` با `raise NotImplementedError` چیست؟ «مذاکره‌ی دوطرفه»ی پایتون هنگام `a + b` را توضیح دهید و بگویید `__radd__` دقیقاً کجای این مذاکره وارد می‌شود.

**۲۱.** با توجه به کلاس زیر، خروجی (یا خطای) هر سه خط آخر را پیش‌بینی کنید:

```python
class Money:
    def __init__(self, amount):
        self.amount = amount
    def __repr__(self):
        return f"Money({self.amount})"
    def __add__(self, other):
        if isinstance(other, Money):
            return Money(self.amount + other.amount)
        if isinstance(other, int):
            return Money(self.amount + other)
        return NotImplemented

print(Money(2_000) + 500)
print(500 + Money(2_000))
print(sum([Money(100), Money(200)], Money(0)))
```

**۲۲. (کدنویسی)** کلاس `Duration` (مدت‌زمان بر حسب ثانیه) بنویسید با این قابلیت‌ها: جمع دو `Duration`، جمع با `int` (ثانیه) از **هر دو طرف**، مقایسه‌ی برابری، و `__repr__` خوانا مثل `Duration(90s)`. رفتارش را با `sum` روی یک لیست هم نشان دهید.

**۲۳. (کدنویسی)** کلاس iterator زیر را به یک **جنریتور** سه‌چهارخطی تبدیل کنید؛ سپس در کلاس `Inventory` (که دیکشنری `name -> stock` دارد) متد `__iter__` را با جنریتور طوری بنویسید که فقط نام کالاهای موجود (`stock > 0`) را بدهد:

```python
class EvenNumbers:
    def __init__(self, limit):
        self.limit = limit
        self.current = 0
    def __iter__(self):
        return self
    def __next__(self):
        if self.current > self.limit:
            raise StopIteration
        value = self.current
        self.current += 2
        return value
```

**۲۴. (تفکر انتقادی)** همکاری می‌گوید: «حالا که عملگرها را یاد گرفتیم، برای همه‌ی کلاس‌هایمان `__add__` و `__lt__` تعریف کنیم؛ کد کوتاه‌تر می‌شود.» این توصیه را نقد کنید: معیار درست تعریف عملگر چیست و عملگر مبهم چرا از متد بدنام خطرناک‌تر است؟ برای `User + User` دست‌کم دو تفسیر متناقض مثال بزنید.
