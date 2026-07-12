# پاسخ تمرین‌های فصل پنجم: Property، دکوراتورهای کلاسی و متدهای جادویی

> پاسخ‌ها به‌ترتیبِ پرسش‌های `05-property-decorators-magic-methods-exercise.md`. هرجا راهِ دیگری هم بود، بخشِ **راهِ دیگر** آورده شده است.

---

## بخش الف: Property

**۱.** در جاوا رسم است attribute را private کنید و `getX()`/`setX()` بنویسید. تازه‌واردی که از جاوا آمده همین را در پایتون هم می‌کند (`get_celsius`/`set_celsius`). مشکل: کاربر مجبور است `t.get_celsius()` بنویسد به‌جای `t.celsius`ِ ساده و خوانا. این «پرحرف» و غیرپایتونی است، چون پایتون راهِ بهتری دارد (`@property`) که هم رابطِ ساده می‌دهد و هم کنترل.

**۲.** مهم‌ترین مزیت: **می‌توانید بدونِ شکستنِ کدِ کاربر، منطق اضافه کنید.** سناریو: ابتدا `celsius` یک attribute ساده است و کدِ کاربر همه‌جا `t.celsius = 30` نوشته. بعداً می‌فهمید باید اعتبارسنجی اضافه کنید. در جاوا مجبور بودید همه‌ی `t.celsius`ها را به `t.setCelsius()` تغییر دهید. در پایتون فقط `celsius` را به property تبدیل می‌کنید و **هیچ خطی از کدِ کاربر عوض نمی‌شود** — چون `t.celsius = 30` خودکار setter را صدا می‌زند.

**۳.** اگر فقط getter تعریف کنید و setter نگذارید، property فقط‌خواندنی می‌شود:

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return 3.14159 * self.radius ** 2
```

چرا `area` بهتر است property باشد تا attribute ذخیره‌شده؟ چون `area` وابسته به `radius` است. اگر آن را ذخیره کنیم، با تغییرِ `radius` مقدارِ ذخیره‌شده کهنه می‌شود و ممکن است ناسازگار بماند. به‌صورتِ property، **هر بار از روی `radius` محاسبه می‌شود**، پس همیشه به‌روز و سازگار است و نمی‌شود به‌اشتباه مقدارِ نامعتبر برایش گذاشت.

**۴.**
- `@property` **هر بار** که دسترسی می‌گیرید، دوباره محاسبه می‌کند.
- `@cached_property` فقط **بارِ اول** محاسبه می‌کند، سپس نتیجه را ذخیره (کش) می‌کند و دفعاتِ بعد همان را برمی‌گرداند.

**Tradeoff:** `cached_property` حافظه مصرف می‌کند و اگر داده‌ی زیربنایی تغییر کند، کش **کهنه** می‌شود (مقدارِ قدیمی را می‌دهد). برای مقادیرِ **گران و ثابت** مناسب است (مثلِ خواندنِ یک فایلِ بزرگ که تغییر نمی‌کند)، نه مقادیری که مدام عوض می‌شوند.

---

## بخش ب: classmethod و staticmethod

**۵.**
- **Instance method** (`self`): روی یک شیءِ خاص کار می‌کند و به داده‌ی آن شیء دسترسی دارد.
- **Class method** (`cls`): روی خودِ کلاس کار می‌کند، نه یک شیءِ خاص؛ `cls` همان کلاس است. به داده‌ی کلاس دسترسی دارد.
- **Static method** (نه `self` نه `cls`): فقط یک تابعِ کمکی است که منطقاً به کلاس تعلق دارد، اما نه به داده‌ی شیء نیاز دارد نه به داده‌ی کلاس.

**۶.** رایج‌ترین کاربردِ `@classmethod`، **Factory Method** است: ساختِ شیء از راه‌های جایگزین (علاوه بر `__init__`). چرا `cls(...)` به‌جای `Date(...)`؟ چون اگر کسی از `Date` ارث بگیرد (مثلاً `class PersianDate(Date)`)، آنگاه `PersianDate.from_string(...)` به‌درستی یک `PersianDate` می‌سازد، نه یک `Date`. `cls` همان کلاسی است که متد روی آن صدا زده شده، پس کد نسبت به ارث‌بری انعطاف‌پذیر می‌ماند.

**۷.** `@staticmethod` وقتی مناسب است که تابعی منطقاً به کلاس مربوط است اما نه `self` می‌خواهد نه `cls` (مثلِ یک اعتبارسنجِ ایمیل درونِ کلاسِ `Validator`). **Tradeoff:** می‌شد آن را یک تابعِ مستقل هم نوشت. static method فقط برای **سازمان‌دهی و خوانایی** است — وقتی رابطه‌ی قوی‌ای با کلاس دارد، داخلِ کلاس گذاشتنش یکپارچه‌تر است. اگر رابطه‌ی قوی ندارد، تابعِ مستقلِ ساده اغلب بهتر است (پیچیدگیِ کمتر).

---

## بخش ج: متدهای جادویی

**۸.** خروجی:

```
True
False
True
```

- `==` برابریِ **مقدار** را می‌سنجد، که `__eq__` تعریفش می‌کند؛ چون `a` و `b` مختصاتِ یکسان دارند، `a == b` برابرِ `True` است.
- `is` برابریِ **هویت** را می‌سنجد (آیا دقیقاً همان شیء در حافظه است)؛ `a` و `b` دو شیءِ جدا هستند، پس `a is b` برابرِ `False`، اما `a is a` برابرِ `True`.

`__eq__` عملگرِ `==` را تعریف می‌کند، نه `is` (که همیشه هویت را می‌سنجد و قابلِ override نیست).

**۹.** به‌محضِ اینکه `__eq__` را تعریف می‌کنید، پایتون شیء را **غیرقابلِ hash** می‌کند و دیگر نمی‌توانید آن را در `set` یا کلیدِ `dict` بگذارید. برای اینکه دوباره hashable شود، باید `__hash__` سازگار تعریف کنید. **قاعده‌ی سازگاری:** دو شیئی که برابرند (`a == b`) باید **hash یکسان** داشته باشند (`hash(a) == hash(b)`). راهِ ساده: hash را از همان فیلدهایی بسازید که در `__eq__` مقایسه می‌کنید، مثلاً `return hash((self.x, self.y))`. (عکسِ آن لازم نیست: دو شیء با hash یکسان می‌توانند نابرابر باشند.)

**۱۰.** `__enter__` و `__exit__` اجازه می‌دهند کلاس با عبارتِ `with` کار کند. **تضمین:** `__exit__` **همیشه** اجرا می‌شود — چه بلوک عادی تمام شود، چه با یک استثنا نیمه‌کاره رها شود. برای همین برای مدیریتِ منابع ایده‌آل است: منبع (فایل/اتصال/قفل) را در `__enter__` می‌گیرید و در `__exit__` آزاد می‌کنید، و مطمئن‌اید حتی در صورتِ خطا هم منبع آزاد می‌شود و نشتی رخ نمی‌دهد.

**۱۱.**

| عملگر/کاربرد | متدِ جادویی |
|--------------|-------------|
| `len(obj)` | `__len__` |
| `obj[i]` | `__getitem__` |
| `x in obj` | `__contains__` |
| `for x in obj` | `__iter__` |

**۱۲.** دام این است: اگر درونِ `__setattr__` بنویسید `self.name = value`، آن خط **دوباره** `__setattr__` را صدا می‌زند، که باز `self.name = value` را اجرا می‌کند، و همین‌طور تا بی‌نهایت (یا خطای `RecursionError`). راهِ درست: از پیاده‌سازیِ والد استفاده کنید تا مقداردهیِ واقعی انجام شود بی‌آنکه دوباره `__setattr__`ِ خودتان صدا زده شود:

```python
def __setattr__(self, name, value):
    super().__setattr__(name, value)   # the actual assignment, without a loop
```

---

## بخش د: کدنویسی و کدخوانی

**۱۳.** فقط **یک بار**. `@cached_property` مقدار را در اولین دسترسی محاسبه و ذخیره می‌کند؛ در دسترسیِ دوم، بدنه‌ی متد اصلاً اجرا نمی‌شود و نتیجه از کش می‌آید. پس خروجی:

```
در حال خواندن...
[1, 2, 3]
[1, 2, 3]
```

(اگر به‌جای `cached_property` از `property` معمولی استفاده می‌کردید، «در حال خواندن...» **دو بار** چاپ می‌شد.)

**۱۴.**

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return 3.14159 * self.radius ** 2

    @property
    def diameter(self):
        return self.radius * 2

c = Circle(5)
print(c.area)       # 78.53975
print(c.diameter)   # 10
c.radius = 10
print(c.area)       # 314.159 — updated automatically
print(c.diameter)   # 20
# c.area = 100      # AttributeError: property 'area' has no setter
```

چون هیچ setter تعریف نکردیم، هر دو فقط‌خواندنی‌اند و تلاش برای مقداردهی خطا می‌دهد.

**۱۵.**

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, text):
        year, month, day = map(int, text.split("-"))
        return cls(year, month, day)

    def __repr__(self):
        return f"Date({self.year}, {self.month}, {self.day})"

d1 = Date(2026, 7, 1)
d2 = Date.from_string("2026-07-01")
print(d1)   # Date(2026, 7, 1)
print(d2)   # Date(2026, 7, 1)
```

**راهِ دیگر (اعتبارسنجی در Factory):** می‌توانید در `from_string` قبل از ساخت، ورودی را بررسی کنید (مثلاً که دقیقاً سه بخش دارد) و در صورتِ خطا `ValueError` بدهید — این کار Factory را مقاوم‌تر می‌کند.

**۱۶.**

```python
class Playlist:
    def __init__(self):
        self._songs = []

    def add(self, song):
        self._songs.append(song)

    def __len__(self):
        return len(self._songs)

    def __getitem__(self, index):
        return self._songs[index]

    def __contains__(self, song):
        return song in self._songs

    def __iter__(self):
        return iter(self._songs)

p = Playlist()
p.add("Song 1")
p.add("Song 2")
print(len(p))              # 2
print(p[0])               # Song 1
print("Song 2" in p)      # True
for song in p:
    print(song)
```

نکته: اگر فقط `__getitem__` را پیاده کنید، پایتون می‌تواند حلقه‌ی `for` و `in` را هم از روی آن بسازد؛ اما تعریفِ صریحِ `__iter__` و `__contains__` هم خواناتر است و هم کاراتر.

**۱۷.** context managerِ `Timer`:

```python
import time

class Timer:
    def __enter__(self):
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        elapsed = time.perf_counter() - self.start
        print(f"Elapsed: {elapsed:.4f} s")
        return False        # do not suppress the exception

with Timer():
    total = sum(range(1_000_000))
# Output: elapsed: 0.0XXX s
```

`return False` در `__exit__` یعنی اگر داخلِ بلوک خطایی رخ داد، آن را سرکوب نکن و بگذار بالا برود.

**راهِ دیگر (با `contextlib`):** برای context managerهای ساده می‌توان به‌جای کلاس از دکوراتورِ `@contextlib.contextmanager` و یک تابعِ generator استفاده کرد — کوتاه‌تر است، اما وقتی به حالتِ بیشتری نیاز دارید کلاس خواناتر می‌ماند:

```python
import time, contextlib

@contextlib.contextmanager
def timer():
    start = time.perf_counter()
    yield
    print(f"Time: {time.perf_counter() - start:.4f} s")
```

---

## بخش ه: پرسشِ تحلیلی

**۱۸.** این جمله **نادرست** است. پیاده‌کردنِ متدهای جادوییِ غیرضروری، کد را پیچیده‌تر و گیج‌کننده‌تر می‌کند، نه حرفه‌ای‌تر. مشکلات:
- هر متدِ جادویی یک «قرارداد» با زبان است؛ اگر `__len__` را اشتباه پیاده کنید یا `__eq__` بی‌`__hash__` بگذارید، رفتارهای پنهانِ گیج‌کننده می‌سازید.
- خواننده‌ی کد باید بفهمد چرا هر متد آنجاست؛ متدهای بی‌استفاده فقط نویز اضافه می‌کنند.

قاعده‌ی درست (نکته‌ی طلاییِ فصل): فقط متدهایی را پیاده کنید که واقعاً به یکپارچگیِ کلاس با زبان کمک می‌کنند. یک کلاسِ ساده که فقط `__repr__` دارد، بهتر از کلاسی است که ده متدِ جادوییِ لازم‌نشده دارد.

**۱۹.** در جاوا، چون تبدیلِ بعدیِ یک field عمومی به getter/setter کدِ کاربر را می‌شکند، برنامه‌نویسان **از ابتدا و همیشه** getter/setter می‌نویسند — حتی وقتی هیچ منطقی پشتشان نیست (احتیاطِ اجباری). در پایتون این احتیاط لازم نیست: چون هر وقت خواستید می‌توانید یک attribute ساده را **بدونِ شکستنِ کدِ کاربر** به property ارتقا دهید. پس منطقی است که با ساده‌ترین شکل (attribute عمومی) شروع کنید و فقط هنگامِ **نیازِ واقعی** به اعتبارسنجی/محاسبه/کنترل، property اضافه کنید. این همان اصلِ **YAGNI** («لازمش نخواهی داشت») است: پیچیدگی را تا لحظه‌ی نیاز به تعویق بیندازید. زبان به شما این آزادی را می‌دهد، پس از آن استفاده کنید.

---

## بخش و: عملگرها و جنریتورها

**۲۰.** `NotImplemented` یک **مقدارِ** ویژه است که یعنی «من این ترکیب را بلد نیستم؛ نوبتِ طرفِ مقابل»؛ `NotImplementedError` یک **استثنا**ست که اجرای برنامه را می‌شکند و مذاکره را می‌کُشد. مذاکره‌ی `a + b`: اول `type(a).__add__(a, b)` امتحان می‌شود؛ اگر `NotImplemented` برگرداند، پایتون `type(b).__radd__(b, a)` را امتحان می‌کند؛ اگر آن هم بلد نبود، تازه `TypeError` می‌آید. `__radd__` همان «شانسِ دوم» است — بدونش، `500 + money` هرگز به کلاسِ شما نمی‌رسد.

**۲۱.**

```
Money(2500)
TypeError (unsupported operand type(s) for +: 'int' and 'Money')
Money(300)
```

خطِ اول از `__add__` با `int` جواب می‌گیرد. خطِ دوم: `int.__add__` بلد نیست و چون `Money` متدِ `__radd__` **ندارد**، مذاکره شکست می‌خورد → `TypeError`. (و دقت کنید: اگر بلاک را یک‌جا اجرا کنید، برنامه همین‌جا متوقف می‌شود و به خطِ سوم اصلاً نمی‌رسد!) خطِ سوم — اگر جداگانه اجرایش کنید — کار می‌کند، چون `sum` را با مقدارِ شروعِ `Money(0)` صدا زدیم و همه‌ی جمع‌ها `Money + Money`اند. (اگر مقدارِ شروع نمی‌دادیم، `sum` از `0 + Money(100)` شروع می‌کرد و باز به همان `TypeError` می‌خوردیم.)

**۲۲.**

```python
class Duration:
    def __init__(self, seconds):
        self.seconds = seconds

    def __repr__(self):
        return f"Duration({self.seconds}s)"

    def __eq__(self, other):
        if not isinstance(other, Duration):
            return NotImplemented
        return self.seconds == other.seconds

    def __add__(self, other):
        if isinstance(other, Duration):
            return Duration(self.seconds + other.seconds)
        if isinstance(other, int):
            return Duration(self.seconds + other)
        return NotImplemented

    def __radd__(self, other):
        return self.__add__(other)

print(Duration(60) + Duration(30))     # Duration(90s)
print(Duration(60) + 15)               # Duration(75s)
print(15 + Duration(60))               # Duration(75s) — thanks to __radd__
print(Duration(45) == Duration(45))    # True
print(sum([Duration(10), Duration(20), Duration(30)]))   # Duration(60s)
```

نکته: `sum` بدونِ مقدارِ شروع از `0` آغاز می‌کند؛ `0 + Duration(10)` به‌لطفِ `__radd__` جواب می‌دهد — برای همین این‌بار به مقدارِ شروع نیاز نبود.

**۲۳.**

```python
def even_numbers(limit):
    current = 0
    while current <= limit:
        yield current
        current += 2

print(list(even_numbers(6)))    # [0, 2, 4, 6]

class Inventory:
    def __init__(self, stock):
        self._stock = stock          # name -> count

    def __iter__(self):
        for name, count in self._stock.items():
            if count > 0:
                yield name

inv = Inventory({"keyboard": 3, "mouse": 0, "pad": 7})
print(list(inv))                # ['keyboard', 'pad']
for item in inv:                # a fresh generator per iteration
    print(item)
```

**راهِ دیگر:** `__iter__` می‌توانست `(n for n, c in self._stock.items() if c > 0)` برگرداند — یک generator expression؛ برای منطقِ تک‌شرطی هم‌ارز و فشرده‌تر است.

**۲۴.** معیارِ تعریفِ عملگر «کوتاه‌شدنِ کد» نیست؛ **بدیهی‌بودنِ معنا برای خواننده** است: عملگر فقط وقتی مجاز است که در دامنه‌ی مسئله یک معنای جاافتاده و بی‌ابهام داشته باشد (`Money + Money`، `Vector * 2`، `Duration + 30`). عملگرِ مبهم از متدِ بدنام خطرناک‌تر است چون متدِ بدنام دستِ‌کم اسمی دارد که بشود درباره‌اش سوال کرد؛ اما `+` ظاهری آشنا دارد و خواننده *بدونِ شک* برداشتِ خودش را می‌کند — باگی که در review هم دیده نمی‌شود. `User + User` دستِ‌کم دو تفسیرِ متناقض دارد: «ادغامِ دو حساب» (با کدام سیاست؟) یا «ساختِ یک گروهِ دونفره»؛ حتی می‌شود تصور کرد کسی انتظارِ «جمعِ امتیازها» را داشته باشد. سه برداشت از یک علامت یعنی هیچ برداشتی قابلِ‌اعتماد نیست — اینجا متدِ صریح (`merge_accounts(a, b, policy=...)`) تنها انتخابِ درست است.
