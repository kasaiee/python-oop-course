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

در برنامه‌نویسی شیءگرا، اشیاء دارای ویژگی‌هایی هستند که وضعیت آنها را مشخص می‌کنند. در پایتون، این ویژگی‌ها معمولاً به صورت attributeهای ساده تعریف می‌شوند. برای نمونه، کلاس `Student` را در نظر بگیرید که دارای attributeهای `name` و `score` است:

```python
class Student:
    def __init__(self, name, score):
        self.name = name
        self.score = score
```

کار با این attributeها بسیار ساده و مستقیم است:

```python
student = Student("Ahmad", 85)
print(student.score)   # 85
student.score = 90     # تغییر مقدار
```

این شیوه‌ی کار، در بسیاری از موارد کافی و مناسب است. اما گاهی نیاز می‌شود که فراتر از یک ذخیره‌سازی ساده، منطق خاصی را هنگام دسترسی به داده‌ها اعمال کنیم.

### مفهوم بنیادین: ویژگی‌های پویا (Dynamic Attributes)

در برنامه‌نویسی شیءگرا، با دو نوع ویژگی مواجه هستیم: ویژگی‌های ایستا که مستقیماً ذخیره می‌شوند، و ویژگی‌های پویا که در لحظه محاسبه می‌گردند. `property` در پایتون ابزاری است برای تعریف ویژگی‌های پویا به گونه‌ای که مانند ویژگی‌های ایستا در دسترس باشند.

### مثال آغازین: سن و تاریخ تولد

فرض کنید در کلاس `Person`، می‌خواهیم سن کاربر را در اختیار داشته باشیم. اما سن یک مفهوم پویا است؛ با گذشت زمان تغییر می‌کند و ذخیره‌ی آن به عنوان یک عدد ثابت، منطقی نیست. راه صحیح، ذخیره‌ی تاریخ تولد است که یک داده‌ی ثابت محسوب می‌شود. سپس سن را به عنوان یک ویژگی محاسبه‌شونده تعریف می‌کنیم:

```python
from datetime import date

class Person:
    def __init__(self, name, birth_date):
        self.name = name
        self.birth_date = birth_date

    @property
    def age(self):
        """سن بر اساس تاریخ تولد محاسبه می‌شود"""
        today = date.today()
        return today.year - self.birth_date.year - (
            (today.month, today.day) < (self.birth_date.month, self.birth_date.day)
        )
```

اکنون کاربر به سادگی با `person.age` کار می‌کند، گویی که یک ویژگی عادی است:

```python
person = Person("Ahmad", date(1990, 5, 15))
print(person.age)   # 36 (مقدار بر اساس تاریخ امروز محاسبه می‌شود)
```

هر بار که `age` خوانده می‌شود، مقدار آن بر اساس تاریخ فعلی بازمحاسبه می‌گردد. این یعنی همیشه دقیق است و نیازی به به‌روزرسانی دستی ندارد.

### تعامل دوطرفه: ارتباط میان ویژگی‌ها

حال فرض کنید کاربر بخواهد سن را تنظیم کند، نه تاریخ تولد را. در این حالت، تنظیم سن باید به معنای تغییر تاریخ تولد باشد. اینجاست که `setter` معنا پیدا می‌کند:

```python
from datetime import date, timedelta

class Person:
    def __init__(self, name, birth_date):
        self.name = name
        self.birth_date = birth_date

    @property
    def age(self):
        today = date.today()
        return today.year - self.birth_date.year - (
            (today.month, today.day) < (self.birth_date.month, self.birth_date.day)
        )

    @age.setter
    def age(self, value):
        """تنظیم سن، تاریخ تولد را به‌روز می‌کند"""
        today = date.today()
        self.birth_date = date(today.year - value, today.month, today.day)
```

اکنون کاربر می‌تواند سن را به صورت مستقیم تنظیم کند:

```python
person = Person("Ahmad", date(1990, 5, 15))
print(person.age)          # 36
person.age = 30            # تنظیم سن جدید
print(person.birth_date)   # تاریخ تولد به‌روز شده است
```

در اینجا، `property` یک ارتباط دوطرفه برقرار کرده است: خواندن سن، تاریخ تولد را محاسبه می‌کند، و نوشتن سن، تاریخ تولد را تغییر می‌دهد.

### اعتبارسنجی با setter: کاربردی دیگر

از `setter` نه تنها برای برقراری ارتباط میان ویژگی‌ها، بلکه برای اعتبارسنجی داده‌ها نیز استفاده می‌شود. فرض کنید در کلاس `Student`، نمره باید همواره بین ۰ و ۱۰۰ باشد. روش ابتدایی برای حل این مسئله، تعریف متدهایی برای دریافت و تنظیم مقدار است:

```python
class Student:
    def __init__(self, name, score):
        self._score = score

    def set_score(self, value):
        if 0 <= value <= 100:
            self._score = value
        else:
            raise ValueError("score must be between 0 and 100")

    def get_score(self):
        return self._score
```

این رویکرد اگرچه مشکل اعتبارسنجی را حل می‌کند، اما هزینه‌ی قابل‌توجهی دارد: کاربران کلاس باید به جای دسترسی مستقیم به attribute، از متدهای `get_score` و `set_score` استفاده کنند. این تغییر، تمام کدهای قبلی را از کار می‌اندازد و نیاز به بازنویسی گسترده دارد.

پایتون با ارائه‌ی مکانیزم `property`، راهی ظریف برای حل این مسئله ارائه می‌دهد:

```python
class Student:
    def __init__(self, name, score):
        self.name = name
        self._score = score

    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if 0 <= value <= 100:
            self._score = value
        else:
            raise ValueError("score must be between 0 and 100")
```

حال کاربر می‌تواند به همان شیوه‌ی قبلی با attribute کار کند:

```python
student = Student("Ahmad", 85)
print(student.score)      # 85
student.score = 95        # معتبر است
# student.score = 150     # ValueError: score must be between 0 and 100
```

**نکته‌ی کلیدی**: کد کاربر هیچ تغییری نکرده است، اما اکنون اعتبارسنجی به صورت خودکار اعمال می‌شود.

همین رویکرد را می‌توان برای اعتبارسنجی سن در کلاس `Person` نیز به کار برد:

```python
class Person:
    def __init__(self, name, birth_date):
        self.name = name
        self.birth_date = birth_date

    @property
    def age(self):
        today = date.today()
        return today.year - self.birth_date.year - (
            (today.month, today.day) < (self.birth_date.month, self.birth_date.day)
        )

    @age.setter
    def age(self, value):
        if 0 < value < 150:
            today = date.today()
            self.birth_date = date(today.year - value, today.month, today.day)
        else:
            raise ValueError("Age must be between 0 and 150")
```

اینجا، `setter` علاوه بر برقراری ارتباط بین سن و تاریخ تولد، از تنظیم سن نامعقول نیز جلوگیری می‌کند.

### مثال دوم: دایره و راه‌های مختلف ساخت

در برخی موارد، یک شیء را می‌توان با داده‌های متفاوتی تعریف کرد. برای نمونه، یک دایره را می‌توان با شعاع یا با مساحت تعریف کرد. `property` به ما اجازه می‌دهد تا هر دو رویکرد را پشتیبانی کنیم:

```python
import math

class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        """مساحت از روی شعاع محاسبه می‌شود"""
        return math.pi * self.radius ** 2

    @area.setter
    def area(self, value):
        """تنظیم مساحت، شعاع را به‌روز می‌کند"""
        self.radius = math.sqrt(value / math.pi)
```

اکنون کاربر می‌تواند به هر دو شکل با دایره کار کند:

```python
circle = Circle(5)
print(circle.area)          # 78.53981633974483
circle.area = 100           # تنظیم مساحت جدید
print(circle.radius)        # 5.641895835477563 (شعاع به‌روز شده است)
```

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
        print("Reading large file...")
        # عملیات سنگین خواندن فایل
        return [1, 2, 3, 4, 5]

dataset = Dataset("data.csv")
print(dataset.data)   # چاپ: "Reading large file..." و سپس [1, 2, 3, 4, 5]
print(dataset.data)   # فقط [1, 2, 3, 4, 5] (بدون خواندن مجدد فایل)
```

تفاوت اساسی با `@property` در این است که `@property` هر بار محاسبه را تکرار می‌کند، در حالی که `@cached_property` نتیجه را در حافظه نگه می‌دارد. این ابزار برای داده‌هایی که تغییر نمی‌کنند و دسترسی به آنها هزینه‌بر است، ایده‌آل می‌باشد. البته باید در نظر داشت که اگر داده‌ی زیربنایی تغییر کند، کش کهنه می‌شود.

### مفهوم فلسفی Property: تفکیک رابط از پیاده‌سازی

`property` در پایتون، فراتر از یک ابزار ساده برای اعتبارسنجی است. این مکانیزم، تجسمی از اصل مهم انتزاع (Abstraction) در برنامه‌نویسی شیءگراست:

- **رابط (Interface)**: کاربر با `student.score` کار می‌کند؛ یک attribute ساده و قابل‌درک.
- **پیاده‌سازی (Implementation)**: پشت این رابط، هر منطقی می‌تواند قرار گیرد؛ از اعتبارسنجی ساده تا دریافت داده از دیتابیس.

این تفکیک، امکان تغییر پیاده‌سازی را بدون تأثیر بر کد کاربر فراهم می‌کند. در واقع، `property` به توسعه‌دهنده این قدرت را می‌دهد که هر زمان نیاز به منطق جدیدی پیدا کرد، attribute ساده را به `property` تبدیل کند، بدون آنکه کاربران کلاس متوجه تغییری شوند.

### استراتژی انتخاب میان attribute و property

برای تصمیم‌گیری بین استفاده از attribute ساده و property، می‌توان از قاعده‌ی زیر پیروی کرد:

1. **با attribute ساده شروع کنید**: تا زمانی که نیازی به منطق اضافی ندارید، از attributeهای ساده استفاده کنید. این کار کد را ساده و خوانا نگه می‌دارد.
2. **در صورت نیاز، به property مهاجرت کنید**: هر گاه به اعتبارسنجی، محاسبه، تبدیل واحد، یا هر منطق دیگری نیاز پیدا کردید، attribute را به `property` تبدیل کنید. این تغییر، کد کاربر را نمی‌شکند.

این رویکرد، که به «برنامه‌نویسی تکاملی» معروف است، به شما اجازه می‌دهد که سیستم را به تدریج و با اطمینان از عدم شکستن کدهای موجود، پیچیده‌تر کنید. property نساختن پیش از نیاز، خودش یک اصل است (YAGNI — «لازمش نخواهی داشت»).

### جمع‌بندی: Property به مثابه یک لایه‌ی انتزاعی

`property` در پایتون، یک لایه‌ی انتزاعی (Abstraction Layer) بین نحوه‌ی ذخیره‌سازی داده و نحوه‌ی ارائه‌ی آن به کاربر ایجاد می‌کند. این لایه به توسعه‌دهنده اجازه می‌دهد:

- نحوه‌ی ذخیره‌سازی داده را بدون تغییر کد کاربر تغییر دهد.
- اعتبارسنجی و تبدیل داده‌ها را پیش از ذخیره یا پس از خواندن اعمال کند.
- ویژگی‌های محاسبه‌شونده را به صورت طبیعی و ساده در اختیار کاربر قرار دهد.

`property` ابزاری است که به کد شما انعطاف‌پذیری می‌بخشد و به شما امکان می‌دهد تا «چیستی» (رابط) را از «چگونگی» (پیاده‌سازی) جدا کنید؛ اصلی که در قلب برنامه‌نویسی شیءگرا قرار دارد.


## دکوراتورهای کلاسی: classmethod و staticmethod

### درآمدی بر انواع متدها در کلاس

در پایتون، متدهای تعریف‌شده در کلاس‌ها به سه دسته تقسیم می‌شوند که هر یک نقش و کاربرد مشخصی دارند. در بخش‌های پیشین، با متدهای معمولی آشنا شدید که اولین پارامتر آنها `self` است و به یک نمونه‌ی خاص از کلاس دسترسی دارند. اکنون دو نوع دیگر را بررسی می‌کنیم که رابطه‌ی متفاوتی با کلاس و نمونه‌های آن برقرار می‌کنند.

### نمای کلی از ساختار هر سه نوع متد

پیش از پرداختن به جزئیات، اجازه دهید یک نمای کلی از هر سه نوع متد مشاهده کنید:

```python
class Example:
    class_var = "shared"  # class-level attribute

    def instance_method(self):
        # regular method, receives instance as first argument
        return self

    @classmethod
    def class_method(cls):
        # class method, receives class as first argument
        return cls

    @staticmethod
    def static_method():
        # static method, receives no special first argument
        return "independent"
```

هر یک از این متدها، رابطه‌ی متفاوتی با کلاس و نمونه‌های آن دارند:

- **متد نمونه (Instance Method)**: با `self` کار می‌کند و به داده‌های یک نمونه‌ی خاص دسترسی دارد.
- **متد کلاس (Class Method)**: با `cls` کار می‌کند و به خود کلاس دسترسی دارد، نه به یک نمونه‌ی خاص.
- **متد ایستا (Static Method)**: نه به `self` نیاز دارد و نه به `cls`؛ صرفاً یک تابع است که در فضای نام کلاس قرار گرفته است.

### متد کلاس و کاربرد آن به عنوان کارخانه‌ی سازنده (Factory Method)

متد کلاس با دکوراتور `@classmethod` مشخص می‌شود و اولین پارامتر آن به جای `self`، `cls` نام دارد. این پارامتر ارجاعی به خود کلاس است، نه به یک نمونه‌ی خاص از آن کلاس.

**کاربرد اصلی**: ساخت نمونه‌های جدید با استفاده از ورودی‌های متفاوت. سازنده‌ی کلاس (`__init__`) تنها یک راه برای ساخت شیء ارائه می‌دهد، اما گاهی نیاز به روش‌های جایگزین داریم. به عنوان مثال، کلاس `Date` را در نظر بگیرید که می‌تواند یک تاریخ را به روش‌های مختلفی بسازد:

```python
class Date:
    def __init__(self, year, month, day):
        # constructor: initialize with year, month, day
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, text):
        # alternative constructor: create from a string like "2026-07-21"
        year, month, day = map(int, text.split("-"))
        return cls(year, month, day)

    @classmethod
    def today(cls):
        # alternative constructor: create with current date
        from datetime import date
        n = date.today()
        return cls(n.year, n.month, n.day)

    def __repr__(self):
        # string representation for debugging
        return f"Date({self.year}, {self.month}, {self.day})"
```

اکنون کاربر می‌تواند به روش‌های مختلفی یک شیء `Date` بسازد:

```python
d1 = Date(2026, 7, 21)           # regular constructor
d2 = Date.from_string("2026-07-21")  # from a string
d3 = Date.today()                # today's date

print(d1, d2, d3)
```

**چرا از `cls` استفاده می‌کنیم به جای نام کلاس؟**

اگر در متد `from_string` به جای `cls`، مستقیماً `Date` می‌نوشتیم، مشکلی پیش نمی‌آمد. اما مزیت استفاده از `cls` در **ارث‌بری** خود را نشان می‌دهد:

```python
class PersianDate(Date):
    # subclass of Date for Persian calendar
    pass

p = PersianDate.from_string("1405-04-30")  # cls correctly refers to PersianDate
print(type(p))  # <class '__main__.PersianDate'>
```

اگر به جای `cls`، نام `Date` را مستقیماً می‌نوشتیم، خروجی از نوع `PersianDate` نمی‌شد و انعطاف‌پذیری ارث‌بری را از دست می‌دادیم.

**کاربرد دیگر: دسترسی به متغیرهای کلاس**

متدهای کلاس می‌توانند به متغیرهای سطح کلاس (متغیرهایی که بین همه‌ی نمونه‌ها مشترک هستند) دسترسی داشته باشند:

```python
class Config:
    default_timeout = 30  # class-level variable

    @classmethod
    def get_timeout(cls):
        # access class variable
        return cls.default_timeout

    @classmethod
    def set_timeout(cls, value):
        # modify class variable
        cls.default_timeout = value

print(Config.get_timeout())  # 30
Config.set_timeout(60)
print(Config.get_timeout())  # 60
```

### متد ایستا و کاربرد آن به عنوان تابع کمکی

متد ایستا با دکوراتور `@staticmethod` مشخص می‌شود و هیچ پارامتر خاصی (نه `self` و نه `cls`) دریافت نمی‌کند. این متدها صرفاً توابعی هستند که در فضای نام کلاس قرار گرفته‌اند.

**چه زمانی از staticmethod استفاده کنیم؟**

زمانی که تابعی منطقاً به یک کلاس مرتبط است، اما به داده‌های نمونه یا خود کلاس نیازی ندارد. چنین توابعی معمولاً عملیات کمکی یا ابزاری انجام می‌دهند که ارتباط تنگاتنگی با مفهوم آن کلاس دارند.

**مثال: اعتبارسنجی ایمیل**

```python
class Validator:
    @staticmethod
    def is_valid_email(text):
        # check if email contains @ and .
        return "@" in text and "." in text

    @staticmethod
    def is_valid_phone(text):
        # check if phone is 11 digits
        return text.isdigit() and len(text) == 11
```

کاربر می‌تواند بدون ساخت نمونه از این متدها استفاده کند:

```python
print(Validator.is_valid_email("user@example.com"))  # True
print(Validator.is_valid_email("invalid"))           # False
print(Validator.is_valid_phone("09121234567"))       # True
```

**چرا به جای تابع مستقل، از staticmethod استفاده کنیم؟**

اگرچه می‌توان این توابع را به صورت توابع معمولی در سطح ماژول تعریف کرد، اما قرار دادن آنها در کلاس مزایایی دارد:

- **سازماندهی بهتر**: توابع مرتبط با یک مفهوم در یک جا جمع می‌شوند.
- **خوانایی بیشتر**: کدخوانان می‌دانند که این تابع به چه مفهومی مربوط است.
- **قابلیت کشف**: در محیط‌های توسعه، تکمیل خودکار (autocomplete) این توابع را در کنار کلاس نشان می‌دهد.

با این حال، اگر تابع ارتباط قوی با کلاس ندارد، بهتر است به عنوان یک تابع مستقل در سطح ماژول تعریف شود.

### مقایسه‌ی سه نوع متد در یک جدول

| ویژگی | متد نمونه (instance) | متد کلاس (classmethod) | متد ایستا (staticmethod) |
|-------|----------------------|------------------------|--------------------------|
| پارامتر اول | `self` (نمونه) | `cls` (کلاس) | هیچ‌کدام |
| دسترسی به داده‌های نمونه | دارد | ندارد | ندارد |
| دسترسی به داده‌های کلاس | دارد | دارد | ندارد |
| رفتار در ارث‌بری | معمولی | با `cls` به‌درستی کار می‌کند | بدون تغییر می‌ماند |
| کاربرد اصلی | کار با داده‌های شیء | Factory Method، کار با داده‌های کلاس | توابع کمکی مرتبط |

### مثالی جامع: کلاس User با هر سه نوع متد

برای درک بهتر تفاوت‌ها، یک مثال جامع ارائه می‌دهیم:

```python
class User:
    default_role = "guest"  # class-level attribute

    def __init__(self, username, email):
        # instance constructor
        self.username = username
        self.email = email

    def display(self):
        # instance method: uses instance data
        return f"User: {self.username} ({self.email})"

    @classmethod
    def from_dict(cls, data):
        # class method: alternative constructor from dictionary
        return cls(data["username"], data["email"])

    @classmethod
    def get_default_role(cls):
        # class method: access class data
        return cls.default_role

    @classmethod
    def set_default_role(cls, role):
        # class method: modify class data
        cls.default_role = role

    @staticmethod
    def is_valid_username(text):
        # static method: utility function, no class/instance data needed
        return len(text) >= 3 and text.isalnum()
```

کاربرد هر یک از این متدها:

```python
# instance method usage
user = User("ahmad", "ahmad@site.com")
print(user.display())

# class method as factory
new_user = User.from_dict({"username": "reza", "email": "reza@site.com"})
print(new_user.display())

# class methods for class-level data
print(User.get_default_role())  # guest
User.set_default_role("admin")
print(User.get_default_role())  # admin

# static method usage (no instance needed)
print(User.is_valid_username("ahmad"))  # True
print(User.is_valid_username("a"))      # False
```

### قاعده‌ی طلایی: چه زمانی از هر کدام استفاده کنیم؟

- **متد نمونه**: زمانی که نیاز به کار با داده‌های یک نمونه‌ی خاص دارید. این رایج‌ترین نوع متد است.

- **متد کلاس**:
  - زمانی که نیاز به ساخت نمونه با روش‌های جایگزین دارید (Factory Method).
  - زمانی که نیاز به دسترسی یا تغییر داده‌های سطح کلاس دارید.
  - زمانی که می‌خواهید متدی بنویسید که با ارث‌بری به‌درستی رفتار کند.

- **متد ایستا**:
  - زمانی که تابعی منطقاً به کلاس مرتبط است اما به داده‌های کلاس یا نمونه نیاز ندارد.
  - زمانی که تابع یک عملیات کمکی انجام می‌دهد که ارتباط تنگاتنگی با مفهوم کلاس دارد.

### جمع‌بندی: قدرت سازماندهی

دکوراتورهای کلاسی ابزارهایی برای سازماندهی بهتر کد هستند. متدهای کلاس به شما امکان می‌دهند تا روش‌های ساخت شیء را توسعه دهید و متدهای ایستا به شما اجازه می‌دهند تا توابع مرتبط را در کنار کلاس قرار دهید. هر دو ابزار، کد شما را خواناتر، منظم‌تر و قابل‌نگهداری‌تر می‌کنند.


## متدهای جادویی (Magic Methods) در پایتون

### متدهای جادویی چه هستند و چرا به آنها جادویی می‌گویند؟

در پایتون، متدهایی وجود دارند که نامشان با دو خط زیرخط (__) شروع و با دو خط زیرخط پایان می‌یابد؛ مانند `__init__`، `__str__` و `__add__`. به این متدها، «متدهای جادویی» یا «Dunder Methods» (مخفف Double UNDERscore) گفته می‌شود.

دلیل «جادویی» نامیدن آنها این است که **نیازی به فراخوانی مستقیم آنها ندارید**؛ پایتون خودش در موقعیت‌های خاص، آنها را به صورت خودکار صدا می‌زند. برای نمونه، هنگامی که یک شیء جدید می‌سازید، پایتون `__init__` را اجرا می‌کند. وقتی از عملگر `+` استفاده می‌کنید، پایتون به سراغ `__add__` می‌رود. و وقتی `print(obj)` را می‌نویسید، پایتون `__str__` را فرا می‌خواند.

این متدها، **پل ارتباطی میان کلاس شما و ساختارهای زبانی پایتون** هستند. با پیاده‌سازی آنها، به پایتون می‌آموزید که اشیاء شما چگونه باید با عملگرها، حلقه‌ها، توابع داخلی و سایر اجزای زبان تعامل داشته باشند.

در این بخش، مهم‌ترین و پرکاربردترین متدهای جادویی را به ترتیب از ساده به پیشرفته بررسی می‌کنیم.

---

### ۱. نمایش اشیاء: `__str__` و `__repr__`

وقتی یک شیء را چاپ می‌کنید یا در کنسول تایپ می‌کنید، پایتون باید آن را به رشته تبدیل کند. دو متد جادویی این کار را انجام می‌دهند.

**`__repr__`**: نمایش رسمی و دقیق شیء برای برنامه‌نویسان. هدف این است که اگر خروجی آن را کپی کنید، بتوانید شیء را بازسازی کنید. این متد زمانی استفاده می‌شود که:

- شیء را در کنسول (REPL) تایپ می‌کنید.
- از تابع `repr(obj)` استفاده می‌کنید.
- شیء داخل یک容器 (لیست، دیکشنری) قرار دارد و چاپ می‌شود.

**`__str__`**: نمایش خوانا و دوستانه برای کاربر نهایی. زمانی استفاده می‌شود که:

- از `print(obj)` استفاده می‌کنید.
- از تابع `str(obj)` استفاده می‌کنید.
- از قالب‌بندی رشته با `f"{obj}"` استفاده می‌کنید.

**قاعده‌ی طلایی**: همیشه `__repr__` را تعریف کنید. اگر `__str__` را تعریف نکنید، پایتون در مواقع نیاز به `__str__`، به‌جای آن از `__repr__` استفاده می‌کند. بنابراین با تعریف `__repr__`، یک تیر و دو نشان زده‌اید.

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages

    def __repr__(self):
        # returns a string that could recreate the object
        return f"Book({self.title!r}, {self.author!r}, {self.pages})"

    def __str__(self):
        # returns a user-friendly string
        return f"{self.title} by {self.author} ({self.pages} pages)"

book = Book("1984", "George Orwell", 328)

print(book)           # 1984 by George Orwell (328 pages)  ← __str__
print(repr(book))     # Book('1984', 'George Orwell', 328)  ← __repr__
print([book])         # [Book('1984', 'George Orwell', 328)]  ← __repr__ inside list
```

**نکته**: در `__repr__` از `!r` استفاده کردیم تا مطمئن شویم رشته‌ها با کوتیشن نمایش داده می‌شوند. این کار بازتولید شیء را دقیق‌تر می‌کند.

---

### ۲. متد `__format__`: کنترل قالب‌بندی پیشرفته

این متد به شما اجازه می‌دهد وقتی از f-string با مشخصه‌ی قالب‌بندی استفاده می‌کنید، خروجی را کنترل کنید.

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages

    def __format__(self, spec):
        if spec == "short":
            return f"{self.title} ({self.pages}p)"
        elif spec == "long":
            return f"{self.title} by {self.author}, {self.pages} pages"
        return str(self)

book = Book("1984", "George Orwell", 328)
print(f"{book:short}")   # 1984 (328p)
print(f"{book:long}")    # 1984 by George Orwell, 328 pages
```

---

### ۳. برابری و هش: `__eq__` و `__hash__`

به طور پیش‌فرض، دو شیء فقط زمانی برابر در نظر گرفته می‌شوند که دقیقاً یک شیء در حافظه باشند (همان هویت). با `__eq__` می‌توانید برابری را بر اساس **مقدار** تعریف کنید.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        # check if other is also a Point
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __hash__(self):
        # objects that are equal must have the same hash
        return hash((self.x, self.y))

p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = Point(2, 3)

print(p1 == p2)    # True (same values)
print(p1 == p3)    # False (different values)
print(p1 is p2)    # False (different objects in memory)

# Now Point objects can be used in sets and dictionaries
points_set = {p1, p2}
print(len(points_set))   # 1 (because p1 and p2 are equal)
```

**نکته‌ی مهم**: اگر `__eq__` را تعریف کنید، پایتون به طور خودکار `__hash__` را حذف می‌کند و شیء شما را «غیرقابل هش» می‌کند. این یعنی نمی‌توانید از آن در `set` یا به عنوان کلید `dict` استفاده کنید. اگر می‌خواهید چنین قابلیتی داشته باشید، باید `__hash__` را به گونه‌ای تعریف کنید که با `__eq__` سازگار باشد (دو شیء برابر باید هش یکسان داشته باشند).

---

### ۴. عملگرهای حسابی: `__add__`، `__radd__`، `__mul__` و دوستان

با پیاده‌سازی این متدها، اشیاء شما از عملگرهای ریاضی پشتیبانی می‌کنند.

```python
class Money:
    def __init__(self, amount, currency="IRR"):
        self.amount = amount
        self.currency = currency

    def __repr__(self):
        return f"Money({self.amount:,} {self.currency})"

    def __add__(self, other):
        # self + other
        if isinstance(other, Money):
            if other.currency != self.currency:
                raise ValueError("Cannot add different currencies")
            return Money(self.amount + other.amount, self.currency)
        if isinstance(other, (int, float)):
            return Money(self.amount + other, self.currency)
        return NotImplemented

    def __radd__(self, other):
        # other + self (when other doesn't know how to add Money)
        return self.__add__(other)

    def __mul__(self, factor):
        # self * factor
        if isinstance(factor, (int, float)):
            return Money(self.amount * factor, self.currency)
        return NotImplemented

price = Money(500_000)
tax = Money(45_000)

print(price + tax)           # Money(545,000 IRR)  ← __add__
print(price + 5_000)         # Money(505,000 IRR)  ← __add__ with int
print(5_000 + price)         # Money(505,000 IRR)  ← __radd__ handles this
print(price * 3)             # Money(1,500,000 IRR)  ← __mul__

# sum() works too!
print(sum([price, tax], Money(0)))   # Money(545,000 IRR)
```

**نکته‌ی کلیدی در مورد `NotImplemented`**: اگر نوع داده‌ای را نمی‌شناسید، `NotImplemented` را برگردانید (نه `NotImplementedError`). این به پایتون اجازه می‌دهد شانس را به طرف مقابل (مثلاً `__radd__`) بدهد. اگر هر دو طرف ناتوان باشند، پایتون خودش `TypeError` را پرتاب می‌کند.

**سایر عملگرها**:

- `__sub__` برای `-`
- `__truediv__` برای `/`
- `__floordiv__` برای `//`
- `__mod__` برای `%`
- `__pow__` برای `**`

برای عملگرهای مقایسه‌ای (`<`, `<=`, `>`, `>=`) نیز می‌توان متدهای جداگانه تعریف کرد، اما با استفاده از `functools.total_ordering` و تعریف تنها `__eq__` و یکی از مقایسه‌ها، بقیه خودکار ساخته می‌شوند:

```python
from functools import total_ordering

@total_ordering
class Version:
    def __init__(self, major, minor):
        self.major = major
        self.minor = minor

    def __eq__(self, other):
        return (self.major, self.minor) == (other.major, other.minor)

    def __lt__(self, other):
        return (self.major, self.minor) < (other.major, other.minor)

v1 = Version(3, 10)
v2 = Version(3, 9)
print(v1 > v2)    # True (we never wrote __gt__!)
print(v1 >= v2)   # True (we never wrote __ge__!)
```

---

### ۵. متد `__bool__`: تعیین صحت و سقم در شرط‌ها

این متد تعیین می‌کند که شیء شما در موقعیت‌های شرطی (مانند `if obj:` یا `while obj:`) چگونه رفتار کند.

```python
class ShoppingCart:
    def __init__(self):
        self.items = []

    def add_item(self, item):
        self.items.append(item)

    def __bool__(self):
        # cart is True if it has items, False if empty
        return len(self.items) > 0

cart = ShoppingCart()
if not cart:
    print("Cart is empty")   # will be printed

cart.add_item("book")
if cart:
    print("Cart has items")  # will be printed
```

اگر `__bool__` تعریف نشود، پایتون به سراغ `__len__` می‌رود: اگر `__len__` صفر برگرداند، شیء `False` در نظر گرفته می‌شود، در غیر این صورت `True`.

---

### ۶. کانتینرها: `__getitem__`، `__setitem__`، `__delitem__`، `__len__`، `__contains__`

این متدها به کلاس شما اجازه می‌دهند مانند لیست‌ها یا دیکشنری‌ها رفتار کند و از ایندکس‌گذاری، پیمایش و عملگر `in` پشتیبانی کند.

```python
class Playlist:
    def __init__(self):
        self._songs = []

    def add(self, song):
        self._songs.append(song)

    def __getitem__(self, index):
        # support for playlist[index]
        return self._songs[index]

    def __setitem__(self, index, value):
        # support for playlist[index] = value
        self._songs[index] = value

    def __delitem__(self, index):
        # support for del playlist[index]
        del self._songs[index]

    def __len__(self):
        # support for len(playlist)
        return len(self._songs)

    def __contains__(self, song):
        # support for "song" in playlist
        return song in self._songs

    def __iter__(self):
        # support for for song in playlist
        return iter(self._songs)

playlist = Playlist()
playlist.add("Song 1")
playlist.add("Song 2")

print(len(playlist))         # 2  ← __len__
print(playlist[0])           # Song 1  ← __getitem__
print("Song 2" in playlist)  # True  ← __contains__

for song in playlist:        # ← __iter__
    print(song)

playlist[1] = "New Song"     # ← __setitem__
del playlist[0]              # ← __delitem__
```

**نکته**: اگر `__getitem__` را تعریف کنید، پایتون به طور خودکار یک `__iter__` پیش‌فرض ایجاد می‌کند که از ایندکس‌گذاری استفاده می‌کند. اما تعریف صریح `__iter__` کنترل بیشتری به شما می‌دهد.

---

### ۷. پروتکل Iterator: `__iter__` و `__next__`

اگر می‌خواهید کنترل کامل روی نحوه‌ی پیمایش داشته باشید، می‌توانید خودتان پروتکل Iterator را پیاده‌سازی کنید.

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        # returns the iterator object (itself)
        return self

    def __next__(self):
        # returns the next value, or raises StopIteration when done
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value

for n in Countdown(5):
    print(n)   # 5, 4, 3, 2, 1
```

---

### ۸. میان‌بر پایتونی: جنریتورها با `yield`

پیاده‌سازی دستی پروتکل Iterator مانند بالا، در کد حرفه‌ای به ندرت دیده می‌شود. پایتون میان‌بری به نام **جنریتور** ارائه می‌دهد که با کلمه‌ی کلیدی `yield` کار می‌کند.

```python
def countdown(start):
    while start > 0:
        yield start   # pause here, return value, resume later
        start -= 1

for n in countdown(5):
    print(n)   # 5, 4, 3, 2, 1
```

جنریتورها در کلاس‌ها نیز قابل استفاده هستند:

```python
class Playlist:
    def __init__(self):
        self._songs = []

    def add(self, song):
        self._songs.append(song)

    def __iter__(self):
        # generator as iterator: filter out empty strings
        for song in self._songs:
            if song:   # skip empty strings
                yield song

playlist = Playlist()
playlist.add("Song 1")
playlist.add("")
playlist.add("Song 3")

for song in playlist:
    print(song)   # Song 1, Song 3 (empty one is skipped)
```

**چه زمانی از جنریتور و چه زمانی از کلاس Iterator استفاده کنیم؟**

- از **جنریتور** استفاده کنید مگر اینکه نیاز به پیاده‌سازی پیچیده‌ای داشته باشید که جنریتور توانایی آن را نداشته باشد (مانند قابلیت pause/resume یا متدهای اضافی). در ۹۹٪ موارد، جنریتور کافی و ساده‌تر است.

---

### ۹. مدیریت زمینه (Context Manager): `__enter__` و `__exit__`

این دو متد به کلاس شما اجازه می‌دهند با عبارت `with` کار کند. این الگو برای مدیریت منابع (فایل‌ها، اتصالات دیتابیس، قفل‌ها) بسیار مناسب است.

```python
class DatabaseConnection:
    def __enter__(self):
        # called when entering the 'with' block
        print("Connection opened")
        return self   # the object that will be bound to 'as'

    def __exit__(self, exc_type, exc_val, exc_tb):
        # called when exiting the 'with' block (even if an error occurred)
        print("Connection closed")
        # return False to propagate exceptions, True to suppress them
        return False

    def query(self, sql):
        return f"Result for: {sql}"

with DatabaseConnection() as db:
    print(db.query("SELECT * FROM users"))
# Output:
# Connection opened
# Result for: SELECT * FROM users
# Connection closed
```

**چرا از `with` استفاده کنیم؟**

- تضمین می‌کند که `__exit__` حتی در صورت بروز استثنا نیز اجرا شود.
- کد را خواناتر و قابل‌نگهداری‌تر می‌کند.

---

### ۱۰. متد `__call__`: شیء به عنوان تابع

با تعریف `__call__`، شیء شما قابل فراخوانی مانند یک تابع می‌شود. این قابلیت برای ساخت توابع با حافظه (stateful functions) بسیار مفید است.

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, x):
        return x * self.factor

double = Multiplier(2)
print(double(10))   # 20  ← __call__
print(callable(double))   # True

# A more practical example: rate limiter
class RateLimiter:
    def __init__(self, max_calls):
        self.max_calls = max_calls
        self.calls = 0

    def __call__(self):
        self.calls += 1
        if self.calls > self.max_calls:
            return "Blocked"
        return "Allowed"

limiter = RateLimiter(2)
print(limiter())   # Allowed
print(limiter())   # Allowed
print(limiter())   # Blocked
```

---

### ۱۱. مدیریت پویای attributeها: `__getattr__` و `__setattr__`

این متدها برای کنترل دسترسی به attributeهایی که وجود ندارند یا به صورت پویا ایجاد می‌شوند، کاربرد دارند.

**`__getattr__`**: فقط زمانی فراخوانی می‌شود که attribute به روش معمول یافت نشود. برای ارائه‌ی مقدار پیش‌فرض برای attributeهای گم‌شده مفید است.

```python
class SafeConfig:
    def __init__(self):
        self.host = "localhost"

    def __getattr__(self, name):
        # called only for missing attributes
        return f"<{name} not defined>"

config = SafeConfig()
print(config.host)      # localhost (exists)
print(config.port)      # <port not defined> (missing)
```

**`__setattr__`**: برای هر مقداردهی attribute (حتی attributeهای موجود) فراخوانی می‌شود. برای اعتبارسنجی یا ثبت‌نام تغییرات کاربرد دارد.

```python
class ValidatedAttributes:
    def __setattr__(self, name, value):
        # validation before setting
        if name == "age" and (value < 0 or value > 150):
            raise ValueError("Age must be between 0 and 150")
        # use super() to avoid infinite recursion
        super().__setattr__(name, value)

obj = ValidatedAttributes()
obj.age = 30      # OK
# obj.age = 200   # ValueError: Age must be between 0 and 150
```

**هشدار**: درون `__setattr__` هرگز از `self.name = value` استفاده نکنید، زیرا این کار دوباره `__setattr__` را فراخوانی می‌کند و به حلقه‌ی بی‌نهایت می‌انجامد. همیشه از `super().__setattr__(name, value)` استفاده کنید.

---

### ۱۲. متدهای پیشرفته‌تر: `__new__` و `__slots__`

**`__new__`**: متدی که قبل از `__init__` اجرا می‌شود و مسئول **ساخت** شیء است (در حالی که `__init__` مسئول **مقداردهی** آن است). در موارد زیر کاربرد دارد:

- ساخت شیء از کلاس‌های تغییرناپذیر مانند `tuple` یا `str`.
- پیاده‌سازی الگوی Singleton.
- هنگامی که نیاز به کنترل دقیق بر فرآیند ساخت شیء دارید.

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, value):
        self.value = value

a = Singleton(10)
b = Singleton(20)
print(a.value)   # 20 (b changed it)
print(a is b)    # True (only one instance exists)
```

**`__slots__`**: مکانیزمی برای بهینه‌سازی حافظه. با تعریف `__slots__`، به پایتون می‌گویید که فقط attributeهای مشخص‌شده را بپذیرد و از ایجاد `__dict__` برای هر شیء جلوگیری کند. این کار حافظه را به شدت کاهش می‌دهد، به ویژه وقتی تعداد زیادی شیء می‌سازید.

```python
class Point:
    __slots__ = ('x', 'y')   # only these attributes are allowed

    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
print(p.x)    # 1
# p.z = 3     # AttributeError: 'Point' object has no attribute 'z'
```

---

### ۱۳. پیش‌درآمدی بر Descriptor Protocol: `__get__`، `__set__`، `__delete__`

این سه متد، هسته‌ی **Descriptor Protocol** هستند که خود `@property`، `@classmethod` و حتی خود متدها روی آن ساخته شده‌اند. Descriptor به شما اجازه می‌دهد منطق دسترسی به attribute را در یک کلاس جداگانه و قابل استفاده‌ی مجدد کپسوله کنید.

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        # remembers the attribute name
        self._name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self._name)

    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError("Value must be positive")
        setattr(obj, self._name, value)

class Product:
    price = PositiveNumber()
    weight = PositiveNumber()

    def __init__(self, price, weight):
        self.price = price
        self.weight = weight

p = Product(100, 5)
print(p.price)    # 100
# p.price = -10   # ValueError: Value must be positive
```

مزیت Descriptor نسبت به `@property` این است که منطق اعتبارسنجی را یک بار نوشته و روی هر تعداد attribute اعمال می‌کنید. این موضوع در پروژه‌های بزرگ بسیار ارزشمند است.

---

### ۱۴. خلاصه‌ی متدهای جادویی پرکاربرد

| متد جادویی | کاربرد | مثال |
|------------|--------|-------|
| `__init__` | مقداردهی اولیه‌ی شیء | `obj = MyClass(args)` |
| `__new__` | ساخت شیء (پیش از `__init__`) | کنترل فرآیند ساخت |
| `__str__` | نمایش برای کاربر | `print(obj)` |
| `__repr__` | نمایش برای برنامه‌نویس | `repr(obj)`، نمایش در REPL |
| `__format__` | قالب‌بندی در f-string | `f"{obj:spec}"` |
| `__eq__` | برابری بر اساس مقدار | `obj1 == obj2` |
| `__hash__` | مقدار هش برای استفاده در set/dict | `hash(obj)` |
| `__bool__` | تعیین صحت در شرط‌ها | `if obj:` |
| `__add__` | جمع | `obj1 + obj2` |
| `__radd__` | جمع معکوس | `something + obj` |
| `__mul__` | ضرب | `obj1 * obj2` |
| `__getitem__` | دسترسی با ایندکس | `obj[index]` |
| `__setitem__` | مقداردهی با ایندکس | `obj[index] = value` |
| `__delitem__` | حذف با ایندکس | `del obj[index]` |
| `__len__` | طول | `len(obj)` |
| `__contains__` | عضویت | `item in obj` |
| `__iter__` | پیمایش | `for x in obj:` |
| `__next__` | مقدار بعدی در پیمایش | استفاده درونی توسط `__iter__` |
| `__enter__` | ورود به بلاک `with` | `with obj as x:` |
| `__exit__` | خروج از بلاک `with` | تضمین آزادسازی منابع |
| `__call__` | فراخوانی شیء به عنوان تابع | `obj(args)` |
| `__getattr__` | دسترسی به attribute گم‌شده | `obj.missing` |
| `__setattr__` | مقداردهی به هر attribute | `obj.attr = value` |
| `__slots__` | محدود کردن attributeها برای بهینه‌سازی حافظه | تعریف در سطح کلاس |

---

### قاعده‌ی طلایی: به اندازه‌ی نیاز استفاده کنید

متدهای جادویی قدرتمند و وسوسه‌انگیز هستند. اما **همه‌ی آنها را در همه‌ی کلاس‌ها نیاز ندارید**. فقط آن دسته از متدها را پیاده‌سازی کنید که واقعاً به یکپارچگی کلاس شما با زبان پایتون کمک می‌کنند. یک کلاس ساده که فقط `__repr__` دارد، اغلب بهتر از کلاسی است که ده متد جادویی پیاده‌سازی کرده که هیچ‌کدام واقعاً مورد استفاده قرار نمی‌گیرند.

**اصل YAGNI** (You Ain't Gonna Need It) را به خاطر داشته باشید: «به آن نیاز نخواهی داشت.» با attributeها و متدهای ساده شروع کنید و فقط در صورت نیاز، متدهای جادویی را اضافه کنید.

---

### جمع‌بندی نهایی: پل زدن میان کلاس و زبان

متدهای جادویی، پلی هستند میان کلاس‌های شما و ساختارهای زبانی پایتون. آنها به شما اجازه می‌دهند تا:

- اشیاء خود را با عملگرها (`+`, `==`, `[]`, `in`) یکپارچه کنید.
- اشیاء خود را با توابع داخلی (`len`, `print`, `iter`, `bool`) هماهنگ کنید.
- از ساختارهای زبانی (`with`, `for`, `if`) به شیوه‌ای طبیعی استفاده کنید.
- کنترل دقیقی بر نحوه‌ی دسترسی به داده‌ها داشته باشید.

این یکپارچگی، کد شما را نه تنها خواناتر، بلکه **طبیعی‌تر** می‌کند. کاربران کلاس شما (که ممکن است خودتان باشید) می‌توانند به جای یادگیری متدهای خاص، از همان ساختارهایی استفاده کنند که با آنها آشنا هستند.


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
