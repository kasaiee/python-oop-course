# فصل پانزدهم: Descriptorها — زیر کاپوت property

---

## مقدمه

در فصل پنجم، آخر بحث متدهای جادویی، یک «پیش‌درآمد» از Descriptorها دیدید و قول دادیم روزی برگردیم. آن روز رسیده — و دلیل اینکه یک فصل کامل برایش کنار گذاشته‌ایم ساده است: **Descriptor همان جایی است که بیشتر دوره‌ها می‌بُرند و برمی‌گردند.** سرفصل‌ها تا metaclass می‌روند، اما descriptor یا حذف می‌شود یا در دو پاراگراف سربسته رد می‌شود. حیف است؛ چون این سازوکار، موتور پنهان نیمی از چیزهایی است که تا حالا استفاده کرده‌اید.

ادعای بزرگی است، پس همین اول مدرک رو می‌کنیم: `@property` یک descriptor است. `@classmethod` و `@staticmethod` هم. `@cached_property` هم. حتی **متدهای معمولی** — همین که `obj.method()` کار می‌کند و `self` خودش پر می‌شود — descriptor است. فیلدهای مدل در Django و SQLAlchemy؟ descriptor. وقتی این فصل تمام شود، به `obj.attr` — ساده‌ترین عبارت پایتون — با چشم دیگری نگاه می‌کنید: پشت آن نقطه‌ی کوچک، یک پروتکل کامل کار می‌کند و شما بلدید سوارش شوید.

پیش‌نیاز این فصل، فصل پنجم (property و مفهوم dunder) و فصل قبل (این‌که attributeها در `__dict__` زندگی می‌کنند) است. اگر آن دو را خوانده باشید، این فصل — با همه‌ی شهرت ترسناکش — قدم‌به‌قدم و منطقی پیش می‌رود.

---

## چرا باید Descriptorها را یاد بگیریم؟

دلیل کاربردی‌اش را با یک درد شروع کنیم که از فصل پنجم می‌شناسید. فرض کنید در سیستم فروشگاه، چند فیلد باید «عدد مثبت» باشند: قیمت، وزن، موجودی. با `@property` برای هرکدام باید یک جفت getter/setter بنویسید:

```python
class Product:
    def __init__(self, price, weight, stock):
        self.price = price
        self.weight = weight
        self.stock = stock

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value <= 0:
            raise ValueError("price must be positive")
        self._price = value

    @property
    def weight(self):
        return self._weight

    @weight.setter
    def weight(self, value):
        if value <= 0:
            raise ValueError("weight must be positive")
        self._weight = value

    # ... and the same thing again for stock. and for the next class. and...
```

سه فیلد، سی خط. منطق واحد «باید مثبت باشد» سه بار کپی شد — نقض آشکار DRY. و اگر ده مدل با فیلدهای عددی داشته باشید؟ Descriptor دقیقاً برای همین ساخته شده: **منطق دسترسی به attribute را یک‌بار بنویس، همه‌جا وصل کن.** در پایان فصل، آن سی خط به این می‌رسد:

```python
class Product:
    price = Positive()
    weight = Positive()
    stock = Positive()
```

و دلیل عمیق‌ترش: بدون descriptor، فهمتان از پایتون یک طبقه‌ی خالی وسطش دارد. می‌دانید `@property` *چه می‌کند*، اما نمی‌دانید *چطور*. می‌دانید متدها `self` می‌گیرند، اما نمی‌دانید *چه کسی آن را می‌گذارد*. مصاحبه‌های سطح سنیور عاشق همین طبقه‌اند.

---

## پروتکل Descriptor: سه متد و یک قرارداد

قرارداد ساده‌تر از شهرتش است. **Descriptor یعنی هر کلاسی که دست‌کم یکی از این متدها را داشته باشد:**

```python
class MyDescriptor:
    def __get__(self, instance, owner): ...      # reading the attribute
    def __set__(self, instance, value): ...      # assigning to it
    def __delete__(self, instance): ...          # deleting it
```

و یک قانون طلایی که همه‌چیز به آن برمی‌گردد: **descriptor باید attribute کلاس باشد، نه instance.** پایتون فقط وقتی این متدها را صدا می‌زند که descriptor را در *کلاس* پیدا کند.

ساده‌ترین نمونه‌ی ممکن، فقط برای دیدن مکانیک:

```python
class Loud:
    def __get__(self, instance, owner):
        print(f"__get__ called! instance={instance}, owner={owner.__name__}")
        return 42

class Config:
    debug = Loud()          # descriptor lives ON THE CLASS

c = Config()
print(c.debug)
# __get__ called! instance=<__main__.Config object ...>, owner=Config
# 42
print(Config.debug)
# __get__ called! instance=None, owner=Config
# 42
```

خط `c.debug` را ببینید: به‌جای اینکه مقدار ذخیره‌شده‌ای برگردد، **پایتون متد `__get__` را صدا زد.** پارامترها:

- `self` — خود شیء descriptor (`Loud()`).
- `instance` — شیئی که attribute رویش خوانده شده (`c`)؛ و اگر از روی خود کلاس خوانده شود، `None`.
- `owner` — کلاس صاحب (`Config`).

همین. `__set__` هم قرینه‌اش است: `c.debug = x` به `__set__(self, c, x)` ترجمه می‌شود. حالا که مکانیک را دیدید، بیایید چیز به‌دردبخور بسازیم.

---

## ساخت اولین Descriptor واقعی: اعتبارسنج

همان درد «عدد مثبت» را حل می‌کنیم. به `__set_name__` دقت کنید — قطعه‌ای که پازل را کامل می‌کند:

```python
class Positive:
    def __set_name__(self, owner, name):
        # called automatically at class definition time:
        # Positive() assigned to "price" -> name = "price"
        self.storage = "_" + name                    # e.g. "_price"

    def __get__(self, instance, owner):
        if instance is None:
            return self                              # accessed on the class itself
        return getattr(instance, self.storage)

    def __set__(self, instance, value):
        if value <= 0:
            raise ValueError(f"{self.storage[1:]} must be positive, got {value}")
        setattr(instance, self.storage, value)

class Product:
    price = Positive()
    weight = Positive()
    stock = Positive()

    def __init__(self, price, weight, stock):
        self.price = price          # goes through Positive.__set__!
        self.weight = weight
        self.stock = stock

p = Product(500_000, 2, 10)
print(p.price)          # 500000
p.stock = 7             # validated again on every assignment
# p.weight = -1         # ValueError: weight must be positive, got -1
# Product(0, 1, 1)      # ValueError: price must be positive, got 0
```

چند نکته‌ی مهم در همین کد کوتاه:

**`__set_name__` مشکل «اسمم چیست؟» را حل می‌کند.** descriptor باید بداند داده را زیر چه نامی در instance ذخیره کند. پایتون (از ۳.۶ به بعد) هنگام تعریف کلاس، خودش به هر descriptor خبر می‌دهد: «تو به نام `price` وصل شدی.» بدون آن مجبور بودیم نام را دستی تکرار کنیم: `price = Positive("price")`.

**داده کجا ذخیره می‌شود؟ در خود instance** (زیر نام `_price`)، نه در descriptor. این حیاتی است: descriptor بین *همه‌ی* اشیاء کلاس مشترک است (یک `Positive()` برای هزار `Product`)؛ اگر داده را در descriptor نگه دارید، همه‌ی محصولات یک قیمت پیدا می‌کنند — همان دام class variable فصل دوم در لباس جدید.

**و در `__init__` هیچ خبری از اعتبارسنجی نیست.** خط `self.price = price` خودش از `__set__` رد می‌شود. منطق یک‌جاست و همه‌ی مسیرها — سازنده، تغییر بعدی، همه — از همان یک در عبور می‌کنند.

یک اعتبارسنج دیگر بسازیم تا الگو جا بیفتد — این‌بار با پارامتر:

```python
class BoundedString:
    def __init__(self, max_length):
        self.max_length = max_length

    def __set_name__(self, owner, name):
        self.storage = "_" + name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.storage)

    def __set__(self, instance, value):
        if not isinstance(value, str):
            raise TypeError(f"{self.storage[1:]} must be a string")
        if len(value) > self.max_length:
            raise ValueError(f"{self.storage[1:]} is longer than {self.max_length}")
        setattr(instance, self.storage, value)

class User:
    username = BoundedString(max_length=30)
    bio = BoundedString(max_length=200)

    def __init__(self, username, bio=""):
        self.username = username
        self.bio = bio

u = User("ali")
# u.username = "x" * 50    # ValueError: username is longer than 30
```

اگر این ساختار برایتان آشناست، باید هم باشد — `models.CharField(max_length=30)` در Django دقیقاً همین است. تا پایان پروژه‌ی چهارم، خودتان یکی می‌سازید.

---

## ترتیب جست‌وجوی attribute: قانونی که همه‌چیز را توضیح می‌دهد

حالا عمیق‌ترین بخش فصل — جایی که پازل چند فصل گذشته یک‌جا حل می‌شود. سوال: وقتی می‌نویسید `obj.x`، پایتون به چه ترتیبی کجاها را می‌گردد؟

اول دو اصطلاح: descriptorی که `__set__` (یا `__delete__`) دارد، **data descriptor** است (مثل `property` و `Positive` ما)؛ descriptorی که فقط `__get__` دارد، **non-data descriptor** (مثل توابع/متدها و `cached_property`). حالا قانون:

```
ترتیب جست‌وجوی obj.x  (ساده‌شده‌ی سازوکار واقعی)
═══════════════════════════════════════════════════
۱. در کلاس (و MRO) بگرد: اگر x یک data descriptor بود
       ← __get__ همان صدا زده می‌شود. (برنده‌ی همیشگی)
۲. وگرنه: اگر x در __dict__ خود instance بود
       ← همان برگردانده می‌شود.
۳. وگرنه: اگر x در کلاس (و MRO) بود:
       ← اگر non-data descriptor بود، __get__اش صدا زده می‌شود؛
       ← وگرنه خود مقدار برمی‌گردد.
۴. وگرنه: __getattr__ (اگر تعریف شده باشد — فصل پنجم)، وگرنه AttributeError.
```

این جدول کوچک، سه معمای قدیمی را یک‌جا جواب می‌دهد:

**معمای یک: چرا نمی‌شود property را با نوشتن در instance دور زد؟** چون property یک data descriptor است — پله‌ی ۱ — و *همیشه* از `__dict__` instance جلوتر بررسی می‌شود. اثباتش:

```python
class Account:
    @property
    def balance(self):
        return 100

a = Account()
a.__dict__["balance"] = 999_999     # sneak a value directly into the instance dict
print(a.balance)                    # 100 — the property still wins!
```

**معمای دو: چرا متدها را می‌شود در instance «سایه» زد؟** چون توابع فقط `__get__` دارند — non-data descriptor — پله‌ی ۳، که *بعد* از `__dict__` instance می‌آید:

```python
class Greeter:
    def hello(self):
        return "hello from the class"

g = Greeter()
g.hello = lambda: "hello from the instance"    # shadowing works here
print(g.hello())                               # hello from the instance
```

**معمای سه (الماس این فصل): `cached_property` چطور کار می‌کند؟** فصل پنجم دیدید که بار اول محاسبه می‌کند و بعد «رایگان» می‌شود. حالا رازش: `cached_property` عمداً non-data است (فقط `__get__` دارد). بار اول، `__get__` نتیجه را حساب می‌کند و **در `__dict__` خود instance می‌نویسد**. از آن به بعد، پله‌ی ۲ زودتر از پله‌ی ۳ می‌رسد و مقدار کش‌شده مستقیم برمی‌گردد — descriptor دیگر هرگز صدا زده نمی‌شود. نه شرطی، نه پرچمی؛ کش‌شدن را خود قانون جست‌وجو انجام می‌دهد. پیاده‌سازی‌اش را خودمان بنویسیم:

```python
class my_cached_property:
    def __init__(self, func):
        self.func = func

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        print(f"(computing {self.name}...)")
        value = self.func(instance)
        instance.__dict__[self.name] = value     # the trick: write into the instance
        return value                             # next time, step 2 wins — we're bypassed

class Report:
    @my_cached_property
    def summary(self):
        return "heavy result"

r = Report()
print(r.summary)     # (computing summary...) heavy result
print(r.summary)     # heavy result — silent. our __get__ never ran again.
```

هفده خط، و یکی از محبوب‌ترین ابزارهای پایتون را از صفر ساختید. این «فهمیدن از درون» همان چیزی است که این دوره را از حفظ سرفصل جدا می‌کند.

---

## اعتراف بزرگ: متدها، property و classmethod همه Descriptorاند

حالا وقت آن مدرکی است که اول فصل قول دادیم.

**توابع descriptorاند.** هر تابعی در پایتون متد `__get__` دارد. وقتی تابعی را در کلاس می‌گذارید و بعد `obj.method` می‌خوانید، پله‌ی ۳ فعال می‌شود: `__get__` تابع صدا زده می‌شود و چیزی برمی‌گرداند که «bound method» می‌نامیمش — همان تابع، با `self` از پیش پرشده:

```python
class Order:
    def total(self):
        return 100

o = Order()
print(Order.total)                    # <function Order.total ...> — a plain function
print(o.total)                        # <bound method Order.total of ...> — bound!
print(o.total())                      # 100 — self was pre-filled

# what the dot actually did behind the scenes:
bound = Order.total.__get__(o, Order)
print(bound())                        # 100 — the exact same thing
```

راز `self` — که از فصل دوم با آن زندگی کرده‌اید — همین سه خط آخر است: **نقطه، `__get__` تابع را صدا می‌زند و `__get__`، instance را به شکم تابع می‌بندد.** جادویی در کار نبود؛ پروتکل بود.

**property هم descriptor دست‌ساز خودمان است.** برای اثبات، یک نسخه‌ی ساده‌اش (فقط getter/setter) را بسازیم:

```python
class my_property:
    def __init__(self, fget=None, fset=None):
        self.fget = fget
        self.fset = fset

    def __get__(self, instance, owner):
        if instance is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(instance)

    def __set__(self, instance, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(instance, value)

    def setter(self, fset):
        return my_property(self.fget, fset)

class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius

    @my_property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("below absolute zero!")
        self._celsius = value

t = Temperature(25)
print(t.celsius)       # 25
t.celsius = 30         # setter runs, validation active
# t.celsius = -300     # ValueError: below absolute zero!
```

همان رفتار فصل پنجم، با کلاس خودمان. توجه کنید `@my_property` چیزی جز «تابع را بده به سازنده‌ی descriptor» نیست — دکوراتور و descriptor اینجا دست در دست هم‌اند. (نسخه‌ی واقعی `deleter` و docstring هم دارد؛ اسکلت همین است.)

**classmethod و staticmethod؟** همین الگو، با `__get__`های متفاوت: `staticmethod.__get__` تابع را دست‌نخورده برمی‌گرداند (نه self می‌بندد نه cls)؛ `classmethod.__get__` به‌جای instance، **کلاس** را می‌بندد. سه دکوراتور «جادویی» فصل پنجم، سه سیاست مختلف یک پروتکل‌اند.

---

## Descriptorها در دنیای واقعی: فیلد ORM

بیایید همه‌ی فصل را در یک مثال جمع‌بندی ببندیم که پیش‌نمایش مستقیم پروژه‌ی چهارم است: فیلدهای یک ORM خانگی. می‌خواهیم مدل این شکلی تعریف شود:

```python
class TypedField:
    """A descriptor that enforces a type and tracks changes - an ORM building block."""

    def __init__(self, field_type, default=None):
        self.field_type = field_type
        self.default = default

    def __set_name__(self, owner, name):
        self.name = name
        self.storage = "_" + name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.storage, self.default)

    def __set__(self, instance, value):
        if not isinstance(value, self.field_type):
            raise TypeError(
                f"{self.name} expects {self.field_type.__name__}, "
                f"got {type(value).__name__}"
            )
        setattr(instance, self.storage, value)
        # ORM-flavored bonus: remember which fields changed (for UPDATE queries)
        if not hasattr(instance, "_dirty"):
            instance._dirty = set()
        instance._dirty.add(self.name)

class Product:
    name = TypedField(str, default="")
    price = TypedField(int, default=0)

p = Product()
p.name = "keyboard"
print(p.name, p.price)      # keyboard 0    (price fell back to its default)
print(p._dirty)             # {'name'}      - the ORM knows what to UPDATE
# p.price = "cheap"         # TypeError: price expects int, got str
```

فیلدی که تایپ را تحمیل می‌کند، مقدار پیش‌فرض دارد و تغییرات را برای `UPDATE` ردیابی می‌کند — این دیگر تمرین آموزشی نیست؛ مکانیزم واقعی SQLAlchemy و Django است در مقیاس کوچک. در پروژه‌ی چهارم همین را با `__init_subclass__` (فصل ششم) و متاکلاس (فصل بعد) کامل می‌کنیم تا مدل‌ها خودشان جدول و کوئری بسازند.

---

## کی Descriptor بسازیم؟ (و کی نه)

مثل همیشه، قدرت بیشتر یعنی مسئولیت بیشتر. قاعده‌ی تصمیم‌گیری:

- **یک فیلد در یک کلاس** منطق دسترسی می‌خواهد → `@property`. تمام. descriptor نسازید.
- **یک منطق تکراری روی چند فیلد یا چند کلاس** (اعتبارسنجی، تبدیل، lazy، ردیابی) → descriptor دقیقاً برای همین است.
- **ساختن فریمورک یا کتابخانه** که کاربرانش با `field = Something()` مدل تعریف می‌کنند → descriptor + `__set_name__` زبان مادری این کار است.

و دو هشدار از جنس کد واقعی: descriptorها **در کلاس مشترک‌اند** — هر حالتی که نگه می‌دارید یا per-instance ذخیره‌اش کنید (مثل `_price` ما) یا عمداً مشترک بخواهیدش؛ این پرتکرارترین باگ descriptorنویس‌های تازه‌کار است. و descriptor برای خواننده‌ی ناآشنا «نامرئی» است — `p.price = -1` که خطا می‌دهد، برای کسی که کلاس را نمی‌شناسد غافلگیرکننده است؛ پس نام‌گذاری روشن (`Positive`, `BoundedString`) و docstring اینجا ادب نیست، ضرورت است.

---

## خلاصه

| مفهوم | یک‌خطی |
|-------|---------|
| پروتکل descriptor | کلاسی با `__get__` / `__set__` / `__delete__` که به‌عنوان attribute *کلاس* می‌نشیند |
| data descriptor | `__set__` دارد؛ از `__dict__` instance **جلوتر** بررسی می‌شود (property) |
| non-data descriptor | فقط `__get__` دارد؛ از `__dict__` instance **عقب‌تر** است (متدها، cached_property) |
| `__set_name__` | در زمان تعریف کلاس، نام attribute را به descriptor خبر می‌دهد |
| ذخیره‌سازی | داده در instance بماند (`_name`)، نه در descriptor مشترک |
| راز self | `obj.method` یعنی `func.__get__(obj, cls)` — نقطه، bound method می‌سازد |
| راز cached_property | non-data بودن + نوشتن در `__dict__` = دورزدن خودکار از بار دوم |

**نکته‌ی طلایی:** از این به بعد هر جا دیدید attributeی «رفتار خاص» دارد — اعتبارسنجی می‌کند، دیر بار می‌شود، در ORM به ستون وصل است — لازم نیست حدس بزنید؛ می‌دانید زیرش یک descriptor نشسته و می‌دانید سه متدش را کجا پیدا کنید.

---

## پرسش‌های مصاحبه

**۱. «descriptor چیست و property چه رابطه‌ای با آن دارد؟»** — پاسخ کامل: کلاسی با `__get__`/`__set__`/`__delete__` که به‌صورت attribute کلاس می‌نشیند و پایتون دسترسی به attribute را به متدهایش واگذار می‌کند؛ `property` صرفاً یک data descriptor آماده است که getter/setter/deleter شما را صدا می‌زند. اگر پیاده‌سازی ده‌خطی `my_property` را هم بنویسید، مصاحبه عملاً تمام است.

**۲. «فرق data و non-data descriptor چیست و چه اثری دارد؟»** — data (`__set__`دار) بر `__dict__` instance مقدم است، پس property را نمی‌شود با نوشتن مستقیم در instance سایه زد؛ non-data (فقط `__get__`) مغلوب `__dict__` است، برای همین متدها را می‌شود سایه زد و `cached_property` از همین قانون برای کش خودکار استفاده می‌کند.

**۳. «وقتی می‌نویسیم obj.method() چه اتفاقی می‌افتد که self پر می‌شود؟»** — توابع descriptorاند؛ نقطه باعث می‌شود `function.__get__(obj, cls)` اجرا شود و یک bound method برگردد که instance را به پارامتر اول تابع بسته. `obj.method()` یعنی `type(obj).method.__get__(obj, type(obj))()`.

**۴. «در descriptorی که برای اعتبارسنجی نوشته‌اید، داده را کجا ذخیره می‌کنید و چرا؟»** — در خود instance (مثلاً زیر `_name` که `__set_name__` ساخته)، چون descriptor یک شیء مشترک بین همه‌ی نمونه‌های کلاس است؛ ذخیره در descriptor یعنی همه‌ی نمونه‌ها یک مقدار — باگی که در کدهای واقعی بارها دیده شده.

---

**در فصل بعد:** descriptorها کنترل «دسترسی به attribute» را به ما دادند. حالا یک پله بالاتر: کنترل «ساخته‌شدن خود کلاس‌ها». **متاکلاس‌ها** — جایی که `type` از جادو به ابزار تبدیل می‌شود و می‌فهمید Django موقع تعریف هر مدل، پشت پرده چه می‌کند.
