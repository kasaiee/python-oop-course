# فصل پنجم: Property، دکوراتورهای کلاسی و متدهای جادویی

## مقدمه

تا اینجا اصول بنیادین شیءگرایی را ساختیم. حالا سراغ ابزارهایی می‌رویم که پایتون را **پایتون** می‌کنند — قابلیت‌هایی که در بسیاری از زبان‌های دیگر وجود ندارند و کد شما را از «درست» به «حرفه‌ای» ارتقا می‌دهند.

سه دسته ابزار در این فصل:

- **Property**
- **دکوراتورهای کلاسی یا Class Decorators**
- **متدهای جادویی یا Magic Methods**

این‌ها همان چیزهایی‌اند که وقتی کد جنگو یا SQLAlchemy را می‌خوانید، همه‌جا می‌بینید.

## چرا این ابزارها را یاد بگیریم؟

بدون این‌ها هم می‌شود کد نوشت، اما با این‌ها می‌توان کد هایی به مراتب حرفه‌ای تر نوشت. با آن‌ها:

- می‌توانید دسترسی به داده را کنترل کنید بی‌آنکه کد کاربر تغییر کند (Property)
- اشیاء را با منطق‌های مختلف بسازید (Factory با `@classmethod`)
- کلاس‌هایتان با حلقه‌ی `for`، عملگر `==`، عبارت `with` و... یکپارچه شوند (متدهای جادویی)
- کدی بنویسید که با فریمورک‌های مدرن پایتون هماهنگ باشد

## Property

## Property در پایتون: از داده‌های ساده تا ویژگی‌های هوشمند

### مقدمه‌ای بر مفاهیم پایه

در برنامه‌نویسی شیءگرا، اشیاء دارای ویژگی‌هایی هستند که وضعیت آنها را مشخص می‌کنند. در پایتون، این ویژگی‌ها معمولاً به صورت attributeهای ساده تعریف می‌شوند. برای نمونه، کلاس `Student` را در نظر بگیرید که دارای attributeهای `name` و `score` است:

```python
class Student:
    def __init__(self, name, score):
        self.name = name
        self.score = score
```

کار با این attributeها بسیار ساده و مستقیم است:

```python
student = Student("احمد", 85)
print(student.score)   # 85
student.score = 90     # تغییر مقدار
```

این شیوه‌ی کار، در بسیاری از موارد کافی و مناسب است. اما گاهی نیاز می‌شود که فراتر از یک ذخیره‌سازی ساده، منطق خاصی را هنگام دسترسی به داده‌ها اعمال کنیم.

### ضرورت وجود منطق در دسترسی به داده‌ها

فرض کنید در پروژه‌ای، نیاز به اعمال محدودیت برای نمره‌ی دانشجو داریم. نمره نباید کمتر از ۰ یا بیشتر از ۱۰۰ باشد. روش ابتدایی برای حل این مسئله، تعریف متدهایی برای دریافت و تنظیم مقدار است:

```python
class Student:
    def __init__(self, name, score):
        self._score = score   # underscore نشان‌دهنده‌ی داده‌ی داخلی است

    def set_score(self, value):
        if 0 <= value <= 100:
            self._score = value
        else:
            raise ValueError("نمره باید بین ۰ و ۱۰۰ باشد")

    def get_score(self):
        return self._score
```

این رویکرد اگرچه مشکل اعتبارسنجی را حل می‌کند، اما هزینه‌ی قابل‌توجهی دارد: کاربران کلاس باید به جای دسترسی مستقیم به attribute، از متدهای `get_score` و `set_score` استفاده کنند. این تغییر، تمام کدهای قبلی که از `student.score` استفاده می‌کردند را از کار می‌اندازد و نیاز به بازنویسی گسترده دارد.

### مفهوم Property در پایتون

پایتون با ارائه‌ی مکانیزم `property`، راهی ظریف برای حل این مسئله ارائه می‌دهد. با استفاده از `property`، می‌توان یک attribute ساده را به گونه‌ای تبدیل کرد که دسترسی به آن، در پشت صحنه، یک متد را فراخوانی کند. در عین حال، نحوه‌ی دسترسی برای کاربر، کاملاً مشابه حالت قبل باقی می‌ماند:

```python
class Student:
    def __init__(self, name, score):
        self._score = score

    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if 0 <= value <= 100:
            self._score = value
        else:
            raise ValueError("نمره باید بین ۰ و ۱۰۰ باشد")
```

حال کاربر می‌تواند به همان شیوه‌ی قبلی با attribute کار کند:

```python
student = Student("احمد", 85)
print(student.score)   # 85 ← متد getter فراخوانی می‌شود
student.score = 90     # ← متد setter فراخوانی می‌شود
# student.score = 150  # ValueError!
```

**نکته‌ی کلیدی**: کد کاربر هیچ تغییری نکرده است، اما اکنون اعتبارسنجی به صورت خودکار اعمال می‌شود.

### مفهوم فلسفی Property: تفکیک رابط از پیاده‌سازی

`property` در پایتون، فراتر از یک ابزار ساده برای اعتبارسنجی است. این مکانیزم، تجسمی از اصل مهم انتزاع (Abstraction) در برنامه‌نویسی شیءگراست:

- **رابط (Interface)**: کاربر با `student.score` کار می‌کند؛ یک attribute ساده و قابل‌درک.
- **پیاده‌سازی (Implementation)**: پشت این رابط، هر منطقی می‌تواند قرار گیرد؛ از اعتبارسنجی ساده تا دریافت داده از دیتابیس.

این تفکیک، امکان تغییر پیاده‌سازی را بدون تأثیر بر کد کاربر فراهم می‌کند. در واقع، `property` به توسعه‌دهنده این قدرت را می‌دهد که هر زمان نیاز به منطق جدیدی پیدا کرد، attribute ساده را به `property` تبدیل کند، بدون آنکه کاربران کلاس متوجه تغییری شوند.

### ویژگی‌های محاسبه‌شونده (Computed Attributes)

یکی از کاربردهای اساسی `property`، ایجاد ویژگی‌هایی است که مقدارشان از داده‌های دیگر شیء محاسبه می‌شود. چنین ویژگی‌هایی، در حافظه ذخیره نمی‌شوند، بلکه در لحظه‌ی دسترسی، بر اساس وضعیت فعلی شیء تولید می‌شوند:

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        """مساحت مستطیل: یک ویژگی محاسبه‌شونده"""
        return self.width * self.height
```

در این مثال، `area` یک داده‌ی ذخیره‌شده نیست، بلکه یک ویژگی است که از عرض و ارتفاع به‌دست می‌آید. کاربر با `rectangle.area` کار می‌کند، گویی که یک attribute ساده است:

```python
rect = Rectangle(4, 5)
print(rect.area)   # 20
rect.width = 10
print(rect.area)   # 50 (به‌روزرسانی خودکار)
```

این رویکرد مزایای قابل‌توجهی دارد:

- **سازگاری همیشگی**: مقدار `area` هیچگاه با داده‌های `width` و `height` ناهماهنگ نمی‌شود، زیرا هر بار از نو محاسبه می‌شود.
- **صرفه‌جویی در حافظه**: داده‌های اضافی در حافظه نگهداری نمی‌شوند.
- **شفافیت کد**: کاربر با یک attribute ساده کار می‌کند و نیازی به دانستن پیچیدگی محاسبه ندارد.

### ویژگی‌های فقط‌خواندنی (Read-Only Properties)

اگر برای یک `property`، متد `setter` تعریف نشود، آن ویژگی به صورت فقط‌خواندنی در می‌آید. این قابلیت برای مواقعی مناسب است که تغییر مستقیم یک ویژگی، منطقی نباشد:

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return 3.14159 * self.radius ** 2

    @property
    def diameter(self):
        return 2 * self.radius

circle = Circle(5)
print(circle.area)       # 78.53975
print(circle.diameter)   # 10
# circle.area = 100      # AttributeError: can't set attribute
```

در اینجا، `area` و `diameter` از روی شعاع محاسبه می‌شوند و کاربر نمی‌تواند آنها را مستقیماً تغییر دهد، که این با منطق ریاضی سازگار است.

### کش کردن محاسبات سنگین با cached_property

در برخی موارد، محاسبه‌ی یک ویژگی سنگین است و نمی‌خواهیم هر بار که به آن دسترسی پیدا می‌شود، دوباره محاسبه شود. ماژول `functools` ابزاری به نام `cached_property` ارائه می‌دهد که اولین بار مقدار را محاسبه و ذخیره می‌کند، و دفعات بعد، همان مقدار ذخیره‌شده را باز می‌گرداند:

```python
from functools import cached_property

class Dataset:
    def __init__(self, file_path):
        self.file_path = file_path

    @cached_property
    def data(self):
        """خواندن داده‌ها از فایل: فقط یک بار انجام می‌شود"""
        print("در حال خواندن فایل حجیم...")
        # عملیات سنگین خواندن فایل
        return [1, 2, 3, 4, 5]

dataset = Dataset("data.csv")
print(dataset.data)   # چاپ: "در حال خواندن فایل حجیم..." و سپس [1, 2, 3, 4, 5]
print(dataset.data)   # فقط [1, 2, 3, 4, 5] (بدون خواندن مجدد فایل)
```

تفاوت اساسی با `@property` در این است که `@property` هر بار محاسبه را تکرار می‌کند، در حالی که `@cached_property` نتیجه را در حافظه نگه می‌دارد. این ابزار برای داده‌هایی که تغییر نمی‌کنند و دسترسی به آنها هزینه‌بر است، ایده‌آل می‌باشد. البته باید در نظر داشت که اگر داده‌ی زیربنایی تغییر کند، کش کهنه می‌شود.

### استراتژی انتخاب میان attribute و property

برای تصمیم‌گیری بین استفاده از attribute ساده و property، می‌توان از قاعده‌ی زیر پیروی کرد:

1.  **با attribute ساده شروع کنید**: تا زمانی که نیازی به منطق اضافی ندارید، از attributeهای ساده استفاده کنید. این کار کد را ساده و خوانا نگه می‌دارد.
2.  **در صورت نیاز، به property مهاجرت کنید**: هر گاه به اعتبارسنجی، محاسبه، تبدیل واحد، یا هر منطق دیگری نیاز پیدا کردید، attribute را به `property` تبدیل کنید. این تغییر، کد کاربر را نمی‌شکند.

این رویکرد، که به «برنامه‌نویسی تکاملی» معروف است، به شما اجازه می‌دهد که سیستم را به تدریج و با اطمینان از عدم شکستن کدهای موجود، پیچیده‌تر کنید.

### جمع‌بندی: Property به مثابه یک لایه‌ی انتزاعی

`property` در پایتون، یک لایه‌ی انتزاعی (Abstraction Layer) بین نحوه‌ی ذخیره‌سازی داده و نحوه‌ی ارائه‌ی آن به کاربر ایجاد می‌کند. این لایه به توسعه‌دهنده اجازه می‌دهد:

- نحوه‌ی ذخیره‌سازی داده را بدون تغییر کد کاربر تغییر دهد.
- اعتبارسنجی و تبدیل داده‌ها را پیش از ذخیره یا پس از خواندن اعمال کند.
- ویژگی‌های محاسبه‌شونده را به صورت طبیعی و ساده در اختیار کاربر قرار دهد.

`property` ابزاری است که به کد شما انعطاف‌پذیری می‌بخشد و به شما امکان می‌دهد تا «چیستی» (رابط) را از «چگونگی» (پیاده‌سازی) جدا کنید؛ اصلی که در قلب برنامه‌نویسی شیءگرا قرار دارد.

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

قاعده‌ی پایتونی: **با attribute ساده شروع کنید.** فقط وقتی به اعتبارسنجی، محاسبه، یا کنترل دسترسی نیاز پیدا کردید، به property ارتقا دهید. property نساختن پیش از نیاز، خودش یک اصل است (YAGNI — «لازمش نخواهی داشت»).

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

- **Instance method** (`self`): روی یک شیء خاص کار می‌کند و به داده‌اش دسترسی دارد.
- **Class method** (`cls`): روی خود کلاس کار می‌کند، نه یک شیء. `cls` همان کلاس است.
- **Static method** (هیچ‌کدام): فقط یک تابع کمکی است که منطقاً به کلاس تعلق دارد اما نه به داده‌ی شیء نیاز دارد نه به داده‌ی کلاس.

### classmethod به‌عنوان Factory Method

پرکاربردترین استفاده‌ی `@classmethod`، ساخت شیء با منطق‌های جایگزین است. سازنده‌ی `__init__` یکی است، اما گاهی می‌خواهید از راه‌های مختلف شیء بسازید:

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

چرا `cls` به‌جای نوشتن مستقیم `Date`؟ چون اگر کسی از `Date` ارث بگیرد، `from_string` روی زیرکلاس هم درست کار می‌کند و نمونه‌ی زیرکلاس می‌سازد، نه `Date`. این انعطاف، مزیت `cls` است.

### staticmethod: توابع کمکی مرتبط

`@staticmethod` برای وقتی است که تابعی منطقاً به کلاس مربوط است اما نه به `self` نیاز دارد نه به `cls`:

```python
class Validator:
    @staticmethod
    def is_valid_email(text):
        return "@" in text and "." in text

# usable without creating an object
print(Validator.is_valid_email("a@b.com"))   # True
```

می‌شد این را یک تابع مستقل هم نوشت. پس چرا static method؟ برای **سازمان‌دهی**: وقتی این تابع به‌شدت با `Validator` مرتبط است، قراردادنش داخل کلاس، خواناتر و یکپارچه‌تر است. **Tradeoff:** اگر رابطه‌ی قوی‌ای با کلاس ندارد، تابع مستقل ساده اغلب بهتر است.

## متدهای جادویی (Dunder Methods)

متدهای جادویی — با نام‌های `__something__` — قلاب‌هایی هستند که پایتون در موقعیت‌های خاص خودکار صدایشان می‌زند. با پیاده‌سازی آن‌ها، کلاس شما با زبان یکپارچه می‌شود.

### __str__ و __repr__: دو چهره‌ی نمایش شیء

اولین جفتی که هر کلاس جدی باید بشناسد، متدهای نمایش‌اند. بدون آن‌ها، `print` روی شیء شما چیزی شبیه `<Money object at 0x7f3a...>` چاپ می‌کند که به هیچ دردی نمی‌خورد. پایتون دو قلاب نمایش دارد که مخاطب‌هایشان فرق می‌کند:

- `__str__`: نمایش **خوانا برای کاربر نهایی**. وقتی `print(obj)` یا `str(obj)` صدا زده شود، اجرا می‌شود.
- `__repr__`: نمایش **دقیق برای برنامه‌نویس** — ایدئال این است که بتوان با کپی‌کردنش شیء را بازساخت. وقتی شیء را در کنسول تایپ کنید یا `repr(obj)` بگیرید، اجرا می‌شود.

```python
class Money:
    def __init__(self, amount, currency):
        self.amount = amount
        self.currency = currency

    def __str__(self):
        return f"{self.amount:,} {self.currency}"            # for the user

    def __repr__(self):
        return f"Money({self.amount!r}, {self.currency!r})"  # for the programmer

m = Money(1000, "Toman")
print(m)          # 1,000 Toman            ← __str__
print(repr(m))    # Money(1000, 'Toman')   ← __repr__
print([m])        # [Money(1000, 'Toman')] ← inside containers, __repr__ is used
```

به خط آخر دقت کنید: وقتی شیء داخل لیست یا دیکشنری چاپ می‌شود، پایتون از `__repr__` استفاده می‌کند نه `__str__` — نکته‌ای که اولین‌بار همه را غافلگیر می‌کند.

قاعده‌ی حرفه‌ای: همیشه دست‌کم `__repr__` را تعریف کنید؛ اگر `__str__` تعریف نشده باشد، پایتون به‌جایش به همان `__repr__` عقب‌گرد می‌کند، پس با یک متد هر دو نمایش را پوشش داده‌اید. `__str__` را فقط وقتی جدا بنویسید که نمایش کاربرپسندِ متفاوتی لازم دارید.

### __format__

عضو سوم این خانواده `__format__` است که کنترل می‌کند وقتی شیء در f-string با مشخصه‌ی قالب می‌آید چه شود:

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

تفاوت `==` و `is`: `==` برابری **مقدار** را می‌سنجد (که `__eq__` تعریفش می‌کند)؛ `is` برابری **هویت** را (آیا دقیقاً همان شیء در حافظه است).

چرا `__hash__` هم لازم شد؟ چون به‌محض تعریف `__eq__`، پایتون شیء را غیرقابل‌hash می‌کند و نمی‌توانید در `set` یا کلید `dict` استفاده‌اش کنید. اگر می‌خواهید شیء در مجموعه یا دیکشنری برود، باید `__hash__` سازگار با `__eq__` تعریف کنید (دو شیء برابر باید hash یکسان داشته باشند):

```python
points = {Point(1, 2), Point(1, 2)}
print(len(points))   # 1 — because they are equal and have the same hash
```

### عملگرها را به خدمت بگیرید: __add__ و دوستانش

حالا که `==` را رام کردیم، سراغ بقیه‌ی عملگرها برویم. فرض کنید در سیستم صورت‌حساب با پول کار می‌کنید. جمع‌زدن مبلغ‌ها با متد، خوانا نیست: `total.add(price).add(tax)`. چقدر بهتر که خود `+` کار کند — و پایتون اجازه می‌دهد:

```python
class Money:
    def __init__(self, amount, currency="IRR"):
        self.amount = amount
        self.currency = currency

    def __repr__(self):
        return f"Money({self.amount:,} {self.currency})"

    def __add__(self, other):                 # self + other
        if isinstance(other, Money):
            if other.currency != self.currency:
                raise ValueError("cannot add different currencies")
            return Money(self.amount + other.amount, self.currency)
        if isinstance(other, int):            # Money + 1000 also makes sense
            return Money(self.amount + other, self.currency)
        return NotImplemented                 # "I don't know this type"

    def __radd__(self, other):                # other + self (when other fails first)
        return self.__add__(other)

    def __mul__(self, factor):                # Money * 3
        if isinstance(factor, int):
            return Money(self.amount * factor, self.currency)
        return NotImplemented

price = Money(500_000)
tax = Money(45_000)
print(price + tax)          # Money(545,000 IRR)
print(price + 5_000)        # Money(505,000 IRR)   ← __add__ with int
print(5_000 + price)        # Money(505,000 IRR)   ← __radd__ saved us
print(price * 3)            # Money(1,500,000 IRR)
print(sum([price, tax], Money(0)))   # Money(545,000 IRR) — even sum() works now
```

سه نکته‌ی حرفه‌ای در این کد هست که در بیشتر آموزش‌ها پیدایش نمی‌کنید:

**`NotImplemented` (نه `NotImplementedError`!)** یک مقدار ویژه است، نه استثنا. وقتی برمی‌گردانیدش، دارید مؤدبانه به پایتون می‌گویید «من با این نوع آشنا نیستم» — و پایتون به‌جای شکست، **شانس را به طرف مقابل می‌دهد**. اگر او هم بلد نبود، آن‌وقت `TypeError` می‌آید. اگر به‌جایش استثنا پرت می‌کردید، این مذاکره‌ی دوطرفه را می‌کشتید.

**`__radd__` همان شانس دوم است:** در `5_000 + price`، پایتون اول `int.__add__(5000, price)` را امتحان می‌کند که `Money` را نمی‌شناسد و `NotImplemented` می‌دهد؛ بعد نوبت `price.__radd__(5000)` می‌رسد. بدون `__radd__`، جمع ما فقط یک‌طرفه کار می‌کرد — باگی که معمولاً اولین‌بار در `sum()` خودش را نشان می‌دهد، چون `sum` از صفر عددی شروع می‌کند.

**عملگر جدید، شیء جدید:** `__add__` ما `Money` تازه می‌سازد و self را دست نمی‌زند — مثل خود اعداد و رشته‌های پایتون. اگر بخواهید `+=` درجا (in-place) عمل کند، `__iadd__` را جدا تعریف می‌کنید؛ برای اشیای تغییرناپذیر منطقی مثل پول، همین ساخت نسخه‌ی جدید درست‌تر است.

برای مقایسه‌ها (`<`، `>=`، ...) هم می‌شود شش متد نوشت — اما لازم نیست. `functools.total_ordering` با داشتن `__eq__` و فقط یکی از مقایسه‌ها، بقیه را خودش می‌سازد:

```python
from functools import total_ordering

@total_ordering
class Version:
    def __init__(self, major, minor):
        self.major, self.minor = major, minor

    def __eq__(self, other):
        return (self.major, self.minor) == (other.major, other.minor)

    def __lt__(self, other):
        return (self.major, self.minor) < (other.major, other.minor)

print(Version(3, 10) > Version(3, 9))    # True — we never wrote __gt__!
print(Version(2, 0) >= Version(2, 0))    # True — nor __ge__
```

**Tradeoff:** عملگرها فقط وقتی تعریف کنید که معنایشان برای خواننده **بدیهی** باشد. `Money + Money` بدیهی است؛ اما `User + User` یعنی چه؟ ازدواج؟ ادغام حساب؟ عملگر مبهم از متد بدنام هم بدتر است، چون ظاهرش ساده و رفتارش غافلگیرکننده است.

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

با این چند متد، `Playlist` طوری رفتار می‌کند که انگار یک لیست داخلی پایتون است — این اوج یکپارچگی با زبان است.

### پروتکل Iterator: __iter__ و __next__

`__iter__` به‌تنهایی می‌تواند یک iterator داخلی برگرداند (مثل بالا). اما اگر بخواهید منطق پیمایش سفارشی داشته باشید، از `__next__` هم استفاده می‌کنید:

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

تفاوت با `__getitem__`: `__getitem__` برای دسترسی ایندکسی است؛ پروتکل iterator (`__iter__`/`__next__`) برای پیمایش کنترل‌شده و بالقوه بی‌پایان یا محاسبه‌شونده مناسب‌تر است.

### میان‌بُر پایتونی: جنریتور به‌جای کلاس Iterator

حالا که زحمت `Countdown` را کشیدیم، بگذارید رازی را بگویم: در کد حرفه‌ای پایتون، به‌ندرت کسی کلاس iterator دستی می‌نویسد. کلیدواژه‌ی `yield` همان پروتکل را **خودکار** پیاده می‌کند:

```python
def countdown(start):
    while start > 0:
        yield start          # pause here, hand the value out, resume later
        start -= 1

for n in countdown(3):
    print(n)                 # 3, 2, 1 — identical behavior, three lines
```

تابعی که `yield` دارد، **جنریتور** است: با هر فراخوانی، شیئی برمی‌گرداند که `__iter__` و `__next__` را از قبل دارد. اجرایش «قابل توقف» است — به `yield` که می‌رسد مقدار را تحویل می‌دهد و منجمد می‌شود تا مقدار بعدی خواسته شود. حالت بین مراحل (متغیر `start`) هم به‌جای attributeهای دستی، در خود متغیرهای محلی تابع زنده می‌ماند.

این میان‌بُر داخل کلاس‌ها هم درخشان است. `__iter__` می‌تواند خودش یک جنریتور باشد:

```python
class OrderHistory:
    def __init__(self):
        self._orders = []

    def add(self, order_id, amount):
        self._orders.append((order_id, amount))

    def __iter__(self):
        for order_id, amount in self._orders:
            if amount > 0:               # filtering logic, cleanly inline
                yield order_id

history = OrderHistory()
history.add(1, 90_000)
history.add(2, 0)          # a cancelled order
history.add(3, 45_000)
print(list(history))       # [1, 3] — for/list/in all work, no __next__ in sight
```

**قاعده‌ی انتخاب:** کلاس iterator دستی وقتی می‌ارزد که پیمایش، خودش یک موجودیت کامل با چند متد و حالت قابل‌بازرسی باشد (مثلاً iterator ی که بشود pause/resume/skipش کرد). برای بقیه‌ی موارد — یعنی تقریباً همیشه — `yield` کوتاه‌تر، خواناتر و کم‌خطاتر است. این را در فصل دوازدهم دوباره می‌بینید: خیلی از الگوهای کلاسیک، در پایتون میان‌بُر زبانی دارند.

### Context Manager: __enter__ و __exit__

این دو متد اجازه می‌دهند کلاس شما با عبارت `with` کار کند — الگوی استاندارد مدیریت منابع (فایل، اتصال دیتابیس، قفل):

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

قدرت context manager این است که `__exit__` **همیشه** اجرا می‌شود — چه بلوک عادی تمام شود چه با استثنا. برای همین برای آزادکردن مطمئن منابع ایده‌آل است.

### __call__: شیءای که تابع می‌شود

`__call__` شیء شما را **قابل‌فراخوانی مثل تابع** می‌کند: با تعریف آن، عبارت `obj(...)` مجاز می‌شود و همین متد اجرا می‌شود:

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, x):           # makes the object callable
        return x * self.factor

double = Multiplier(2)
print(double(10))       # 20 — as if double were a function!
print(callable(double)) # True
```

چرا به‌جای یک تابع ساده، کلاس با `__call__` بسازیم؟ چون این «تابع»، **حالت** هم دارد: `factor` را در خودش نگه می‌دارد. هر جا چیزی لازم دارید که مثل تابع صدا زده شود اما بین فراخوانی‌ها چیزی به خاطر بسپارد، `__call__` جواب است. یک کاربرد واقعی‌اش، ماشین حالت است:

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

- `__getattr__`: فقط وقتی صدا زده می‌شود که attribute به‌روش معمول **پیدا نشود**. برای مقادیر پیش‌فرض یا پروکسی مفید است.
- `__getattribute__`: برای **هر** دسترسی صدا زده می‌شود (خطرناک‌تر، به‌ندرت لازم).
- `__setattr__`: هنگام هر مقداردهی attribute. اینجا باید مراقب **حلقه‌ی بی‌نهایت** بود.

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

`__get__`، `__set__` و `__delete__` هسته‌ی **Descriptor** هستند — مکانیزمی که خود `@property`، `@classmethod` و حتی متدها روی آن ساخته شده‌اند. Descriptor به شما اجازه می‌دهد منطق دسترسی به attribute را در یک کلاس جداگانه و **قابل‌استفاده‌ی مجدد** بگذارید:

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

مزیت نسبت به property: منطق `Positive` یک‌بار نوشته شد و روی `price` و `weight` (و هر تعداد دیگر) اعمال شد، بی‌تکرار. Descriptorها آن‌قدر مهم‌اند که در فصل پانزدهم یک فصل کامل برایشان داریم.

## خلاصه

| ابزار | کاربرد اصلی |
|-------|-------------|
| `@property` | دسترسی attribute-مانند با منطق پشت آن |
| `@cached_property` | محاسبه‌ی گران، فقط یک‌بار |
| `@classmethod` | Factory Method؛ ساخت شیء با منطق جایگزین (`cls`) |
| `@staticmethod` | تابع کمکی مرتبط با کلاس، بدون `self`/`cls` |
| `__str__`/`__repr__` | نمایش برای کاربر / برنامه‌نویس |
| `__eq__`/`__hash__` | برابری بر اساس مقدار، عضویت در set/dict |
| `__getitem__`/`__len__`/`__iter__` | تبدیل کلاس به کانتینر |
| `__enter__`/`__exit__` | پشتیبانی از `with` (مدیریت منابع) |
| `__call__` | شیء قابل‌فراخوانی مثل تابع |
| Descriptor | منطق دسترسی قابل‌استفاده‌ی مجدد |

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

**نکته‌ی طلایی:** متدهای جادویی وسوسه‌انگیزند، اما همه را لازم ندارید. فقط آن‌هایی را پیاده کنید که واقعاً به یکپارچگی کلاس با زبان کمک می‌کنند. یک کلاس ساده که فقط `__repr__` دارد، بهتر از کلاسی است که ده متد جادویی دارد که هیچ‌کدام لازم نبوده‌اند.

## پرسش‌های مصاحبه

**۱. «فرق classmethod و staticmethod؟ هرکدام کی؟»** — `classmethod` کلاس (`cls`) را می‌گیرد → سازنده‌های جایگزین (`from_dict`) و کارهایی که به خود کلاس ربط دارند و با ارث‌بری درست رفتار می‌کنند؛ `staticmethod` هیچ‌چیز نمی‌گیرد → تابع کمکی مرتبط که فقط از نظر موضوعی به کلاس چسبیده. جواب ممتاز: «اگر staticmethod به هیچ‌چیز کلاس دست نمی‌زند، شاید اصلاً باید یک تابع سطح ماژول باشد.»

**۲. «__str__ و __repr__ چه فرقی دارند؟»** — `__str__` برای انسان (خروجی `print`)، `__repr__` برای برنامه‌نویس (دیباگ، REPL؛ ترجیحاً بازتولیدکننده‌ی شیء). قاعده‌ی حرفه‌ای: `__repr__` را همیشه بنویسید — اگر `__str__` نباشد، به `__repr__` عقب‌گرد می‌شود، پس یک تیر و دو نشان.

**۳. «قرارداد __eq__ و __hash__ چیست؟»** — دو شیء برابر (`__eq__`) باید hash یکسان داشته باشند، وگرنه `set` و `dict` رفتار غیرقابل‌پیش‌بینی پیدا می‌کنند؛ و تعریف `__eq__` به‌تنهایی، شیء را unhashable می‌کند. اشاره به اینکه اشیای تغییرپذیر بهتر است hashable نباشند، عمق جواب را نشان می‌دهد.

**۴. «context manager چطور کار می‌کند و چرا از try/finally بهتر است؟»** — پروتکل `__enter__`/`__exit__`؛ `with` تضمین می‌کند `__exit__` حتی هنگام استثنا اجرا شود. بهتر است چون الگوی «باز کن، مطمئن ببند» را یک‌بار در خود کلاس کپسوله می‌کند نه در تک‌تک محل‌های استفاده. اگر مقدار برگشتی `__exit__` (بلعیدن یا عبور استثنا) را هم بگویید، سوال بعدی را پیش‌خور کرده‌اید.

**در فصل بعد:** به سراغ ارث‌بری پیشرفته می‌رویم — وراثت چندگانه، الگوریتم C3 برای MRO، و Mixinها — تا بفهمیم پایتون چطور سلسله‌مراتب‌های پیچیده را مدیریت می‌کند.
