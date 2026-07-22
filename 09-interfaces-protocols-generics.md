## فصل نهم: اینترفیس‌ها، پروتکل‌ها و ژنریک‌ها

### مقدمه

در فصل گذشته، با اصول SOLID آشنا شدیم و در خلال آن، بارها با واژه‌های «انتزاع» و «اینترفیس» روبرو شدیم. اما آن فصل، ابزارهای عینی و عملی پایتون برای پیاده‌سازی این مفاهیم را معرفی نکرد. این فصل، دقیقاً برای پر کردن همین خلأ طراحی شده است.

پایتون سه ابزار اصلی برای ایجاد انتزاع و قرارداد در اختیار برنامه‌نویس قرار می‌دهد که هر یک کاربرد خاص خود را دارند:

۱. **کلاس‌های انتزاعی (Abstract Base Classes — ABC)**: برای تعریف اینترفیس‌های اسمی (Nominal) که در آنها، یک کلاس باید به صراحت از یک پایه‌ی انتزاعی ارث‌ببرد تا قرارداد را بپذیرد.

۲. **پروتکل‌ها (Protocols)**: برای تعریف اینترفیس‌های ساختاری (Structural) که در آنها، صرفاً وجود متدها و ویژگی‌های خاص در یک شیء کافی است تا آن شیء قرارداد را برآورده کند؛ بدون نیاز به ارث‌بری.

۳. **ژنریک‌ها (Generics)**: برای نوشتن کدهایی که با انواع داده‌ی مختلف کار می‌کنند، بدون اینکه نیاز به تکرار کد یا از دست دادن ایمنی نوع داشته باشند.

در انتهای این فصل، با تایپ‌هینترهای پیشرفته‌ای نیز آشنا خواهید شد که ابزارهای تحلیل ایستای کد (مانند mypy) را قادر می‌سازند تا خطاهای مربوط به نوع داده را پیش از اجرای برنامه شناسایی کنند. فرض بر این است که شما پیش‌تر با مفاهیم پایه‌ی تایپ‌هینترها در پایتون آشنا شده‌اید.

### چرا این ابزارها را یاد بگیریم؟

بسیاری از برنامه‌نویسان پایتون، سال‌ها بدون استفاده از ABC یا Protocol کد می‌نویسند و برنامه‌هایشان به درستی کار می‌کند. اما وقتی کد یک کتابخانه‌ی بزرگ و حرفه‌ای (مانند جنگو، SQLAlchemy یا Pydantic) را باز می‌کنید، به وفور با این مفاهیم روبرو می‌شوید. درک این ابزارها، شما را به سطح بالاتری از طراحی نرم‌افزار می‌رساند و مزایای زیر را به همراه دارد:

- نوشتن کدی که **به راحتی قابل تست** است، زیرا وابستگی‌ها را می‌توان با پیاده‌سازی‌های جایگزین (ساختگی) تعویض کرد. (همان اصل وارونگی وابستگی که در فصل قبل دیدیم).
- استفاده از تایپ‌هینترها برای **گرفتن باگ‌ها پیش از اجرا**، که هزینه‌ی اصلاح خطا را به شدت کاهش می‌دهد.
- طراحی کتابخانه‌ها و ماژول‌هایی که **قابل توسعه** هستند و دیگران می‌توانند به راحتی قابلیت‌های جدید به آنها اضافه کنند.
- پیاده‌سازی **معماری‌های تمیز** که در آن لایه‌های مختلف کد، به جای وابستگی به جزئیات، به قراردادها وابسته هستند.

---

## Abstract Base Classes (ABC)

### تعریف و مفهوم پایه

یک **کلاس انتزاعی (Abstract Base Class)**، کلاسی است که به تنهایی قابل نمونه‌سازی (ایجاد شیء) نیست و صرفاً به عنوان **قالبی برای سایر کلاس‌ها** طراحی می‌شود. این کلاس، یک **قرارداد** را تعریف می‌کند که هر کلاس مشتق‌شده‌ای (زیرکلاسی) موظف به رعایت آن است. این قرارداد، شامل مجموعه‌ای از **متدهای انتزاعی** است که در کلاس پایه، تنها نام و امضا (پارامترها) را دارند و بدنه‌ی آنها خالی است. زیرکلاس‌ها باید این متدها را با منطق مناسب خود پیاده‌سازی کنند.

**چرا به ABC نیاز داریم؟**

تصور کنید کلاسی به نام `PaymentGateway` طراحی کرده‌اید و انتظار دارید که همه‌ی درگاه‌های پرداخت (مانند زرین‌پال، stripe و...) متدهای `pay` و `refund` را داشته باشند. اگر از ABC استفاده نکنید، ممکن است یک توسعه‌دهنده، زیرکلاسی بنویسد و تنها متد `pay` را پیاده‌سازی کند و `refund` را فراموش کند. در این حالت، خطا تنها زمانی رخ می‌دهد که کد سعی کند متد `refund` را صدا بزند — شاید در محیط تولید و با هزینه‌ی بسیار بالا.

اما با استفاده از ABC، به محض اینکه تلاش کنید یک شیء از آن زیرکلاس را بسازید، اگر همه‌ی متدهای انتزاعی را پیاده‌سازی نکرده باشد، پایتون بلافاصله یک خطای `TypeError` پرتاب می‌کند. این یعنی **شکست زودهنگام** (Fail-Fast) که ارزش بسیار بالایی در مهندسی نرم‌افزار دارد.

### نحوه‌ی تعریف ABC

برای تعریف یک کلاس انتزاعی در پایتون، باید از ماژول `abc` و دو ابزار آن استفاده کنیم:

- `ABC`: کلاس پایه‌ای که کلاس انتزاعی شما باید از آن ارث‌ببرد.
- `abstractmethod`: دکوراتوری که روی متدهای انتزاعی قرار می‌گیرد.

```python
from abc import ABC, abstractmethod

class PaymentGateway(ABC):
    @abstractmethod
    def pay(self, amount: float) -> str:
        pass

    @abstractmethod
    def refund(self, amount: float) -> str:
        pass

class StripeGateway(PaymentGateway):
    def pay(self, amount: float) -> str:
        return f"Stripe: paid {amount}"

    def refund(self, amount: float) -> str:
        return f"Stripe: refunded {amount}"

# Creating an instance of an abstract class is impossible:
# gateway = PaymentGateway()  # TypeError!

# Creating an instance of a complete subclass is possible:
stripe = StripeGateway()
print(stripe.pay(100))   # Stripe: paid 100
```

**نکته‌ی کلیدی**: اگر زیرکلاسی یک یا چند متد انتزاعی را پیاده‌سازی نکند، آن زیرکلاس نیز خود به یک کلاس انتزاعی تبدیل می‌شود و نمی‌توان از آن نمونه ساخت:

```python
class BrokenGateway(PaymentGateway):
    def pay(self, amount: float) -> str:
        return f"Broken: paid {amount}"
    # refund method is not implemented

# Error occurs at object creation time, not when calling refund!
# bg = BrokenGateway()  # TypeError: Can't instantiate abstract class BrokenGateway
```

### چه زمانی از ABC استفاده کنیم و چه زمانی نه؟

**زمان استفاده از ABC:**
- زمانی که می‌خواهید **خانواده‌ای از کلاس‌ها** را ایجاد کنید که همه باید یک قرارداد مشخص را رعایت کنند.
- زمانی که رابطه‌ی **«Is-A»** به درستی برقرار است؛ مثلاً «درگاه پرداخت زرین‌پال، یک نوع درگاه پرداخت است».
- زمانی که می‌خواهید **پیاده‌سازی مشترکی** را در کلاس پایه قرار دهید و زیرکلاس‌ها تنها بخش‌های خاص را بازنویسی کنند.

**زمان عدم استفاده از ABC:**
- زمانی که فقط می‌خواهید بگویید «هر چیزی که این متدها را داشته باشد، قابل قبول است» و نیازی به ارث‌بری اجباری ندارید. در این موارد، **Protocol** انتخاب بهتری است.
- زمانی که با کلاس‌هایی روبرو هستید که از کتابخانه‌های دیگر آمده‌اند و نمی‌توانید آنها را تغییر دهید تا از ABC شما ارث‌ببرند. باز هم Protocol راه‌حل مناسبی است.

---

## Protocolها (Structural Subtyping)

### Protocol چیست و چه مسئله‌ای را حل می‌کند؟

**Protocol** که از پایتون نسخه‌ی ۳.۸ به بعد در ماژول `typing` معرفی شده است، راهی برای تعریف اینترفیس بر اساس **ساختار** شیء ارائه می‌دهد، نه بر اساس ارث‌بری. با Protocol، شما می‌گویید: «هر شیءای که این متدها و ویژگی‌ها را داشته باشد، این قرارداد را برآورده می‌کند» — بدون اینکه آن شیء لازم باشد از یک کلاس خاص ارث‌ببرد.

این رویکرد، با فلسفه‌ی اصلی پایتون یعنی **Duck Typing** هماهنگ است. در Duck Typing، به جای اینکه بپرسید «این شیء از چه کلاسی است؟»، می‌پرسید «این شیء چه کاری می‌تواند انجام دهد؟». Protocol، این رویکرد را به سطح تایپ‌هینترها می‌آورد و به ابزارهای تحلیل ایستا (مانند mypy) اجازه می‌دهد تا صحت نوع‌ها را بررسی کنند.

### مثال عملی Protocol

فرض کنید می‌خواهیم تابعی بنویسیم که هر شیء قابل ترسیم (Drawable) را بگیرد و آن را رندر کند. اشیاء مختلفی در سیستم ما وجود دارند که قابلیت ترسیم دارند، اما ممکن است از یک کلاس پایه مشتق نشده باشند:

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> str:
        ...

class Button:
    def draw(self) -> str:
        return "[Button]"

class Icon:
    def draw(self) -> str:
        return "(Icon)"

class TextBox:
    def draw(self) -> str:
        return "|TextBox|"

def render(item: Drawable) -> None:
    print(item.draw())

# All of these objects are accepted without inheriting from Drawable
render(Button())   # [Button]
render(Icon())     # (Icon)
render(TextBox())  # |TextBox|
```

در این مثال، کلاس‌های `Button`، `Icon` و `TextBox` هیچ‌کدام از `Drawable` ارث‌نبرده‌اند، اما همگی متد `draw` را دارند. تابع `render` هر شیءای که این ساختار را داشته باشد می‌پذیرد. ابزارهای تایپ‌چک، اگر شیءای بدون متد `draw` به `render` داده شود، پیش از اجرا هشدار می‌دهند.

### تفاوت‌های کلیدی ABC و Protocol

| ویژگی | ABC | Protocol |
| :--- | :--- | :--- |
| **نوع رابطه** | **اسمی (Nominal)**: شیء باید به صراحت از کلاس پایه ارث‌ببرد. | **ساختاری (Structural)**: تنها شکل شیء (وجود متدها) مهم است. |
| **نیاز به ارث‌بری** | بله، کلاس باید از `ABC` یا کلاس انتزاعی دیگری ارث‌ببرد. | خیر، هیچ نیازی به ارث‌بری نیست. |
| **بررسی در زمان ساخت** | بله، اگر متدی پیاده‌سازی نشده باشد، `TypeError` می‌گیرید. | خیر، Protocol فقط برای تایپ‌چک ایستا طراحی شده است. |
| **مناسب برای** | کلاس‌هایی که خودتان طراحی می‌کنید و کنترل کاملی دارید. | کلاس‌هایی که از کتابخانه‌های دیگر می‌آیند یا نمی‌خواهید ارث‌بری تحمیل کنید. |
| **پیاده‌سازی مشترک** | می‌توانید متدهای معمولی (غیرانتزاعی) با پیاده‌سازی پیش‌فرض داشته باشید. | نمی‌توانید پیاده‌سازی ارائه دهید؛ فقط قرارداد را تعریف می‌کنید. |

### runtime_checkable: بررسی در زمان اجرا

به طور پیش‌فرض، Protocolها فقط برای تحلیل ایستا (با mypy و مانند آن) کاربرد دارند و تابع `isinstance` روی آنها کار نمی‌کند. اما با استفاده از دکوراتور `@runtime_checkable`، می‌توانید این قابلیت را فعال کنید:

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Closeable(Protocol):
    def close(self) -> None:
        ...

class Connection:
    def close(self) -> None:
        print("Connection closed")

class Resource:
    def release(self) -> None:  # does not have a close method
        print("Resource released")

conn = Connection()
res = Resource()

print(isinstance(conn, Closeable))  # True  (because it has the close method)
print(isinstance(res, Closeable))   # False (because it does not have the close method)
```

**نکته‌ی مهم**: `isinstance` روی Protocolهای runtime_checkable، صرفاً **وجود** متدها را بررسی می‌کند و نه امضا (پارامترها) یا رفتار آنها را. بنابراین، یک شیء ممکن است Protocol را برآورده کند اما در عمل، رفتار مورد انتظار را نداشته باشد. برای اطمینان بیشتر، همچنان به تایپ‌چک‌های ایستا تکیه کنید.

### چه زمانی Protocol و چه زمانی ABC؟

این یک سوال رایج و مهم است. قاعده‌ی کلی به این صورت است:

- **از ABC استفاده کنید** زمانی که:
  - شما کنترل کاملی بر سلسله‌مراتب کلاس‌ها دارید و می‌خواهید یک قرارداد را به زور تحمیل کنید.
  - می‌خواهید از **شکست زودهنگام** بهره‌مند شوید (خطا در زمان ساخت شیء).
  - می‌خواهید یک **پیاده‌سازی مشترک** را در کلاس پایه قرار دهید و زیرکلاس‌ها تنها بخش‌های خاص را بازنویسی کنند.

- **از Protocol استفاده کنید** زمانی که:
  - می‌خواهید با کلاس‌هایی کار کنید که کنترل آنها را ندارید (مثل کلاس‌های کتابخانه‌های خارجی).
  - نمی‌خواهید ارث‌بری را به کاربران کتابخانه‌ی خود تحمیل کنید؛ می‌خواهید هر شیء با ساختار درست، پذیرفته شود.
  - با فلسفه‌ی Duck Typing پایتون هماهنگ‌تر هستید و می‌خواهید قرارداد را بر اساس رفتار تعریف کنید، نه هویت.

**نکته‌ی طلایی**: اگر شک دارید، از **Protocol** شروع کنید، زیرا انعطاف‌پذیرتر و با روح پایتون سازگارتر است. فقط در مواردی که نیاز به تحمیل قرارداد و شکست زودهنگام دارید، به ABC مهاجرت کنید.

---

## ژنریک‌ها (Generics)

### مشکل تکرار کد برای انواع مختلف

فرض کنید می‌خواهیم یک کلاس «مخزن» (Repository) طراحی کنیم که بتواند اشیاء از انواع مختلف را در خود نگهداری کند. بدون استفاده از ژنریک، سه راه پیش رو داریم:

۱. برای هر نوع (User، Product، Order و...) یک کلاس مخزن جداگانه بنویسیم. این کار، تکرار کد را به شدت افزایش می‌دهد.

۲. از نوع `Any` (در تایپ‌هینتر) استفاده کنیم که هر چیزی را می‌پذیرد، اما ایمنی نوع را کاملاً از بین می‌برد و تایپ‌چکر نمی‌تواند خطاهای مربوط به نوع را تشخیص دهد.

۳. از ژنریک استفاده کنیم که یک بار کد را می‌نویسیم و با انواع مختلف کار می‌کند، در عین حال ایمنی نوع را حفظ می‌کند.

**مشکل بدون تایپ‌هینتر و ژنریک**: اگر از تایپ‌هینتر استفاده نکنیم (یا از `Any` استفاده کنیم)، ابزارهای تحلیل ایستا نمی‌توانند متوجه شوند که یک `Repository` برای کاربران، فقط باید کاربران را بپذیرد و تحویل دهد. در نتیجه، ممکن است اشتباهاً یک `Product` را به مخزن کاربران اضافه کنیم و این خطا تا زمان اجرا (و شاید در محیط تولید) شناسایی نشود. همچنین، کد ما فاقد مستندسازی صریح است و خواننده نمی‌داند که هر مخزن با چه نوعی کار می‌کند.

### TypeVar و Generic

برای تعریف یک کلاس ژنریک، به دو ابزار نیاز داریم:

- `TypeVar`: یک متغیر نوع (Type Variable) تعریف می‌کند که می‌تواند هر نوعی را بپذیرد.
- `Generic`: کلاسی که از آن ارث‌می‌بریم تا کلاس ما ژنریک شود.

```python
from typing import TypeVar, Generic

# T is a type variable that can be any type
T = TypeVar("T")

class Repository(Generic[T]):
    def __init__(self):
        self._items: list[T] = []

    def add(self, item: T) -> None:
        self._items.append(item)

    def get(self, index: int) -> T:
        return self._items[index]

    def get_all(self) -> list[T]:
        return self._items

# Simple classes for the example
class User:
    def __init__(self, name: str):
        self.name = name

class Product:
    def __init__(self, title: str, price: float):
        self.title = title
        self.price = price

# Using the repository for users
user_repo: Repository[User] = Repository()
user_repo.add(User("Ali"))
user = user_repo.get(0)  # Type checker knows that user is of type User
print(user.name)         # Ali

# Using the repository for products
product_repo: Repository[Product] = Repository()
product_repo.add(Product("Laptop", 1200.0))
product = product_repo.get(0)  # Type checker knows that product is of type Product
print(product.title)           # Laptop
```

مزیت اصلی این است که ما فقط یک کلاس `Repository` نوشته‌ایم، اما با هر نوعی کار می‌کند و تایپ‌چکر نیز ایمنی نوع را تضمین می‌کند. اگر سعی کنید یک `Product` را به `user_repo` اضافه کنید، ابزار تایپ‌چک خطا می‌دهد. همچنین، کد ما به وضوح نشان می‌دهد که هر مخزن با چه نوعی کار می‌کند.

### TypeVar با محدودیت (bound)

گاهی می‌خواهیم متغیر نوع را به یک زیرمجموعه‌ی خاص از نوع‌ها محدود کنیم. مثلاً فرض کنید تابعی می‌نویسیم که بزرگترین عنصر یک لیست را پیدا می‌کند. برای این کار، باید مطمئن باشیم که عناصر لیست قابلیت مقایسه دارند.

```python
from typing import TypeVar, Generic

class Comparable:
    def compare(self, other: "Comparable") -> int:
        """Compares two objects and returns an integer"""
        pass

# T is limited to Comparable and its subclasses
T = TypeVar("T", bound=Comparable)

def find_max(items: list[T]) -> T:
    if not items:
        raise ValueError("List is empty")
    result = items[0]
    for item in items[1:]:
        # Because of the bound, the type checker knows that item has a compare method
        if item.compare(result) > 0:
            result = item
    return result
```

بدون استفاده از `bound`، تایپ‌چکر نمی‌توانست تشخیص دهد که `item` متد `compare` را دارد و خطا می‌داد. با `bound`، به صراحت اعلام می‌کنیم که `T` باید زیرکلاس `Comparable` باشد و بنابراین، متد `compare` در دسترس است.

### ژنریک با چند پارامتر نوع

کلاس‌های ژنریک می‌توانند بیش از یک پارامتر نوع داشته باشند. برای نمونه، یک کلاس `Pair` که یک جفت کلید-مقدار را نگهداری می‌کند:

```python
from typing import TypeVar, Generic

K = TypeVar("K")  # Type of the key
V = TypeVar("V")  # Type of the value

class Pair(Generic[K, V]):
    def __init__(self, key: K, value: V):
        self.key = key
        self.value = value

    def get_key(self) -> K:
        return self.key

    def get_value(self) -> V:
        return self.value

# Using with specific types
pair1: Pair[str, int] = Pair("age", 30)
key1 = pair1.get_key()   # str
value1 = pair1.get_value()  # int

pair2: Pair[int, str] = Pair(1, "one")
key2 = pair2.get_key()   # int
value2 = pair2.get_value()  # str
```

---

## تایپ‌هینترهای پیشرفته

### Optional، Union و عملگر |

سه روش برای بیان اینکه یک مقدار می‌تواند یکی از چندین نوع باشد، وجود دارد. `Optional` برای مواقعی به کار می‌رود که مقدار می‌تواند از یک نوع خاص یا `None` باشد. `Union` برای مواقعی که مقدار می‌تواند یکی از چندین نوع باشد (که `None` هم می‌تواند یکی از آنها باشد). از پایتون ۳.۱۰ به بعد، عملگر `|` جایگزین ساده‌تری برای `Union` و `Optional` است.

```python
from typing import Optional, Union

# A function that may return a string or None
def find_user(id: int) -> Optional[str]:
    """Searches for an id and returns the user name, or None"""
    if id == 1:
        return "Ali"
    return None

# Equivalent to Optional[str] in Python 3.10+
def find_user_v2(id: int) -> str | None:
    return "Ali" if id == 1 else None

# A function that accepts an integer or a string
def parse(value: Union[int, str]) -> int:
    """Converts the input value to an integer"""
    return int(value)

# Equivalent to Union[int, str] in Python 3.10+
def parse_v2(value: int | str) -> int:
    return int(value)
```

**نکته**: `Optional[str]` به معنای «رشته یا هیچ‌چیز» است و ربطی به اختیاری بودن آرگومان‌ها ندارد. برای آرگومان‌های اختیاری، از مقدار پیش‌فرض استفاده می‌کنیم.

### Any و خطرات آن

`Any` یعنی «هر نوعی، بدون بررسی». وقتی از `Any` استفاده می‌کنید، عملاً تایپ‌چک را برای آن بخش از کد **خاموش** می‌کنید.

```python
from typing import Any

def process(data: Any) -> None:
    # Type checker gives no warnings, even if the method does not exist
    data.anything_goes_here()  # This error only occurs at runtime
```

**قاعده**: از `Any` فقط در موارد خاص استفاده کنید که واقعاً نوع مشخصی ندارید (مثلاً در توابعی که داده‌ی خام JSON را پردازش می‌کنند). در غیر این صورت، `object` یا یک Protocol خاص انتخاب بهتری است، زیرا محدودیت‌های بیشتری اعمال می‌کند.

### TypeAlias: نام‌گذاری تایپ‌های پیچیده

وقتی یک تایپ پیچیده (مانند دیکشنری تو در تو) را تکرار می‌کنید، می‌توانید با `TypeAlias` یک نام معنی‌دار به آن بدهید:

```python
from typing import TypeAlias

# An alias for a type
UserId: TypeAlias = int
UserInfo: TypeAlias = dict[str, str | int | bool]

def get_user_info(uid: UserId) -> UserInfo:
    return {"id": uid, "name": "Ali", "active": True}

# Instead of writing dict[str, str | int | bool] everywhere, we use UserInfo
```

این کار، کد را خواناتر و قابل‌نگهداری‌تر می‌کند.

### @overload: امضاهای متعدد

وقتی یک تابع بسته به نوع ورودی، خروجی متفاوتی دارد، می‌توان از دکوراتور `@overload` برای تعریف چندین امضا استفاده کرد. پیاده‌سازی اصلی، آخرین تابع خواهد بود.

```python
from typing import overload

@overload
def double(x: int) -> int: ...

@overload
def double(x: str) -> str: ...

def double(x: int | str) -> int | str:
    """Doubles the input value"""
    return x * 2

# Type checker knows that the output is int
result1: int = double(5)  # 10

# Type checker knows that the output is str
result2: str = double("ab")  # "abab"
```

توجه کنید که `@overload` فقط برای تایپ‌چکر است و در زمان اجرا، تنها تابع پیاده‌سازی‌شده (آخرین تابع) اجرا می‌شود.

### Sequence، Iterable و Collection

این تایپ‌ها، انواع انتزاعی‌تری هستند که به جای یک نوع مشخص (مانند `list`)، **قابلیت** را توصیف می‌کنند:

- `Iterable[T]`: هر چیزی که قابل پیمایش در حلقه‌ی `for` باشد (کمترین الزام).
- `Sequence[T]`: قابل ایندکس‌گذاری و دارای طول؛ مانند `list` و `tuple`.
- `Collection[T]`: قابل پیمایش، دارای طول و پشتیبانی از عملگر `in`.

```python
from typing import Iterable

def sum_all(numbers: Iterable[int]) -> int:
    """Calculates the sum of iterable numbers"""
    total = 0
    for n in numbers:
        total += n
    return total

# All of the following are accepted
print(sum_all([1, 2, 3]))       # list
print(sum_all((1, 2, 3)))       # tuple
print(sum_all({1, 2, 3}))       # set
print(sum_all(range(1, 4)))     # range
```

استفاده از `Iterable` به جای `list` یا `tuple`، تابع شما را انعطاف‌پذیرتر می‌کند و با اصل **جداسازی اینترفیس** هماهنگ است: فقط کمترین قابلیت لازم را درخواست کنید.

### TypeGuard: باریک کردن نوع

`TypeGuard` به تایپ‌چکر کمک می‌کند تا پس از یک بررسی سفارشی، نوع متغیر را «باریک‌تر» (نوع خاص‌تر) کند.

```python
from typing import TypeGuard, Any

def is_string_list(val: list[Any]) -> TypeGuard[list[str]]:
    """Checks whether all members of the list are of type str"""
    return all(isinstance(x, str) for x in val)

def process(items: list[int | str]) -> None:
    # Here, the type of items is known as list[int | str]
    if is_string_list(items):
        # After the check, the type checker knows that items is of type list[str]
        print(" ".join(items))  # This does not raise an error
    else:
        # Otherwise, the type remains list[int | str]
        print("Not a list of strings")
```

بدون `TypeGuard`، تایپ‌چکر نمی‌توانست تشخیص دهد که درون بلوک `if`، نوع `items` به `list[str]` تغییر کرده است و استفاده از متد `join` خطا می‌داد.

---

## خلاصه

```
ابزارهای انتزاع در پایتون
════════════════════════════════════════════════
  ABC       ──► اینترفیس اسمی (بر اساس ارث‌بری)
  Protocol  ──► اینترفیس ساختاری (بر اساس شکل)
  Generics  ──► کد نوع‌پارامتری با ایمنی نوع
```

| ابزار | کاربرد اصلی | نکته‌ی کلیدی |
| :--- | :--- | :--- |
| **ABC** | تعریف قرارداد با ارث‌بری | خطا در زمان ساخت شیء در صورت عدم پیاده‌سازی |
| **Protocol** | تعریف قرارداد بر اساس ساختار | بدون نیاز به ارث‌بری؛ هماهنگ با Duck Typing |
| **`@runtime_checkable`** | فعال کردن `isinstance` برای Protocol | بررسی محدود (فقط وجود متدها) |
| **`Generic` و `TypeVar`** | کلاس‌ها و توابع نوع‌پارامتری | یک بار بنویس، با هر نوعی استفاده کن |
| **`bound` در `TypeVar`** | محدود کردن نوع پارامتری | تضمین وجود متدها یا ویژگی‌های خاص |
| **`Optional` / `Union` / `\|`** | بیان چندین نوع ممکن | `\|` معادل مدرن‌تر است |
| **`Any`** | خاموش‌کردن تایپ‌چک | با احتیاط بسیار زیاد استفاده شود |
| **`Iterable` / `Sequence`** | درخواست قابلیت، نه نوع خاص | افزایش انعطاف‌پذیری توابع |
| **`TypeGuard`** | باریک‌کردن نوع پس از بررسی | کمک به تایپ‌چکر در تحلیل جریان کد |

**نکته‌ی طلایی**: اگر بین ABC و Protocol مردد هستید، Protocol را انتخاب کنید، زیرا با روح پایتون (Duck Typing) سازگارتر است و ارث‌بری اجباری را تحمیل نمی‌کند. همچنین، تایپ‌هینترها در زمان اجرا اجرا نمی‌شوند؛ ارزش واقعی آنها در استفاده از ابزارهایی مانند **mypy** است که خطاهای نوع را پیش از اجرا شناسایی می‌کنند. اما حتی بدون تایپ‌چکر، تایپ‌هینترها مستندسازی ارزشمندی هستند که خوانایی کد را افزایش می‌دهند.

---

## پرسش‌های مصاحبه

**۱. «تفاوت ABC و Protocol چیست؟ چه زمانی از هر کدام استفاده می‌کنید؟»**

- **ABC**: یک قرارداد اسمی است. برای استفاده از آن، کلاس باید به صراحت از ABC ارث‌ببرد. اگر متدی پیاده‌سازی نشود، در زمان ساخت شیء خطا رخ می‌دهد. مناسب زمانی است که کنترل سلسله‌مراتب را دارید و می‌خواهید قرارداد را تحمیل کنید.
- **Protocol**: یک قرارداد ساختاری است. کلاس‌ها نیازی به ارث‌بری ندارند؛ فقط باید متدهای مشخص‌شده را داشته باشند. برای تایپ‌چک ایستا طراحی شده و خطا را پیش از اجرا نشان می‌دهد. مناسب زمانی است که با کلاس‌های خارجی کار می‌کنید یا نمی‌خواهید ارث‌بری تحمیل کنید.
- **جمله‌ی کلیدی**: «ABC می‌گوید *کی هستی*، Protocol می‌گوید *چه می‌توانی*».

**۲. «Duck Typing چیست و Protocol چه چیزی به آن اضافه کرد؟»**

- **Duck Typing**: فلسفه‌ای که می‌گوید «اگر چیزی مانند اردک راه می‌رود و مانند اردک صدا می‌دهد، پس اردک است». یعنی به جای بررسی نوع، بررسی می‌کنیم که آیا متد مورد نظر وجود دارد یا نه.
- **Protocol**: این فلسفه را **رسمی و قابل بررسی** می‌کند. با Protocol، ابزارهای تایپ‌چک می‌توانند پیش از اجرا تشخیص دهند که یک شیء متدهای مورد نیاز را دارد یا نه. Duck Typing با کمربند ایمنی تایپ‌هینتر.

**۳. «Generic و TypeVar چه مشکلی را حل می‌کنند و کجا واقعاً به کار می‌آیند؟»**

- **مشکل**: نوشتن کد تکراری برای انواع مختلف، یا استفاده از `Any` که ایمنی نوع را از بین می‌برد و مستندسازی را ضعیف می‌کند.
- **راه‌حل**: با `Generic` و `TypeVar`، یک بار کد را می‌نویسیم و با انواع مختلف استفاده می‌کنیم، در حالی که تایپ‌چکر رابطه‌ی بین ورودی و خروجی را درک می‌کند و مستندسازی به وضوح نشان می‌دهد که هر کد با چه نوع‌هایی کار می‌کند.
- **مکان استفاده**: هر جا که منطق یکسانی برای انواع مختلف داریم؛ مانند مخزن‌ها (Repository)، کانتینرها، و توابع ابزاری که با انواع مختلف کار می‌کنند.

**۴. «`@runtime_checkable` روی Protocol چه می‌کند و محدودیت آن چیست؟»**

- کار: به `isinstance` اجازه می‌دهد روی Protocol کار کند.
- محدودیت: فقط **وجود** متدها را بررسی می‌کند، نه **امضا** یا **رفتار** آنها را. بنابراین ممکن است یک شیء Protocol را برآورده کند اما در عمل، رفتار مورد انتظار را نداشته باشد.
- توصیه: برای بررسی دقیق، به تایپ‌چک ایستا (mypy) تکیه کنید و از runtime_checkable صرفاً در موارد ضروری استفاده کنید.

---

**در فصل بعد:** حالا که با ابزارهای انتزاع و تایپ‌هینتر آشنا شدیم، سراغ ابزارهای مدرن پایتون برای تعریف کلاس‌های حامل داده می‌رویم: **dataclass، NamedTuple، Enum و TypedDict**. این ابزارها، کد تکراری را به حداقل می‌رسانند و پایه‌ی مدل‌سازی داده در پروژه‌های واقعی را تشکیل می‌دهند.