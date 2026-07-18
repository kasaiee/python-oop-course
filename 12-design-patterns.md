# فصل دوازدهم: الگوهای طراحی — به سبک پایتونی

---

## مقدمه

تا اینجا ابزارها را جمع کرده‌ایم: کلاس و ارث‌بری و Composition، اصول SOLID، انتزاع با ABC و Protocol، مدل‌های داده و استثناها. حالا وقت سوال بعدی است: وقتی با یک مسئله‌ی آشنا روبه‌رو می‌شوید — «رفتار باید عوض‌شدنی باشد»، «چند بخش سیستم باید از یک اتفاق باخبر شوند»، «ساخت شیء پیچیده شده» — از کجا شروع می‌کنید؟

خبر خوب این است که شما اولین نفری نیستید که به این مسئله‌ها برخورده. مهندسان نرم‌افزار ده‌ها سال است همین مسئله‌ها را حل می‌کنند و راه‌حل‌های تکرارشونده‌شان اسم گرفته‌اند: **الگوهای طراحی (Design Patterns)**. این فصل مهم‌ترین‌هایشان را یاد می‌دهد — اما با یک تبصره‌ی بزرگ که بیشتر منابع نمی‌گویند: **الگوها را به زبان پایتون یاد می‌گیریم، نه به لهجه‌ی جاوا.**

---

## چرا الگوهای طراحی؟ و چرا «به سبک پایتونی»؟

سال ۱۹۹۴ چهار نفر — که بعدها به **Gang of Four (GoF)** معروف شدند — کتابی نوشتند و ۲۳ راه‌حل تکرارشونده‌ی طراحی را فهرست کردند. آن کتاب واژگان مشترک حرفه‌ی ما را ساخت: وقتی همکارتان می‌گوید «اینجا Strategy بزن»، یک پاراگراف توضیح در دو کلمه منتقل شده.

سه دلیل برای یادگیری‌شان:

- **واژگان مشترک:** در جلسه‌ی طراحی و code review، با دو کلمه حرف هم را می‌فهمید. در مصاحبه‌های استخدامی هم پای ثابت‌اند.
- **راه‌حل آزموده:** الگو یعنی «این مسیر را هزاران تیم رفته‌اند و چاله‌هایش را می‌شناسیم».
- **درک فریمورک‌ها:** Django و SQLAlchemy و pytest پُر از این الگوهایند. کسی که الگو بلد است، فریمورک را «می‌فهمد» نه اینکه فقط «استفاده کند».

اما تبصره‌ی مهم: کتاب GoF برای C++ و Smalltalk نوشته شد — زبان‌هایی که نه تابع درجه‌یک داشتند و نه Duck Typing. بخشی از آن ۲۳ الگو در پایتون **آب می‌روند یا کلاً محو می‌شوند**، چون خود زبان مشکل را حل کرده. یک مثال معروف: الگوی Strategy در جاوا به یک اینترفیس و چند کلاس نیاز دارد؛ در پایتون گاهی فقط «یک تابع پاس بده» است. اگر الگوها را با لهجه‌ی جاوا در پایتون پیاده کنید، کدتان پرتشریفات و غیرپایتونی می‌شود — این اشتباه آن‌قدر رایج است که در code reviewها برایش اسم داریم: «جاوا نوشتن با سینتکس پایتون».

پس قانون این فصل: برای هر الگو، اول **مسئله**، بعد **شکل کلاسیک**، بعد **میان‌بُر پایتونی** و اینکه کی کدام را انتخاب کنیم.

---

## دسته‌ی اول: الگوهای رفتاری

### Strategy — رفتار تعویض‌پذیر

**مسئله:** فروشگاهی داریم که هزینه‌ی ارسال را به روش‌های مختلف حساب می‌کند: پیک موتوری، پست، تیپاکس. روش محاسبه باید به‌راحتی عوض شود و روش جدید بدون دست‌کاری کد قبلی اضافه شود (همان Open/Closed فصل هشتم).

راه خام، `if/elif` است که با هر روش جدید متورم‌تر می‌شود. شکل کلاسیک Strategy، هر روش را یک کلاس با اینترفیس مشترک می‌کند:

```python
from abc import ABC, abstractmethod

class ShippingStrategy(ABC):
    @abstractmethod
    def calculate(self, weight_kg: float, distance_km: float) -> int: ...

class BikeCourier(ShippingStrategy):
    def calculate(self, weight_kg, distance_km):
        return 30_000 + int(distance_km * 5_000)

class NationalPost(ShippingStrategy):
    def calculate(self, weight_kg, distance_km):
        return 20_000 + int(weight_kg * 15_000)

class Order:
    def __init__(self, weight_kg, distance_km, shipping: ShippingStrategy):
        self.weight_kg = weight_kg
        self.distance_km = distance_km
        self.shipping = shipping          # the strategy is injected

    def shipping_cost(self):
        return self.shipping.calculate(self.weight_kg, self.distance_km)

order = Order(2, 12, BikeCourier())
print(order.shipping_cost())              # 90000
order.shipping = NationalPost()           # swap behavior at runtime
print(order.shipping_cost())              # 50000
```

**میان‌بُر پایتونی:** وقتی استراتژی فقط «یک عمل» است و حالت داخلی ندارد، کلاس تشریفات اضافه است. تابع پاس بدهید:

```python
def bike_courier(weight_kg, distance_km):
    return 30_000 + int(distance_km * 5_000)

def national_post(weight_kg, distance_km):
    return 20_000 + int(weight_kg * 15_000)

class Order:
    def __init__(self, weight_kg, distance_km, shipping):
        self.weight_kg = weight_kg
        self.distance_km = distance_km
        self.shipping = shipping          # any callable

    def shipping_cost(self):
        return self.shipping(self.weight_kg, self.distance_km)

order = Order(2, 12, shipping=bike_courier)
print(order.shipping_cost())              # 90000
```

**کدام را انتخاب کنیم؟** اگر استراتژی داده/پیکربندی خودش را دارد (مثلاً تعرفه‌هایی که از دیتابیس می‌آیند) یا چند متد مرتبط دارد → کلاس. اگر یک تابع خالص ساده است → تابع. این اولین نمونه از قاعده‌ی کلی فصل است: **در پایتون، ساده‌ترین چیزی که کار می‌کند، پایتونی‌ترین است.**

### Observer — خبررسانی بدون وابستگی

**مسئله:** وقتی سفارشی ثبت می‌شود، باید ایمیل تأیید برود، موجودی انبار کم شود و آمار فروش به‌روز شود. راه خام این است که `OrderService` هر سه کار را صدا بزند — یعنی به هر سه ماژول وابسته شود و با هر نیاز جدید (پیامک؟ کد تخفیف وفاداری؟) دوباره جراحی شود.

Observer این وابستگی را وارونه می‌کند: سوژه فقط «رویداد» را اعلام می‌کند؛ هر که خواست، گوش می‌دهد.

```python
class OrderPlaced:
    """A tiny event object: what happened, with which data."""
    def __init__(self, order_id, total):
        self.order_id = order_id
        self.total = total

class EventBus:
    def __init__(self):
        self._subscribers = {}            # event type -> list of callables

    def subscribe(self, event_type, handler):
        self._subscribers.setdefault(event_type, []).append(handler)

    def publish(self, event):
        for handler in self._subscribers.get(type(event), []):
            handler(event)

bus = EventBus()

# each part of the system subscribes independently:
bus.subscribe(OrderPlaced, lambda e: print(f"email: order {e.order_id} confirmed"))
bus.subscribe(OrderPlaced, lambda e: print(f"inventory: reserve items of {e.order_id}"))
bus.subscribe(OrderPlaced, lambda e: print(f"analytics: +{e.total}"))

# the order service knows nothing about email/inventory/analytics:
bus.publish(OrderPlaced(101, 750_000))
# email: order 101 confirmed
# inventory: reserve items of 101
# analytics: +750000
```

به وابستگی‌ها دقت کنید: سرویس سفارش فقط `bus` را می‌شناسد. افزودن شنونده‌ی جدید، حتی یک خط کد قدیمی را لمس نمی‌کند. این الگو قلب سیستم‌های event-driven است و در پروژه‌ی ششم دوره یک موتور اعلان کامل بر همین اساس می‌سازیم — آنجا نکته‌های حرفه‌ای‌ترش (مثل اینکه چرا نگه‌داشتن ارجاع مستقیم به شنونده‌ها می‌تواند نشت حافظه بسازد و `weakref` فصل چهاردهم چه کمکی می‌کند) را هم می‌بینید.

**Tradeoff:** جریان اجرا غیرمستقیم می‌شود؛ برای فهمیدن «بعد از ثبت سفارش چه می‌شود» باید فهرست مشترکان را پیدا کنید. در سیستم‌های کوچک، فراخوانی مستقیم صادقانه‌تر است.

### Template Method — اسکلت ثابت، جزئیات متغیر

**مسئله:** سه نوع گزارش خروجی داریم (CSV، JSON، HTML) که همه یک مراحل ثابت دارند — بارگیری داده، فیلتر، قالب‌بندی، ذخیره — و فقط قالب‌بندی‌شان فرق می‌کند.

Template Method می‌گوید: اسکلت الگوریتم را در کلاس پایه قفل کن و «جاهای خالی» را به زیرکلاس‌ها بسپار. با ابزار فصل نهم (ABC) پیاده می‌شود:

```python
from abc import ABC, abstractmethod

class ReportExporter(ABC):
    def export(self, records):            # the template: fixed skeleton
        data = self._only_active(records)
        body = self._format(data)         # the hook: subclasses fill this
        return f"{self._header()}\n{body}"

    def _only_active(self, records):      # shared step
        return [r for r in records if r.get("active")]

    def _header(self):                    # a hook with a default
        return "# report"

    @abstractmethod
    def _format(self, data): ...          # a hook subclasses MUST fill

class CsvExporter(ReportExporter):
    def _format(self, data):
        return "\n".join(f"{r['name']},{r['score']}" for r in data)

class JsonExporter(ReportExporter):
    def _format(self, data):
        import json
        return json.dumps(data, ensure_ascii=False)

records = [{"name": "ali", "score": 18, "active": True},
           {"name": "sara", "score": 19, "active": False}]
print(CsvExporter().export(records))
# # report
# ali,18
```

زیرکلاس نمی‌تواند ترتیب مراحل را خراب کند — اسکلت در والد قفل است. جاهایی که فریمورک شما را وادار می‌کند «فقط این متد را override کن» (مثل `save` در مدل‌های Django یا هوک‌های تست فریمورک‌ها)، همین الگوست.

**Tradeoff:** Template Method بر ارث‌بری سوار است با همه‌ی شکنندگی‌هایش (فصل ششم و هفتم). اگر «جای خالی» فقط یکی است، پاس‌دادن یک تابع (همان Strategy) اغلب ساده‌تر است.

### State — وقتی رفتار به وضعیت بستگی دارد

**مسئله:** یک سند در سیستم اداری سه وضعیت دارد: پیش‌نویس، در انتظار تأیید، منتشرشده. رفتار `edit` و `publish` در هر وضعیت فرق می‌کند و پر از قوانین است: سند منتشرشده ویرایش نمی‌شود، سند پیش‌نویس مستقیم منتشر نمی‌شود و...

راه خام: `if self.status == "draft": ... elif ...` در تک‌تک متدها — همان جهنم elif، این‌بار ضرب‌در تعداد متدها. الگوی State هر وضعیت را یک کلاس می‌کند و شیء، رفتارش را به وضعیت فعلی‌اش **واگذار** می‌کند:

```python
class DocumentState:
    def edit(self, doc, text):
        raise PermissionError(f"cannot edit in {type(self).__name__}")
    def submit(self, doc):
        raise PermissionError(f"cannot submit in {type(self).__name__}")
    def publish(self, doc):
        raise PermissionError(f"cannot publish in {type(self).__name__}")

class Draft(DocumentState):
    def edit(self, doc, text):
        doc.content = text
    def submit(self, doc):
        doc.state = PendingReview()

class PendingReview(DocumentState):
    def publish(self, doc):
        doc.state = Published()

class Published(DocumentState):
    pass                                   # everything is forbidden here

class Document:
    def __init__(self):
        self.content = ""
        self.state = Draft()

    # delegate behavior to the current state:
    def edit(self, text):  self.state.edit(self, text)
    def submit(self):      self.state.submit(self)
    def publish(self):     self.state.publish(self)

doc = Document()
doc.edit("first version")     # fine — Draft allows it
doc.submit()
# doc.edit("oops")            # PermissionError: cannot edit in PendingReview
doc.publish()                 # fine — PendingReview allows it
```

قوانین هر وضعیت یک‌جا جمع شده‌اند و افزودن وضعیت جدید (مثلاً «بایگانی‌شده») یعنی یک کلاس جدید، نه جراحی همه‌ی متدها. در پروژه‌ی سوم دوره، چرخه‌ی صندلی (آزاد/رزرو/فروخته‌شده) را با نسخه‌ی سبک‌تر همین ایده — `Enum` + بررسی گذارهای مجاز — ساختیم؛ وقتی رفتارها پیچیده‌تر شوند، ارتقا به کلاس‌های State طبیعی است.

### Command و Null Object — دو الگوی کوچک پرکاربرد

**Command** یعنی «درخواست را به یک شیء تبدیل کن» تا بشود صف‌اش کرد، لاگش کرد یا undo کرد. در پایتون معمولاً یک `dataclass` با متد `execute` (یا حتی یک تابع بسته‌بندی‌شده) کافی است — ویرایشگرها و سیستم‌های صف کار (task queue) بر همین سوارند.

**Null Object** ظریف‌تر است و کدهای واقعی را تمیز می‌کند: به‌جای `None` و باران `if x is not None`، یک نسخه‌ی «هیچ‌کاره» از شیء بدهید:

```python
class NullLogger:
    def log(self, message):
        pass                               # do nothing, silently

class OrderService:
    def __init__(self, logger=None):
        self.logger = logger or NullLogger()   # never None again

    def place(self, order_id):
        self.logger.log(f"placing {order_id}")
        return "ok"

OrderService().place(1)                    # works without any if-checks
```

`OrderService` دیگر هرگز نمی‌پرسد «لاگر دارم یا نه؟». یک شرط تکراری در سراسر کلاس حذف شد.

---

## دسته‌ی دوم: الگوهای ساختی (Creational)

### Factory — تولد شیء را متمرکز کن

**مسئله:** بر اساس ورودی کاربر یا فایل پیکربندی، باید یکی از چند کلاس ساخته شود (درگاه پرداخت «سامان» یا «ملت» یا «زرین‌پال»). اگر منطق انتخاب در سراسر کد پخش شود، افزودن درگاه جدید یعنی جست‌وجوی کل پروژه.

ساده‌ترین شکل — **تابع کارخانه** — همان است که در فصل هجدهم برای پکیج‌ها هم می‌بینید:

```python
class SamanGateway:
    def pay(self, amount): return f"saman: {amount}"

class MellatGateway:
    def pay(self, amount): return f"mellat: {amount}"

def create_gateway(name: str):
    gateways = {"saman": SamanGateway, "mellat": MellatGateway}
    try:
        return gateways[name]()
    except KeyError:
        raise ValueError(f"unknown gateway: {name}") from None

gw = create_gateway("saman")
print(gw.pay(10_000))          # saman: 10000
```

دو نکته در همین چند خط: کلاس‌ها در پایتون خودشان callable و درجه‌یک‌اند، پس دیکشنری «نام → کلاس» طبیعی‌ترین Factory است؛ و `from None` (فصل یازدهم) زنجیره‌ی استثنای بی‌ربط را قطع می‌کند.

شکل دوم را از فصل پنجم می‌شناسید: **`classmethod` به‌عنوان Factory** (`Invoice.from_dict(...)`) — وقتی راه‌های مختلف ساخت *همان یک* کلاس را می‌خواهید.

و شکل سوم، حرفه‌ای‌ترین: **Factory خود-ثبت‌شونده** با `__init_subclass__` فصل ششم، که دیکشنری را هم خودکار می‌کند:

```python
class Gateway:
    registry = {}

    def __init_subclass__(cls, key=None, **kwargs):
        super().__init_subclass__(**kwargs)
        if key:
            Gateway.registry[key] = cls    # every subclass registers itself

class Saman(Gateway, key="saman"):
    def pay(self, amount): return f"saman: {amount}"

class Zarinpal(Gateway, key="zarinpal"):
    def pay(self, amount): return f"zarinpal: {amount}"

def create_gateway(name):
    return Gateway.registry[name]()

print(create_gateway("zarinpal").pay(5_000))   # zarinpal: 5000
# adding a new gateway = just defining its class. nothing else to edit.
```

این الگوی **Registry** ستون سیستم‌های پلاگینی است — و دقیقاً همان چیزی که در پروژه‌ی چهارم (ساخت ORM) برای ثبت خودکار مدل‌ها به‌کار می‌بریم.

### Builder — ساخت مرحله‌به‌مرحله

**مسئله‌ی کلاسیک:** سازنده‌ای با ده پارامتر اختیاری که فراخوانی‌اش ناخواناست. در جاوا برایش کلاس Builder با متدهای زنجیره‌ای می‌سازند. در پایتون؟ **آب می‌رود**: آرگومان‌های نام‌دار و مقدار پیش‌فرض همان مشکل را حل می‌کنند:

```python
class Report:
    def __init__(self, title, format="pdf", include_charts=False, page_size="A4"):
        self.title = title
        self.format = format
        self.include_charts = include_charts
        self.page_size = page_size

# you rarely need a Builder class in Python. keyword arguments ARE the builder:
report = Report(
    title="Sales Q4",
    format="pdf",
    include_charts=True,
    page_size="A4",
)
```

Builder واقعی فقط وقتی می‌ارزد که ساخت واقعاً **چندمرحله‌ای و دارای اعتبارسنجی میانی** باشد (مثل ساختن یک Query پیچیده — `query.filter(...).order_by(...).limit(10)` در SQLAlchemy همین است). درس مهم‌تر از خود الگو: قبل از وارد کردن هر الگو بپرسید «زبان، خودش این مشکل را حل نکرده؟»

### Singleton — الگویی که باید با احتیاط دوست داشت

Singleton یعنی «از این کلاس فقط یک نمونه وجود داشته باشد». شکل فنی‌اش با `__new__` (فصل دوم):

```python
class AppConfig:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

a = AppConfig()
b = AppConfig()
print(a is b)     # True — the same single object
```

اما دو حقیقت صادقانه: اول، پایتونی‌ترین Singleton خود **ماژول** است (فصل هجدهم — ماژول‌ها ذاتاً یک‌بار ساخته می‌شوند). دوم، Singleton در عمل یک **حالت سراسری پنهان** است: هر که از هر جا به آن دست می‌زند، و تست‌ها به هم وابسته می‌شوند چون حالتش بینشان باقی می‌ماند — در فصل بعد (تست‌نویسی) دقیقاً می‌بینید که چرا تست کد Singleton‌زده دردناک است و تزریق وابستگی (فصل هشتم) جایگزین سالم‌تری است. Singleton را بشناسید — در مصاحبه‌ها محبوب است — اما در طراحی، آخرین گزینه‌تان باشد.

---

## دسته‌ی سوم: الگوهای ساختاری

### Adapter — مترجم بین دو اینترفیس

**مسئله:** کد شما با اینترفیس `send(to, text)` کار می‌کند، اما کتابخانه‌ی پیامک جدیدی که خریده‌اید متد `deliver_sms(payload: dict)` دارد. کد خودتان را عوض نکنید؛ **مترجم** بسازید:

```python
# their code (we cannot change it):
class ForeignSmsLib:
    def deliver_sms(self, payload: dict):
        return f"delivered to {payload['phone']}"

# our expected interface: send(to, text)
class SmsAdapter:
    def __init__(self, lib: ForeignSmsLib):
        self._lib = lib

    def send(self, to, text):                       # our interface...
        payload = {"phone": to, "message": text}
        return self._lib.deliver_sms(payload)       # ...their language

notifier = SmsAdapter(ForeignSmsLib())
print(notifier.send("0912...", "hi"))    # delivered to 0912...
```

Adapter چیزی جز Composition (فصل هفتم) + یک لایه‌ی ترجمه نیست. هر جا «کد ما» و «کد آن‌ها» زبان هم را نمی‌فهمند — کتابخانه‌ی خارجی، سیستم قدیمی (legacy)، API شخص ثالث — Adapter همان‌جاست. به‌لطف Duck Typing نیازی نیست Adapter از کلاسی ارث ببرد؛ کافی است متد درست را داشته باشد (و اگر خواستید قرارداد را رسمی کنید، `Protocol` فصل نهم).

### Decorator (الگو) — لایه‌لایه افزودن رفتار

**مسئله:** به کلاس ارسال اعلان می‌خواهید لاگ اضافه کنید. و جداگانه retry. و جداگانه سنجش زمان. ارث‌بری برای هر ترکیب یک کلاس می‌خواهد (انفجار ترکیبی — فصل هفتم). الگوی Decorator هر قابلیت را یک **پوشش (wrapper)** می‌کند که همان اینترفیس را دارد:

```python
class EmailNotifier:
    def send(self, message):
        return f"email: {message}"

class WithLogging:
    def __init__(self, inner):
        self.inner = inner                 # wraps any notifier

    def send(self, message):
        print(f"[log] sending: {message}")
        return self.inner.send(message)

class WithRetry:
    def __init__(self, inner, attempts=3):
        self.inner = inner
        self.attempts = attempts

    def send(self, message):
        for i in range(self.attempts):
            try:
                return self.inner.send(message)
            except ConnectionError:
                print(f"[retry] attempt {i + 1} failed")
        raise ConnectionError("all attempts failed")

# compose layers freely, like wrapping a gift:
notifier = WithRetry(WithLogging(EmailNotifier()))
print(notifier.send("hello"))
# [log] sending: hello
# email: hello
```

ربطش با `@decorator`های فصل پنجم؟ ایده یکی است — «بپیچ و رفتار اضافه کن» — فقط آنجا دور *تابع* می‌پیچیدیم و با سینتکس `@`؛ اینجا دور *شیء*، در زمان اجرا و به هر ترکیبی که بخواهیم. وقتی رفتار افزودنی به حالت یا ترکیب پویا نیاز دارد، الگوی شیءگرا؛ وقتی یک پوشش ساده دور تابع کافی است، `@decorator`.

### Facade — نمای ساده روی سیستم شلوغ

**مسئله:** ثبت یک سفارش یعنی رقص هماهنگ چند زیرسیستم: موجودی، پرداخت، ارسال، اعلان. اگر هر بخش برنامه خودش این رقص را اجرا کند، هم تکرار داریم هم خطا. Facade یک در ورودی ساده جلوی این پیچیدگی می‌گذارد:

```python
class OrderFacade:
    """One simple door in front of four subsystems."""
    def __init__(self, inventory, payment, shipping, notifier):
        self.inventory = inventory
        self.payment = payment
        self.shipping = shipping
        self.notifier = notifier

    def place_order(self, user, items):
        self.inventory.reserve(items)
        receipt = self.payment.charge(user, items)
        tracking = self.shipping.schedule(user.address, items)
        self.notifier.send(user, f"order confirmed: {tracking}")
        return receipt
```

کاربر این کلاس فقط `place_order` را می‌بیند؛ جزئیات چهار زیرسیستم پشت نما پنهان است — همان **انتزاع** فصل چهارم، در مقیاس سیستم. اگر دقت کنید، لایه‌ی Service در معماری لایه‌ای فصل هفتم (و `Library` در پروژه‌ی اول) دقیقاً نقش Facade را بازی می‌کنند؛ گاهی سال‌هاست از الگویی استفاده می‌کنید و فقط اسمش را نمی‌دانستید.

### Proxy — نماینده‌ای با اختیارات

Proxy همان اینترفیس شیء اصلی را دارد، اما بین شما و آن می‌ایستد تا کاری اضافه کند: کنترل دسترسی، کش، بارگذاری تنبل. اسکلتش عین Decorator است؛ فرق در **نیت** است — Decorator «رفتار اضافه می‌کند»، Proxy «دسترسی را مدیریت می‌کند»:

```python
class ReportService:
    def sales_report(self):
        print("(expensive query running...)")
        return "sales data"

class CachedReportService:
    def __init__(self, inner):
        self.inner = inner
        self._cache = None

    def sales_report(self):
        if self._cache is None:                    # first call only
            self._cache = self.inner.sales_report()
        return self._cache

service = CachedReportService(ReportService())
service.sales_report()     # (expensive query running...)
service.sales_report()     # silent — served from cache
```

آشناست؟ `@cached_property` فصل پنجم نسخه‌ی فشرده‌ی همین ایده بود. و اگر ذهنتان رفت سراغ `__getattr__` فصل پنجم برای ساختن Proxy عمومی که «هر متد ناشناخته را به شیء داخلی بفرست» — آفرین، دقیقاً همان‌جا به‌کار می‌آید.

---

## هشدار: تب الگو (Pattern Fever)

بیماری شناخته‌شده‌ای بین کسانی که تازه الگوها را یاد گرفته‌اند: همه‌جا الگو می‌بینند و همه‌چیز را الگو می‌کنند. اسکریپت پنجاه‌خطی با AbstractFactory و Singleton و Observer — چیزی که باید نیم‌ساعت خوانده شود، نیم‌روز خوانده می‌شود.

سه قاعده برای واکسینه‌شدن:

- الگو **پاسخ به یک درد** است. اول درد را حس کنید (تکرار، شکنندگی، وابستگی خفه‌کننده)، بعد الگو بزنید. کدی که هنوز دردی ندارد، الگو هم نمی‌خواهد.
- الگوها **واژگان‌اند، نه چک‌لیست**. قرار نیست پروژه‌تان «همه‌ی الگوها را داشته باشد»؛ قرار است وقتی الگویی دیدید یا لازمش داشتید، بشناسیدش.
- **میان‌بُر پایتونی را اول چک کنید.** تابع درجه‌یک، دیکشنری، ماژول، دکوراتور — خیلی وقت‌ها خود زبان، الگو را یک‌خطی کرده.

و نکته‌ای که تجربه به آدم یاد می‌دهد: در code reviewها، کد بیش‌ازحد الگوزده را باید همان‌قدر جدی نقد کرد که کد بی‌ساختار را. اولی پیچیدگی خودخواسته است، دومی پیچیدگی ناخواسته — و اتفاقاً اولی سخت‌تر اصلاح می‌شود، چون نویسنده‌اش به آن افتخار می‌کند.

---

## خلاصه

| درد | الگو | میان‌بُر پایتونی |
|------|------|------------------|
| رفتار تعویض‌پذیر | Strategy | پاس‌دادن تابع |
| خبررسانی به چند بخش مستقل | Observer | — (خودش ساده است) |
| اسکلت ثابت + جزئیات متغیر | Template Method | پاس‌دادن تابع برای تک‌هوک |
| رفتار وابسته به وضعیت | State | `Enum` + گذارهای مجاز (نسخه‌ی سبک) |
| نبودن شیء (`None`چک‌های تکراری) | Null Object | — |
| تمرکز منطق ساخت | Factory | دیکشنری «نام → کلاس»؛ `classmethod`؛ Registry با `__init_subclass__` |
| سازنده‌ی شلوغ | Builder | آرگومان‌های نام‌دار |
| فقط یک نمونه | Singleton | ماژول (و اغلب: اصلاً نه — تزریق وابستگی) |
| دو اینترفیس ناسازگار | Adapter | Composition + Duck Typing |
| افزودن رفتار لایه‌به‌لایه | Decorator | `@decorator` برای توابع |
| نمای ساده روی زیرسیستم‌ها | Facade | — (لایه‌ی Service همین است) |
| مدیریت دسترسی به شیء | Proxy | `__getattr__`، `@cached_property` |

**نکته‌ی طلایی:** الگوها را با «درد»شان حفظ کنید نه با UMLشان. در مصاحبه و در کار واقعی، هنر این نیست که تعریف Adapter را بگویید؛ این است که وسط کد، ناسازگاری دو اینترفیس را ببینید و بگویید «اینجا جای Adapter است» — یا شجاعانه بگویید «اینجا هیچ الگویی لازم نیست».

---

## پرسش‌های مصاحبه

این سوال‌ها در مصاحبه‌های واقعی استخدام پایتون‌کار تکرار می‌شوند؛ قبل از خواندن پاسخ، خودتان جواب دهید.

**۱. «الگوی Singleton را پیاده کنید و بعد توضیح دهید چرا استفاده‌اش اغلب ایده‌ی بدی است.»** — پیاده‌سازی با `__new__` و `_instance` (بالا دیدید). نقد: حالت سراسری پنهان می‌سازد، وابستگی‌ها را نامرئی می‌کند و تست‌ها را به هم گره می‌زند؛ در پایتون، ماژول یا تزریق وابستگی معمولاً جایگزین سالم‌تر است. مصاحبه‌گر خوب دقیقاً دنبال همین بخش دوم است.

**۲. «Strategy را در پایتون چطور پیاده می‌کنید؟»** — پاسخ ممتاز دو سطح دارد: شکل کلاسیک با ABC برای استراتژی‌های حالت‌دار، و اشاره به اینکه در پایتون برای استراتژی ساده، پاس‌دادن تابع کافی است چون توابع درجه‌یک‌اند. گفتن همین تمایز، شما را از حفظ‌کننده‌ها جدا می‌کند.

**۳. «فرق Adapter و Decorator و Proxy چیست؟ اسکلتشان که شبیه است.»** — هر سه «شیئی دور شیء دیگر»اند؛ فرق در نیت: Adapter اینترفیس را **تغییر** می‌دهد (ترجمه)، Decorator همان اینترفیس را نگه می‌دارد و رفتار **اضافه** می‌کند، Proxy همان اینترفیس را نگه می‌دارد و دسترسی را **مدیریت** می‌کند (کش/تنبلی/مجوز).

**۴. «در Django یا SQLAlchemy چه الگوهایی می‌بینید؟»** — چند نمونه‌ی قابل‌دفاع: QuerySetسازی زنجیره‌ای (Builder)، سیگنال‌های Django (Observer)، `Model.objects` (Facade/Repository)، فیلدهای مدل (Descriptor — فصل پانزدهم)، ثبت خودکار مدل‌ها (Registry/متاکلاس — فصل شانزدهم). لازم نیست همه را بگویید؛ دو تا را عمیق توضیح دهید بهتر از پنج اسم خالی است.

---

**در فصل بعد:** الگوها به ما «طراحی خوب» دادند؛ اما از کجا بدانیم کدمان واقعاً درست کار می‌کند و با تغییر بعدی نمی‌شکند؟ وقت یادگیری مهارتی است که در آگهی‌های استخدام کنار خود پایتون می‌نشیند: **تست‌نویسی برای کد شیءگرا** — با pytest، fixtureها و هنر جداکردن کد از دنیای بیرون.
