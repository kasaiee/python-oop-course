# فصل یازدهم: مدل‌های داده مدرن — dataclass، NamedTuple، Enum و TypedDict

---

## مقدمه

در فصل‌های قبل کلاس‌ها را دستی ساختیم: `__init__` نوشتیم، `__repr__` و `__eq__` را دستی پیاده کردیم. اما بخشِ بزرگی از کلاس‌های واقعی فقط **حاملِ داده**‌اند — رفتارِ چندانی ندارند، فقط چند فیلد را نگه می‌دارند. نوشتنِ دستیِ همه‌ی این متدها برای چنین کلاس‌هایی، کارِ تکراری و خسته‌کننده‌ای است.

پایتون مدرن ابزارهای قدرتمندی برای این کار دارد که کدِ تکراری (Boilerplate) را حذف می‌کنند: **dataclass، NamedTuple، TypedDict و Enum**. در این فصل یاد می‌گیریم هرکدام چه می‌کنند و کِی کدام را انتخاب کنیم.

---

## چرا مدل‌های داده مدرن را یاد بگیریم؟

نوشتنِ دستیِ کلاسِ حاملِ داده یعنی تکرارِ نامِ هر فیلد چند بار (در `__init__`، در `__repr__`، در `__eq__`). این هم خسته‌کننده است، هم مستعدِ خطا. این ابزارها:

- کدِ تکراری را به یک خط کاهش می‌دهند
- کد را خواناتر و مختصرتر می‌کنند
- از قابلیت‌های مدرنِ پایتون بهره می‌برند
- با فریمورک‌های مدرن (مثلِ Pydantic و FastAPI) سازگارند

---

## dataclass

### مشکل: __init__ دستی برای کلاس‌های پرفیلد

بدونِ dataclass، یک کلاسِ ساده‌ی حاملِ داده این‌قدر تکرار دارد:

```python
class Product:
    def __init__(self, name, price, stock, category):
        self.name = name
        self.price = price          # هر فیلد دو بار نوشته شد
        self.stock = stock
        self.category = category

    def __repr__(self):
        return (f"Product(name={self.name!r}, price={self.price!r}, "
                f"stock={self.stock!r}, category={self.category!r})")

    def __eq__(self, other):
        if not isinstance(other, Product):
            return NotImplemented
        return (self.name, self.price, self.stock, self.category) == \
               (other.name, other.price, other.stock, other.category)
```

نامِ هر فیلد سه‌چهار بار تکرار شد. حالا نسخه‌ی dataclass را ببینید.

### دکوراتور @dataclass

```python
from dataclasses import dataclass

@dataclass
class Product:
    name: str
    price: int
    stock: int = 0                  # مقدار پیش‌فرض
    category: str = "عمومی"

p = Product("کیبورد", 500_000)
print(p)                            # Product(name='کیبورد', price=500000, stock=0, category='عمومی')
print(p == Product("کیبورد", 500_000))   # True
```

همین. `@dataclass` در پشت‌صحنه خودکار می‌سازد: `__init__`، `__repr__`، `__eq__` و چند متدِ دیگر — همه از روی تعریفِ فیلدها با تایپ‌هینت. کدی که ۲۰ خط بود، حالا ۵ خط است.

### پارامترهای @dataclass

`@dataclass` گزینه‌های مفیدی دارد:

```python
from dataclasses import dataclass

@dataclass(frozen=True)             # تغییرناپذیر (Immutable)
class Point:
    x: int
    y: int

p = Point(1, 2)
# p.x = 5      # FrozenInstanceError — نمی‌توان تغییر داد

@dataclass(order=True)              # عملگرهای مقایسه (< > <= >=) هم ساخته می‌شوند
class Version:
    major: int
    minor: int

print(Version(1, 2) < Version(1, 5))   # True
```

- `frozen=True`: شیء را تغییرناپذیر می‌کند (مزایای Immutable از فصل قبل).
- `order=True`: متدهای مقایسه (`__lt__` و...) را می‌سازد.
- `kw_only=True` (پایتون ۳.۱۰+): همه‌ی فیلدها را keyword-only می‌کند.
- `unsafe_hash=True`: `__hash__` را حتی وقتی frozen نیست می‌سازد (با احتیاط).

نکته: `frozen=True` علاوه بر تغییرناپذیری، `__hash__` هم می‌سازد، پس شیء قابلِ استفاده در `set` و کلیدِ `dict` می‌شود.

### __post_init__

اگر بعد از مقداردهیِ خودکار به منطقِ اضافی نیاز دارید (مثلِ اعتبارسنجی یا محاسبه)، از `__post_init__` استفاده کنید:

```python
from dataclasses import dataclass

@dataclass
class Rectangle:
    width: int
    height: int
    area: int = 0

    def __post_init__(self):
        if self.width <= 0 or self.height <= 0:
            raise ValueError("ابعاد باید مثبت باشند")
        self.area = self.width * self.height     # محاسبه‌ی خودکار

r = Rectangle(3, 4)
print(r.area)      # 12
```

`__post_init__` درست بعد از `__init__`ِ خودکار اجرا می‌شود.

### InitVar

گاهی می‌خواهید مقداری را فقط به `__init__` بدهید بدونِ اینکه فیلدِ دائمیِ شیء شود. `InitVar` این کار را می‌کند:

```python
from dataclasses import dataclass, InitVar

@dataclass
class User:
    name: str
    password: InitVar[str]           # فقط برای init، فیلدِ دائمی نمی‌شود
    password_hash: str = ""

    def __post_init__(self, password):
        self.password_hash = f"hashed({password})"

u = User("ali", "1234")
print(u.password_hash)          # hashed(1234)
print(hasattr(u, "password"))   # False — password ذخیره نشد
```

### field(default_factory=...)

یادِ دامِ «پیش‌فرضِ تغییرپذیر» از فصل قبل؟ در dataclass نمی‌توانید مستقیم `items: list = []` بنویسید (پایتون خطا می‌دهد). به‌جایش از `default_factory` استفاده کنید:

```python
from dataclasses import dataclass, field

@dataclass
class Cart:
    items: list = field(default_factory=list)    # هر شیء لیستِ تازه‌ی خودش

c1 = Cart()
c2 = Cart()
c1.items.append("کتاب")
print(c2.items)      # [] — مستقل، بدون نشت
```

`default_factory=list` یعنی «برای هر نمونه، یک `list()`ِ تازه بساز» — راهِ درستِ پیش‌فرضِ تغییرپذیر. تفاوتش با `default`: `default` یک مقدارِ ثابتِ مشترک است؛ `default_factory` هر بار مقدارِ تازه می‌سازد.

### dataclass در برابر Pydantic

`@dataclass` اعتبارسنجیِ **نوع** در زمانِ اجرا انجام نمی‌دهد — تایپ‌هینت‌ها فقط راهنمایند:

```python
@dataclass
class Product:
    price: int

p = Product("رشته!")     # خطا نمی‌دهد! پایتون تایپ را اجبار نمی‌کند
print(p.price)           # رشته!
```

اینجا **Pydantic** وارد می‌شود. `Pydantic.BaseModel` شبیهِ dataclass است اما نوع را در زمانِ اجرا **اعتبارسنجی و تبدیل** می‌کند:

```python
# نیازمند نصب: pip install pydantic
from pydantic import BaseModel

class Product(BaseModel):
    price: int

p = Product(price="500")     # Pydantic خودکار به int تبدیل می‌کند
print(p.price, type(p.price))    # 500 <class 'int'>
# Product(price="سلام")      # ValidationError — قابلِ تبدیل نیست
```

**Tradeoff:** `@dataclass` سبک، بخشی از کتابخانه‌ی استاندارد، و سریع است — مناسبِ داده‌ی داخلیِ مطمئن. Pydantic سنگین‌تر است اما اعتبارسنجیِ قوی می‌دهد — مناسبِ داده‌ی ورودیِ نامطمئن (API، فرم، فایلِ کاربر). انتخاب بستگی دارد به اینکه داده از کجا می‌آید.

### dataclass(slots=True)

از پایتون ۳.۱۰، می‌توانید `__slots__` (فصل قبل) را خودکار با dataclass ترکیب کنید:

```python
from dataclasses import dataclass

@dataclass(slots=True)
class Point:
    x: int
    y: int

# معادلِ نوشتنِ __slots__ = ("x", "y") به‌صورت دستی، اما تمیزتر
```

این هم مزیتِ dataclass (کمتر کد) و هم مزیتِ `__slots__` (حافظه‌ی کمتر) را می‌دهد.

---

## NamedTuple و TypedDict

### NamedTuple

`NamedTuple` یک tuple است که فیلدهایش نام دارند — تغییرناپذیر، سبک، و مناسبِ داده‌های کوچکِ ثابت:

```python
from typing import NamedTuple

class Point(NamedTuple):
    x: int
    y: int

p = Point(1, 2)
print(p.x, p.y)      # 1 2 — دسترسی با نام
print(p[0])          # 1 — و هم مثل tuple با ایندکس
x, y = p             # قابلِ unpack مثل tuple
# p.x = 5            # AttributeError — تغییرناپذیر
```

`NamedTuple` وقتی عالی است که یک رکوردِ کوچکِ تغییرناپذیر می‌خواهید که مثل tuple هم رفتار کند (unpack، ایندکس).

### TypedDict

`TypedDict` ساختارِ یک **دیکشنری** را تعریف می‌کند — کلیدهای مشخص با انواعِ مشخص. برخلافِ dataclass، خروجی هنوز یک `dict` معمولی است، فقط تایپ‌چکر ساختارش را می‌داند:

```python
from typing import TypedDict

class UserDict(TypedDict):
    name: str
    age: int

u: UserDict = {"name": "ali", "age": 30}     # یک dict معمولی
print(u["name"])      # ali
# تایپ‌چکر می‌داند u["age"] یک int است و اگر کلیدِ اشتباه بزنید هشدار می‌دهد
```

`TypedDict` وقتی مفید است که با دیکشنری کار می‌کنید (مثلاً داده‌ی JSON) اما می‌خواهید ساختارش تایپ‌چک شود — بدونِ تبدیل به کلاس.

---

## Enum

### Enum چیست و چرا بهتر از رشته یا عدد است؟

**Enum** مجموعه‌ای از ثابت‌های نام‌دار است. به‌جای اینکه وضعیتِ سفارش را با رشته‌ی `"pending"` یا عددِ `1` نگه دارید، از Enum استفاده می‌کنید:

```python
from enum import Enum

class OrderStatus(Enum):
    PENDING = "pending"
    PAID = "paid"
    SHIPPED = "shipped"
    DELIVERED = "delivered"

order_status = OrderStatus.PENDING
print(order_status)              # OrderStatus.PENDING
print(order_status.value)        # pending
print(order_status.name)         # PENDING
```

چرا بهتر از رشته؟ مقایسه‌ی نمونه‌ها:

```python
# با رشته — شکننده و مستعدِ غلطِ املایی
if order_status == "penging":    # غلطِ املایی! هیچ خطایی نمی‌دهد، فقط False
    ...

# با Enum — امن
if order_status == OrderStatus.PENDING:    # اگر غلط بنویسید، AttributeError
    ...
```

مزایای Enum: **جلوگیری از مقادیرِ نامعتبر** (فقط اعضای تعریف‌شده مجازند)، **خطای زودهنگام** برای غلطِ املایی، **خودتوضیح‌بودن** (کد خواناتر)، و **امکانِ پیمایش** همه‌ی حالت‌ها.

### @unique

اگر می‌خواهید مطمئن شوید هیچ دو عضوی مقدارِ تکراری ندارند:

```python
from enum import Enum, unique

@unique
class Priority(Enum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3
    # URGENT = 1     # اگر این را اضافه کنید: ValueError (تکراری با LOW)
```

`@unique` تضمین می‌کند مقادیر یکتا بمانند — مفید برای جلوگیری از خطای سهوی.

### Enum در برابر دیکشنری یا ثابت‌های معمولی

می‌شد وضعیت‌ها را با متغیرهای ساده هم تعریف کرد (`PENDING = "pending"`). اما Enum مزیت‌های مهمی دارد: اعضایش **گروه‌بندیِ منطقی** دارند، **قابلِ پیمایش**اند (`for s in OrderStatus`)، **نوعِ مشخص** دارند (تایپ‌هینت `OrderStatus`)، و در برابرِ مقادیرِ نامعتبر مقاوم‌اند. برای هر مجموعه‌ای از حالت‌های محدود و مشخص (وضعیت، نقش، اولویت)، Enum انتخابِ درست است.

---

## کدام را انتخاب کنیم؟

جمع‌بندیِ تصمیم:

| ابزار | چه زمانی |
|-------|----------|
| `@dataclass` | کلاسِ حاملِ داده با احتمالِ رفتار؛ تغییرپذیر یا frozen |
| `@dataclass(frozen=True)` | داده‌ی تغییرناپذیر با امکانِ متد |
| `NamedTuple` | رکوردِ کوچکِ تغییرناپذیر که مثل tuple رفتار کند |
| `TypedDict` | ساختارِ دیکشنری (مثلِ JSON) با تایپ‌چک |
| `Enum` | مجموعه‌ی محدودی از حالت‌های نام‌دار |
| `Pydantic` | داده‌ی ورودیِ نامطمئن که اعتبارسنجی می‌خواهد |

---

## خلاصه

```
مدل‌های داده مدرن
═══════════════════════════════════════════════════
  @dataclass  ──► خودکار: __init__، __repr__، __eq__
  NamedTuple  ──► tuple نام‌دار، تغییرناپذیر
  TypedDict   ──► dict با ساختارِ تایپ‌شده
  Enum        ──► ثابت‌های نام‌دار و امن
```

| ابزار | می‌سازد | تغییرپذیری |
|-------|---------|-------------|
| `@dataclass` | متدهای کلاس خودکار | پیش‌فرض تغییرپذیر (یا frozen) |
| `NamedTuple` | tuple نام‌دار | تغییرناپذیر |
| `TypedDict` | فقط تایپِ dict | تغییرپذیر (dict معمولی) |
| `Enum` | ثابت‌های گروه‌بندی‌شده | تغییرناپذیر |

**نکته‌ی طلایی:** این ابزارها برای حذفِ کدِ تکراری‌اند، نه جایگزینِ همه‌ی کلاس‌ها. اگر کلاسِ شما رفتارِ غنی دارد (منطقِ پیچیده، متدهای زیاد)، کلاسِ معمولی بهتر است. اگر عمدتاً حاملِ داده است، dataclass. برای انتخابِ درست، بپرسید: «آیا این عمدتاً داده است یا رفتار؟» و «آیا باید تغییرپذیر باشد؟». پاسخ، ابزارِ درست را مشخص می‌کند.

---

**در فصل بعد:** سراغِ سه موضوعِ کاربردی می‌رویم که اغلب نادیده گرفته می‌شوند اما در پروژه‌های واقعی حیاتی‌اند: **استثناهای سفارشی، سریال‌سازی و Introspection**.
