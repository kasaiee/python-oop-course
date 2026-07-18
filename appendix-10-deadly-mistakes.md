# پیوست ویژه: ۱۰ اشتباه مرگبار در کد شیءگرا — و پادزهرشان

> این پیوست جمع‌بندی چیزی است که از سال‌ها code review در ذهنم ته‌نشین شده. هر ده مورد این فهرست را بارها در کد واقعی دیده‌ام — بعضی را در کد تازه‌کارها، و راستش بعضی را در کد خودم. برای هرکدام: چطور تشخیصش بدهید، چرا «مرگبار» است (نه فقط «بد»)، و درمانش چیست. این پیوست بعد از پایان دوره بهترین استفاده را دارد، اما طوری نوشته شده که از میانه‌ی راه هم بتوانید سراغش بیایید — کنار هر اشتباه، فصل درمانش را نوشته‌ام.

یک نکته قبل از شروع: «مرگبار» را دقیق به‌کار می‌برم. کد کند، آزاردهنده است؛ کد زشت، خجالت‌آور. اما این ده مورد از جنس دیگری‌اند — **این‌ها آرام می‌کُشند.** ماه‌ها کار می‌کنند، بعد در بدترین لحظه — زیر بار، وسط تحویل، جلوی مشتری — سیستم را زمین می‌زنند یا تیم را از تغییر می‌ترسانند. و ترس از تغییر، مرگ تدریجی هر پروژه است.

---

## اشتباه ۱: God Object — کلاسی که «و» دارد

**نشانه‌ی تشخیص:** از نویسنده بخواهید کار کلاس را در یک جمله بگوید. اگر جمله‌اش «و» داشت — «سفارش را ثبت می‌کند **و** ایمیل می‌فرستد **و** موجودی را کم می‌کند» — بیمار پیدا شده. نشانه‌ی کمّی‌اش: کلاسی که در هر pull request، بی‌ربط به موضوعش، تغییر می‌خورَد.

```python
# sick: one class, four unrelated jobs
class OrderManager:
    def create_order(self, items): ...
    def send_confirmation_email(self, order): ...      # notification job
    def update_inventory(self, order): ...             # inventory job
    def generate_sales_report(self, month): ...        # reporting job
    def export_to_excel(self, report): ...             # file-format job
```

**چرا مرگبار است:** هر تغییر کوچک در گزارش‌گیری، ریسک شکستن ثبت سفارش را دارد؛ تستش یعنی بالاآوردن همه‌چیز با هم؛ و چون همه به آن وابسته‌اند، هیچ‌کس جرئت ریفکتورش را ندارد. God Object به‌مرور «منطقه‌ی ممنوعه»ی پروژه می‌شود — کدی که همه از کنارش پاورچین رد می‌شوند.

**درمان:** تفکیک بر اساس مسئولیت — هر «و»، مرز یک کلاس جدید:

```python
class OrderService:
    def __init__(self, notifier, inventory):
        self.notifier = notifier          # collaborators are injected
        self.inventory = inventory

    def create_order(self, items):
        order = Order(items)
        self.inventory.reserve(items)
        self.notifier.send_confirmation(order)
        return order

class SalesReporter:                      # reporting lives its own life
    def monthly(self, month): ...
```

📖 درمان کامل: فصل ۳ (مسئولیت) و فصل ۸ (SRP).

---

## اشتباه ۲: اشتراک ناخواسته‌ی داده‌ی تغییرپذیر

**نشانه‌ی تشخیص:** «داده‌ی یک شیء، از شیء دیگری سر درمی‌آورد.» دو چهره‌ی رایج دارد: class variable تغییرپذیر، و آرگومان پیش‌فرض تغییرپذیر.

```python
# sick — face 1: a mutable class variable
class Team:
    members = []                     # ONE list, shared by every team!
    def add(self, name):
        self.members.append(name)

# sick — face 2: a mutable default argument
class Order:
    def __init__(self, items=[]):    # ONE list, shared by every default-built order!
        self.items = items

t1, t2 = Team(), Team()
t1.add("ali")
print(t2.members)        # ['ali'] — t2 was infected

o1, o2 = Order(), Order()
o1.items.append("keyboard")
print(o2.items)          # ['keyboard'] — o2 was infected too
```

**چرا مرگبار است:** این باگ در تست کوچک دیده نمی‌شود — با یک شیء همه‌چیز درست است. در production، جایی که ده‌ها شیء ساخته می‌شوند، داده‌ها به هم نشت می‌کنند: سبد خرید یک کاربر در سبد کاربر دیگر. از آن باگ‌هایی است که بازتولیدش روزها طول می‌کشد چون به ترتیب و تعداد ساخت اشیاء وابسته است.

**درمان:** داده‌ی مختص هر شیء را در `__init__` بسازید؛ پیش‌فرض تغییرپذیر را با `None` (یا در dataclass با `default_factory`) جایگزین کنید:

```python
class Team:
    def __init__(self):
        self.members = []            # a fresh list per team

class Order:
    def __init__(self, items=None):
        self.items = items if items is not None else []
```

📖 درمان کامل: فصل ۲ (class/instance variable) و فصل ۱۰ (`field(default_factory=...)`).

---

## اشتباه ۳: ارث‌بری با رابطه‌ی Is-Aی دروغین

**نشانه‌ی تشخیص:** ارث‌بری فقط برای «کش رفتن چند متد»، بی‌آنکه جمله‌ی «هر X یک Y است» صادق باشد. علامت قطعی: زیرکلاسی که بعضی متدهای والد را با `raise` یا بدنه‌ی خالی «خنثی» می‌کند.

```python
# sick: a Stack is NOT a list — but we inherited to steal append/pop
class Stack(list):
    def push(self, item):
        self.append(item)

s = Stack()
s.push(1)
s.insert(0, 999)     # oops — the whole list API leaked into our Stack!
print(s)             # [999, 1] — someone just broke LIFO from the outside
```

**چرا مرگبار است:** هر متد والد، بخشی از قرارداد عمومی شماست — چه بخواهید چه نه. کاربران کلاس به آن متدهای قاچاقی تکیه می‌کنند و از جایی به بعد نه می‌توانید ارث‌بری را بردارید نه رفتار را درست کنید. و اگر زیرکلاس قرارداد را «سفت‌تر» کند، هر کدی که با نوع والد کار می‌کرد، با زیرکلاس می‌شکند — نقض LSP، یعنی خیانت به چندریختی.

**درمان:** رابطه‌ی واقعی Has-A است؟ پس Composition:

```python
class Stack:
    def __init__(self):
        self._items = []             # HAS a list; IS not a list

    def push(self, item):
        self._items.append(item)

    def pop(self):
        return self._items.pop()
```

📖 درمان کامل: فصل ۷ (Composition بر ارث‌بری) و فصل ۸ (LSP).

---

## اشتباه ۴: زنجیره‌ی if/isinstance به‌جای چندریختی

**نشانه‌ی تشخیص:** یک `if/elif` روی «نوع» که در چند جای برنامه کپی شده و با هر نوع جدید، همه‌جا جراحی می‌خواهد.

```python
# sick: the type-switch that grows forever, everywhere
def calculate_fee(shipment):
    if isinstance(shipment, ExpressShipment):
        return shipment.weight * 20_000
    elif isinstance(shipment, StandardShipment):
        return shipment.weight * 10_000
    elif isinstance(shipment, EconomyShipment):
        return shipment.weight * 5_000
    # next month: elif DroneShipment... in this function AND four others
```

**چرا مرگبار است:** منطق هر نوع، به‌جای اینکه یک‌جا در کلاس خودش بنشیند، در سراسر برنامه پاشیده شده. افزودن نوع جدید یعنی «همه‌ی جاهایی که سوییچ کرده‌ایم را پیدا کن» — و همیشه یکی از قلم می‌افتد؛ همان یکی که در production پیدا می‌شود. این دقیقاً وارونه‌ی وعده‌ی شیءگرایی است.

**درمان:** رفتار را به خود نوع واگذار کنید (چندریختی) تا نوع جدید = فقط کلاس جدید:

```python
class ExpressShipment:
    rate = 20_000
    def fee(self):
        return self.weight * self.rate

class StandardShipment:
    rate = 10_000
    def fee(self):
        return self.weight * self.rate

def calculate_fee(shipment):
    return shipment.fee()            # no ifs. it will never grow again.
```

📖 درمان کامل: فصل ۴ (چندریختی) و فصل ۸ (Open/Closed).

---

## اشتباه ۵: دست‌درازی به اندرونی دیگران

**نشانه‌ی تشخیص:** زیرخط‌هایی که نادیده گرفته می‌شوند (`obj._internal`) و زنجیره‌های نقطه‌ی طولانی (`order.customer.wallet.balance -= x`).

```python
# sick: outside code reaches deep into other objects' guts
def apply_refund(order, amount):
    order.customer.wallet._balance += amount          # touching a _private!
    order.customer.wallet._history.append(("refund", amount))
    if order.customer.wallet._balance > 10_000_000:
        order.customer.tier = "vip"                   # side-managing someone else's state
```

**چرا مرگبار است:** منطق کیف پول حالا در دو جا زندگی می‌کند: داخل `Wallet` و داخل هر کدی که به اندرونی‌اش دست زده. روزی که قانون موجودی عوض شود (مثلاً سقف، یا لاگ اجباری)، باید همه‌ی این دست‌درازی‌ها را پیدا کنید. بدتر: هیچ اعتبارسنجی‌ای اجرا نمی‌شود چون از در پشتی وارد شده‌اید. کپسوله‌سازی دقیقاً برای بستن همین در بود.

**درمان:** Tell, Don't Ask — از شیء بخواهید کار را انجام دهد، نه اینکه اندرونی‌اش را بیرون بکشید:

```python
def apply_refund(order, amount):
    order.customer.refund(amount)     # one call to the direct neighbor

class Customer:
    def refund(self, amount):
        self.wallet.deposit(amount)   # Wallet enforces its own rules
        self._update_tier()
```

📖 درمان کامل: فصل ۴ (کپسوله‌سازی) و فصل ۷ (قانون دیمتر، Tell Don't Ask).

---

## اشتباه ۶: __eq__ بدون __hash__ (و hash روی شیء تغییرپذیر)

**نشانه‌ی تشخیص:** کلاسی که `__eq__` گرفته و بعد جایی در برنامه، داخل `set` یا کلید `dict` استفاده شده — یا برعکس: hashable است اما فیلدهای مبنای برابری‌اش تغییر می‌کنند.

```python
class Product:
    def __init__(self, sku):
        self.sku = sku
    def __eq__(self, other):
        return isinstance(other, Product) and self.sku == other.sku

# somewhere far away, months later:
seen = set()
# seen.add(Product("A100"))     # TypeError: unhashable type: 'Product'
```

**چرا مرگبار است:** خود `TypeError` بالا تازه حالت *خوش‌شانسی* است — بلند می‌ترکد. حالت بدشانسی وقتی است که `__hash__` دستی می‌دهید اما شیء تغییرپذیر است: شیء را در `set` می‌گذارید، بعد فیلدش عوض می‌شود، و از آن لحظه شیء در مجموعه «گم» می‌شود — `in` می‌گوید نیست، در حالی که هست. داده‌ی خاموش خراب، بدترین نوع باگ.

**درمان:** قرارداد را کامل رعایت کنید — `__eq__` و `__hash__` با هم و بر مبنای فیلدهای **تغییرناپذیر**؛ یا کار را به ابزار مطمئن بسپارید:

```python
from dataclasses import dataclass

@dataclass(frozen=True)          # eq + hash + immutability, all consistent
class Product:
    sku: str
```

📖 درمان کامل: فصل ۵ (`__eq__`/`__hash__`) و فصل ۱۰ (`frozen=True`).

---

## اشتباه ۷: بلعیدن استثناها

**نشانه‌ی تشخیص:** `except Exception: pass` — یا برادر محترم‌ترش که فقط `print` می‌کند و رد می‌شود.

```python
# sick: the error-swallower
class PaymentProcessor:
    def process(self, order):
        try:
            self.gateway.charge(order.card, order.total)
            self.inventory.reserve(order.items)
            self.notifier.send(order.user, "paid!")
        except Exception:
            pass          # "so it never crashes" — the most expensive line in the codebase
```

**چرا مرگبار است:** این کد هرگز crash نمی‌کند — و این دقیقاً فاجعه است. پرداخت شکست خورده اما سیستم «موفق» گزارش می‌دهد؛ موجودی رزرو نشده اما سفارش پیش می‌رود. خطا نمرده؛ فقط بی‌صدا شده و چند لایه دورتر، به شکل داده‌ی متناقض، دوباره ظاهر می‌شود — جایی که دیگر هیچ ردی به علت اصلی نیست. تیم‌هایی دیده‌ام که هفته‌ها دنبال «باگ گم‌شدن سفارش‌ها» گشتند و آخرش به یک `pass` رسیدند.

**درمان:** فقط استثنایی را بگیرید که **می‌دانید با آن چه کنید**؛ بقیه را بگذارید بالا بروند. و اگر ترجمه می‌کنید، زنجیره را نگه دارید:

```python
class PaymentError(Exception): ...

class PaymentProcessor:
    def process(self, order):
        try:
            self.gateway.charge(order.card, order.total)
        except ConnectionError as e:                       # specific, and handled
            raise PaymentError(f"gateway unreachable for order {order.id}") from e
        self.inventory.reserve(order.items)                # let other errors surface
        self.notifier.send(order.user, "paid!")
```

📖 درمان کامل: فصل ۱۱ (سلسله‌مراتب استثنا، `raise ... from ...`).

---

## اشتباه ۸: حالت سراسری پنهان

**نشانه‌ی تشخیص:** Singletonها، متغیرهای سطح ماژول که همه‌جا import و دست‌کاری می‌شوند، و کلاسی که وابستگی‌هایش را از «فضای سراسری» برمی‌دارد نه از سازنده‌اش. علامت آزمایشگاهی: تستی که تنها سبز است و کنار بقیه قرمز.

```python
# sick: module-level mutable state that everyone touches
class Database:                       # (stub, just so the example is self-contained)
    def __init__(self, dsn): ...
    def query(self, sql): ...

current_user = None
db_connection = Database("prod-server")

class ReportService:
    def generate(self):
        return db_connection.query(f"... where user = {current_user.id}")
        # who set current_user? when? from which thread? nobody knows.
```

**چرا مرگبار است:** وابستگی‌ها نامرئی شده‌اند — امضای هیچ متدی لو نمی‌دهد که به `current_user` نیاز دارد. ترتیب اجرا مهم می‌شود، تست‌ها به هم نشت می‌کنند، و اجرای موازی (نخ‌ها!) به قمار تبدیل می‌شود. کدی که با حالت سراسری کار می‌کند، قابل «استدلال محلی» نیست: برای فهمیدن یک تابع باید کل برنامه را در ذهن نگه دارید.

**درمان:** وابستگی را صریح و تزریقی کنید — همان چیزی که کل نیمه‌ی دوم دوره ساخت:

```python
class ReportService:
    def __init__(self, db, user):        # visible, explicit, testable
        self.db = db
        self.user = user

    def generate(self):
        return self.db.query("... where user = ?", self.user.id)
```

📖 درمان کامل: فصل ۸ (DIP)، فصل ۱۲ (نقد Singleton)، فصل ۱۳ (اثرش بر تست).

---

## اشتباه ۹: سازنده‌ی پرکار

**نشانه‌ی تشخیص:** `__init__`ای که به شبکه یا دیسک می‌زند، فایل می‌خواند، اتصال باز می‌کند یا محاسبه‌ی سنگین انجام می‌دهد. تست ساده: «آیا می‌توانم این شیء را در تست، بدون اینترنت و در یک میلی‌ثانیه بسازم؟»

```python
# sick: building the object = talking to the whole world
class CurrencyConverter:
    def __init__(self):
        response = urlopen("https://api.rates.example/latest")   # network in __init__!
        self.rates = json.load(response)
```

**چرا مرگبار است:** ساختن شیء باید ارزان و بی‌خطر باشد — این قرارداد نانوشته‌ی همه‌ی کدهایی است که با کلاس شما کار می‌کنند. سازنده‌ی پرکار یعنی: هر تستی که این کلاس را لمس کند به اینترنت وابسته می‌شود، هر import بی‌احتیاط ممکن است یک درخواست شبکه بزند، و خطای شبکه در عجیب‌ترین جای برنامه (وسط ساخت یک شیء!) منفجر می‌شود.

**درمان:** سازنده فقط **بگیرد و نگه دارد**؛ کار گران را به factory صریح یا بارگذاری تنبل بسپارید:

```python
class CurrencyConverter:
    def __init__(self, rates):               # cheap, honest, testable
        self.rates = rates

    @classmethod
    def from_api(cls, url):                  # the expensive path, explicitly named
        response = urlopen(url)
        return cls(json.load(response))

# in production:  converter = CurrencyConverter.from_api(RATES_URL)
# in tests:       converter = CurrencyConverter({"USD": 1.0, "EUR": 0.9})
```

📖 درمان کامل: فصل ۵ (factory با classmethod) و فصل ۱۳ (طراحی تست‌پذیر).

---

## اشتباه ۱۰: پیچیدگی خودخواسته

**نشانه‌ی تشخیص:** انتزاعی که فقط یک پیاده‌سازی دارد و «شاید روزی» دومی بیاید؛ متاکلاس جایی که یک دکوراتور کافی بود؛ `__slots__` روی کلاسی با ده نمونه؛ سلسله‌مراتب پنج‌طبقه برای مسئله‌ای دوکلاسه.

```python
from abc import ABC, abstractmethod

# sick: an AbstractFactory... for creating exactly one kind of report
class AbstractReportFactoryProvider(ABC):
    @abstractmethod
    def get_factory(self) -> "AbstractReportFactory": ...

class DefaultReportFactoryProvider(AbstractReportFactoryProvider):
    def get_factory(self):
        return SimpleReportFactory()          # the one and only, forever
```

**چرا مرگبار است:** این تنها اشتباه فهرست است که قربانی‌اش مستقیماً داده یا uptime نیست — قربانی‌اش **تیم** است. هر لایه‌ی انتزاع بی‌دلیل، هزینه‌ی فهمیدن را برای همیشه بالا می‌برد؛ کد ساده‌ی زیرش قابل‌دفاع بود، اما حالا خواندن جریان برنامه یعنی شش پرش بین فایل‌ها. و خطرناک‌تر از همه: چون «حرفه‌ای» به نظر می‌رسد، کسی جرئت نقدش را ندارد. کد بیش‌مهندسی‌شده از کد کم‌مهندسی‌شده دیرتر اصلاح می‌شود — نویسنده‌اش به آن افتخار می‌کند.

**درمان:** قاعده‌ی سه‌مرحله‌ای که در کل دوره تکرار شد: اول **درد**، بعد **میان‌بُر زبان**، و فقط بعدش **ابزار سنگین**. انتزاع را وقتی بسازید که پیاده‌سازی دوم واقعاً وجود دارد، نه در خیال:

```python
def build_report(records):        # yes. sometimes THIS is the professional answer.
    ...
```

📖 درمان کامل: فصل ۱۲ (تب الگو)، فصل ۱۶ (سلسله‌ی جایگزین‌های متاکلاس)، فصل ۱۷ (بهینه‌سازی زودرس).

---

## چک‌لیست نهایی برای Code Review

این جدول را موقع بازبینی کد — مال خودتان یا دیگران — جلوی چشم بگذارید:

| # | بپرس | اگر جواب «بله» بود |
|---|------|---------------------|
| ۱ | توصیف کلاس «و» دارد؟ | بشکن — اشتباه ۱ |
| ۲ | class variable یا پیش‌فرض تغییرپذیر داری؟ | به `__init__`/`default_factory` ببر — اشتباه ۲ |
| ۳ | زیرکلاس متدی از والد را خنثی می‌کند؟ | Composition — اشتباه ۳ |
| ۴ | روی نوع سوییچ می‌کنی و این سوییچ چند جا کپی شده؟ | چندریختی — اشتباه ۴ |
| ۵ | به `_چیز` دیگران یا زنجیره‌ی نقطه‌ی بلند دست می‌زنی؟ | Tell, Don't Ask — اشتباه ۵ |
| ۶ | `__eq__` بدون `__hash__` سازگار؟ شیء hashable تغییر می‌کند؟ | `frozen=True` یا جفت کامل — اشتباه ۶ |
| ۷ | جایی `except Exception: pass` هست؟ | مشخص بگیر، ترجمه کن، بگذار بالا برود — اشتباه ۷ |
| ۸ | وابستگی از فضای سراسری می‌آید نه از سازنده؟ | تزریق — اشتباه ۸ |
| ۹ | `__init__` به شبکه/دیسک می‌زند؟ | factory صریح — اشتباه ۹ |
| ۱۰ | انتزاعی هست که فقط یک پیاده‌سازی دارد؟ | حذفش کن تا دردش واقعی شود — اشتباه ۱۰ |

**و یک جمله‌ی آخر:** هیچ‌کدام از این ده اشتباه از بی‌سوادی نمی‌آید؛ از عجله، از «فعلاً کار می‌کند»، و از تقلید بی‌سوال می‌آید. تفاوت مهندس خوب این نیست که هرگز مرتکبشان نمی‌شود — این است که در code review، هم در کد دیگران و هم در کد دیروز خودش، می‌بیندشان و بی‌تعارف اصلاحشان می‌کند.
