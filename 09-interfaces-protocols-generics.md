# فصل نهم: اینترفیس‌ها، پروتکل‌ها و ژنریک‌ها

---

## مقدمه

فصل قبل، SOLID مدام از «انتزاع» و «اینترفیس» حرف زد، اما ابزارِ رسمیِ ساختنشان را نشان نداد. این فصل همان خلأ را پر می‌کند.

پایتون سه ابزارِ اصلی برای انتزاع دارد: **Abstract Base Classes (ABC)** برای تعریفِ اینترفیسِ اسمی، **Protocol** برای اینترفیسِ ساختاری (Duck Typing در سطحِ تایپ)، و **Generics** برای نوشتنِ کدی که با انواعِ مختلف کار می‌کند بدونِ تکرار. در پایان، نگاهی به تایپ‌هینترهای پیشرفته می‌اندازیم که کد شما را امن‌تر و خواناتر می‌کنند.

---

## چرا این ابزارها را یاد بگیریم؟

بسیاری از برنامه‌نویسانِ پایتون هرگز از ABC یا Protocol استفاده نمی‌کنند — و کدشان کار می‌کند. اما لحظه‌ای که کدِ کتابخانه‌های بزرگ را باز کنید، همه‌جا این‌ها را می‌بینید. یادگیری‌شان به شما کمک می‌کند:

- کدی بنویسید که به‌راحتی قابل تست و جایگزینی باشد (همان DIP فصل قبل)
- از تایپ‌هینترها برای گرفتنِ باگ‌ها **پیش از اجرا** استفاده کنید
- کتابخانه‌های حرفه‌ای و قابل‌توسعه بسازید
- معماری‌های تمیز را پیاده کنید

---

## Abstract Base Classes (ABC)

### ABC چیست و چرا لازم است؟

یک **کلاس انتزاعی** (Abstract Base Class) کلاسی است که قرار **نیست** مستقیماً نمونه‌سازی شود؛ فقط یک **قرارداد** تعریف می‌کند که زیرکلاس‌ها باید رعایتش کنند. متدهای انتزاعی‌اش بدنه ندارند و زیرکلاس موظف است پیاده‌شان کند.

بدون ABC، اگر زیرکلاس یک متد را پیاده نکند، خطا فقط **هنگام فراخوانی** آن متد رخ می‌دهد — شاید خیلی دیر. با ABC، خطا **هنگام ساختِ شیء** رخ می‌دهد — خیلی زودتر و امن‌تر.

```python
from abc import ABC, abstractmethod

class PaymentGateway(ABC):
    @abstractmethod
    def pay(self, amount): ...

    @abstractmethod
    def refund(self, amount): ...

class StripeGateway(PaymentGateway):
    def pay(self, amount):
        return f"Stripe: {amount}"
    def refund(self, amount):
        return f"Stripe refund: {amount}"

s = StripeGateway()      # works
print(s.pay(100))        # Stripe: 100

# if a subclass does not implement a method:
class BrokenGateway(PaymentGateway):
    def pay(self, amount):
        return "..."
    # refund not implemented!

# BrokenGateway()   # TypeError at construction — not when calling refund
```

نکته‌ی کلیدی: خودِ `PaymentGateway` را نمی‌توان نمونه‌سازی کرد (`PaymentGateway()` خطا می‌دهد)، و هر زیرکلاسی که همه‌ی متدهای انتزاعی را پیاده نکند، همان لحظه‌ی ساختِ شیء خطا می‌دهد. این «شکستِ زودهنگام» ارزشمند است.

### چه زمانی از ABC استفاده کنیم و چه زمانی نه؟

**مناسب است وقتی:** می‌خواهید یک خانواده از کلاس‌ها را وادار به رعایتِ یک قرارداد کنید، و رابطه‌ی ارث‌بریِ اسمی («این یک نوعِ Gateway است») معنا دارد.

**مناسب نیست وقتی:** فقط می‌خواهید بگویید «هر چیزی که این متدها را دارد قبول است» بدونِ الزام به ارث‌بری — که کارِ Protocol است.

---

## Protocolها

### Protocol چیست؟ (Structural Subtyping)

**Protocol** (از پایتون ۳.۸، در ماژول `typing`) اجازه می‌دهد اینترفیس را بر اساسِ **ساختار** تعریف کنید نه ارث‌بری. یعنی: «هر شیئی که این متدها را داشته باشد، این Protocol را برآورده می‌کند» — بدونِ اینکه لازم باشد از چیزی ارث ببرد. این، همان Duck Typing است اما در سطحِ تایپ‌هینتر و قابل‌بررسی توسط ابزارها.

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> str: ...

# these classes don't inherit from Drawable, but they have its structure
class Button:
    def draw(self) -> str:
        return "[button]"

class Icon:
    def draw(self) -> str:
        return "(icon)"

def render(item: Drawable) -> str:      # anything that has draw is accepted
    return item.draw()

print(render(Button()))   # [button]
print(render(Icon()))     # (icon)
```

`Button` و `Icon` هیچ ارث‌بریِ مشترکی ندارند، اما هر دو `Drawable` را برآورده می‌کنند چون متدِ `draw` را دارند. ابزارهای تایپ‌چک (مثل mypy) این را می‌فهمند و اگر شیئی بدونِ `draw` پاس دهید، **پیش از اجرا** هشدار می‌دهند.

### ABC در برابر Protocol

این مهم‌ترین تمایزِ فصل است:

| | ABC | Protocol |
|---|-----|----------|
| نوعِ رابطه | اسمی (Nominal) — باید ارث ببری | ساختاری (Structural) — فقط شکل مهم است |
| الزام ارث‌بری | بله (`class X(MyABC)`) | خیر |
| بررسی هنگام ساخت | بله (متدِ نپیاده = خطا) | خیر (پیش‌فرض فقط تایپ‌چکِ ایستا) |
| مناسبِ | خانواده‌ی کلاس‌های تحتِ کنترلِ شما | تطبیقِ کلاس‌هایی که کنترلشان ندارید |

مثالِ روشن‌کننده: فرض کنید می‌خواهید تابعی بنویسید که با هر چیزی که `read()` دارد کار کند — چه `open()` استانداردِ پایتون، چه یک شیءِ سفارشی، چه کلاسِ یک کتابخانه‌ی دیگر. با ABC باید همه‌شان از یک پایه ارث می‌بردند (غیرممکن، چون کدِ آن‌ها را کنترل نمی‌کنید). با Protocol، فقط داشتنِ `read()` کافی است.

### runtime_checkable

پیش‌فرض، Protocolها فقط برای تایپ‌چکِ ایستا هستند و در زمان اجرا `isinstance` رویشان کار نمی‌کند. با `@runtime_checkable` این را فعال می‌کنید:

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Closeable(Protocol):
    def close(self) -> None: ...

class Connection:
    def close(self) -> None:
        print("closed")

conn = Connection()
print(isinstance(conn, Closeable))   # True — without inheritance!
```

**Tradeoff:** `isinstance` روی Protocolِ runtime فقط **وجودِ** متدها را چک می‌کند، نه امضا یا رفتارشان را. پس صددرصد امن نیست. برای بررسیِ دقیق، به تایپ‌چکِ ایستا (mypy) تکیه کنید.

### چه زمانی Protocol و چه زمانی ABC؟

- **ABC**: وقتی خانواده‌ای از کلاس‌ها را خودتان می‌سازید و می‌خواهید قرارداد را با ارث‌بری تحمیل و هنگام ساخت بررسی کنید.
- **Protocol**: وقتی می‌خواهید با کلاس‌هایی کار کنید که کنترلشان ندارید، یا نمی‌خواهید ارث‌بری تحمیل کنید. Protocol با فلسفه‌ی Duck Typingِ پایتون سازگارتر است.

---

## ژنریک‌ها (Generics)

### مشکل: کد تکراری برای انواع مختلف

فرض کنید یک «مخزن» (Repository) می‌خواهید که هر نوع موجودیتی را نگه دارد. بدونِ ژنریک، یا باید برای هر نوع یک کلاسِ جدا بنویسید، یا از تایپ‌هینتِ مبهمِ `Any` استفاده کنید که ایمنیِ نوع را از بین می‌برد.

### TypeVar و Generic

**Generic** به شما اجازه می‌دهد یک کلاس را با یک «نوعِ پارامتری» بنویسید که هنگام استفاده مشخص می‌شود:

```python
from typing import TypeVar, Generic

T = TypeVar("T")                     # a parametric type

class Repository(Generic[T]):
    def __init__(self):
        self._items: list[T] = []

    def add(self, item: T) -> None:
        self._items.append(item)

    def get(self, index: int) -> T:
        return self._items[index]

class User: ...
class Product: ...

user_repo: Repository[User] = Repository()     # user repository
product_repo: Repository[Product] = Repository()  # product repository

user_repo.add(User())
u = user_repo.get(0)    # the type checker knows u is a User, not Any
```

مزیت: یک کلاسِ `Repository` نوشتیم که با هر نوعی کار می‌کند، اما ایمنیِ نوع حفظ می‌شود. `user_repo.get(0)` را تایپ‌چکر به‌عنوان `User` می‌شناسد و اگر اشتباهی محصول اضافه کنید، هشدار می‌دهد.

### TypeVar با محدودیت (bound)

گاهی می‌خواهید نوعِ پارامتری هر چیزی نباشد، بلکه زیرمجموعه‌ی خاصی را محدود کنید:

```python
from typing import TypeVar

class Comparable:
    def compare(self, other) -> int: ...

# T can only be Comparable or its subclasses
T = TypeVar("T", bound=Comparable)

def find_max(items: list[T]) -> T:
    result = items[0]
    for item in items[1:]:
        if item.compare(result) > 0:    # because we have a bound, compare is allowed
            result = item
    return result
```

بدونِ `bound=Comparable`، تایپ‌چکر نمی‌دانست `item.compare` وجود دارد. با bound، تضمین می‌کنیم `T` حتماً `compare` دارد.

### Generic با چند پارامتر

می‌توان چند نوعِ پارامتری داشت:

```python
from typing import TypeVar, Generic

K = TypeVar("K")
V = TypeVar("V")

class Pair(Generic[K, V]):
    def __init__(self, key: K, value: V):
        self.key = key
        self.value = value

p: Pair[str, int] = Pair("age", 30)
# the type checker knows p.key is a str and p.value is an int
```

---

## تایپ‌هینترهای پیشرفته

### Optional، Union و |

سه راه برای گفتنِ «این می‌تواند چند نوع باشد»:

```python
from typing import Optional, Union

def find_user(id: int) -> Optional[str]:      # str or None
    return "ali" if id == 1 else None

def parse(x: Union[int, str]) -> int:         # int or str
    return int(x)

# since Python 3.10, the | syntax is the more modern alternative:
def find_user_v2(id: int) -> str | None:      # equivalent to Optional[str]
    return "ali" if id == 1 else None

def parse_v2(x: int | str) -> int:            # equivalent to Union[int, str]
    return int(x)
```

نکته: `Optional[str]` دقیقاً یعنی `str | None` — نه «اختیاری بودنِ آرگومان»، بلکه «می‌تواند None باشد».

### Any و خطرش

`Any` یعنی «هر نوعی، بی‌بررسی». استفاده‌اش تایپ‌چک را **خاموش** می‌کند:

```python
from typing import Any

def process(data: Any):      # the type checker no longer checks anything
    return data.anything()   # gives no warning, even if it's wrong
```

`Any` را فقط وقتی به کار ببرید که واقعاً نوع نامشخص است (مثلاً داده‌ی خامِ JSON). استفاده‌ی بی‌رویه‌اش، کلِ فایده‌ی تایپ‌هینت را از بین می‌برد. اغلب `object` یا یک Protocol، انتخابِ امن‌تری است.

### TypeAlias

برای نام‌گذاریِ تایپ‌های پیچیده و خواناتر کردن:

```python
from typing import TypeAlias

UserId: TypeAlias = int
ConnectionOptions: TypeAlias = dict[str, str | int]

def get_user(uid: UserId) -> str: ...     # more readable than a bare int
```

### @overload

وقتی یک تابع بسته به نوعِ ورودی، خروجیِ متفاوتی دارد:

```python
from typing import overload

@overload
def double(x: int) -> int: ...
@overload
def double(x: str) -> str: ...

def double(x):                    # the real implementation
    return x * 2

r1 = double(5)      # the type checker knows it's int → 10
r2 = double("ab")   # the type checker knows it's str → "abab"
```

`@overload` فقط برای تایپ‌چکر است؛ در زمان اجرا فقط پیاده‌سازیِ آخر اجرا می‌شود.

### Sequence، Iterable و Collection

این‌ها تایپ‌های انتزاعی‌اند که به‌جای نوعِ مشخص، **قابلیت** را توصیف می‌کنند:

- `Iterable[T]`: هر چیزی که بتوان رویش `for` زد (کمترین الزام).
- `Sequence[T]`: قابلِ ایندکس و دارای طول (مثل list و tuple).
- `Collection[T]`: قابلِ پیمایش، دارای طول، و پشتیبان `in`.

```python
from typing import Iterable

def total(numbers: Iterable[int]) -> int:    # list, tuple, set, generator... all accepted
    return sum(numbers)

print(total([1, 2, 3]))       # list
print(total((1, 2, 3)))       # tuple
print(total({1, 2, 3}))       # set
```

استفاده از این انتزاع‌ها به‌جای نوعِ مشخص (`list`)، تابع را منعطف‌تر می‌کند — همان روحِ Interface Segregation: کمترین قابلیتِ لازم را بخواه.

### TypeGuard

به تایپ‌چکر کمک می‌کند بعد از یک بررسی، نوع را «باریک» کند:

```python
from typing import TypeGuard

def is_str_list(val: list[object]) -> TypeGuard[list[str]]:
    return all(isinstance(x, str) for x in val)

def process(items: list[object]):
    if is_str_list(items):
        # here the type checker knows items is a list[str]
        print(" ".join(items))
```

---

## خلاصه

```
ابزارهای انتزاع در پایتون
════════════════════════════════════════════════
  ABC       ──► اینترفیسِ اسمی: «باید ارث ببری و پیاده کنی»
  Protocol  ──► اینترفیسِ ساختاری: «فقط شکلت مهم است»
  Generics  ──► «با هر نوعی کار می‌کنم، بی‌تکرار، بی‌ازدست‌دادنِ ایمنی»
```

| ابزار | نکته‌ی کلیدی |
|-------|--------------|
| ABC | قرارداد با ارث‌بری؛ خطا هنگام ساخت |
| Protocol | Duck Typing تایپ‌شده؛ بدون ارث‌بری |
| `@runtime_checkable` | اجازه‌ی `isinstance` روی Protocol (محدود) |
| `Generic` / `TypeVar` | کلاسِ نوع‌پارامتری، ایمنِ نوع |
| `bound=` | محدودکردنِ نوعِ پارامتری |
| `Optional` / `Union` / `\|` | چند نوعِ ممکن |
| `Any` | خاموش‌کردنِ تایپ‌چک (با احتیاط) |
| `Iterable`/`Sequence` | خواستنِ قابلیت، نه نوعِ مشخص |

**نکته‌ی طلایی:** بین ABC و Protocol، اگر شک دارید، Protocol را انتخاب کنید — چون با روحِ Duck Typingِ پایتون سازگارتر است و ارث‌بریِ اجباری تحمیل نمی‌کند. و به‌یاد داشته باشید تایپ‌هینترها در زمانِ اجرا اجرا نمی‌شوند؛ ارزششان در ابزارهایی مثل mypy است که باگ را پیش از اجرا می‌گیرند. تایپ‌هینت بدونِ تایپ‌چکر، فقط مستندسازی است — که آن هم بی‌ارزش نیست.

---

## پرسش‌های مصاحبه

**۱. «ABC یا Protocol؟ معیارِ انتخابت چیست؟»** — ABC: قراردادِ اسمی و صریح؛ زیرکلاس باید ارث ببرد و نبودِ متد در لحظه‌ی ساخت می‌ترکد؛ مناسبِ وقتی مالکِ سلسله‌مراتب خودتانید یا پیاده‌سازیِ مشترک می‌دهید. Protocol: قراردادِ ساختاری؛ هر کلاسی با «شکلِ» درست می‌گنجد حتی کلاسِ کتابخانه‌ی غریبه؛ مناسبِ مرزها و افزونه‌ها. جمله‌ی طلایی: «ABC می‌گوید *کی هستی*، Protocol می‌گوید *چه می‌توانی*.»

**۲. «Duck Typing چیست و Protocol چه چیزی به آن اضافه کرد؟»** — «اگر مثل اردک راه می‌رود و صدا می‌دهد، اردک است»: به‌جای نوع، به رفتار اعتماد کن. Protocol همین را **رسمی و قابلِ‌بررسی** کرد: type checker حالا قبل از اجرا می‌گوید این شیء شکلِ لازم را ندارد — Duck Typing با کمربندِ ایمنی.

**۳. «Generic و TypeVar کجا واقعاً می‌ارزند؟»** — جایی که یک منطق برای انواعِ مختلف تکرار می‌شود و می‌خواهید **رابطه‌ی** ورودی/خروجی حفظ شود: `Repository[User]` که `get`اش `User` برگرداند نه `Any`. بدونِ Generic یا کدِ تکراری دارید یا `Any` — یعنی خاموش‌کردنِ type checker دقیقاً جایی که بیشترین ارزش را دارد.

---

**در فصل بعد:** حالا که تایپ‌هینت و انتزاع را می‌شناسیم، سراغِ ابزارهای مدرنِ پایتون برای ساختنِ کلاس‌های حاملِ داده می‌رویم — **dataclass، NamedTuple، Enum و TypedDict** — که کدِ تکراری را به حداقل می‌رسانند و پایه‌ی مدل‌سازیِ داده در پروژه‌های واقعی‌اند.
