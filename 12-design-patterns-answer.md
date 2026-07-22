# پاسخ تمرین‌های فصل دوازدهم: الگوهای طراحی

---

## تمرین ۱ - پرسش‌های مفهومی

**۱.** الگوی طراحی، راه‌حل نام‌دار و آزموده برای یک مسئله‌ی تکرارشونده‌ی طراحی است. سه فایده: **واژگان مشترک** (انتقال یک پاراگراف طراحی در دو کلمه، در جلسه و مصاحبه)، **راه‌حل آزموده** (چاله‌هایش شناخته شده)، و **درک فریمورک‌ها** (Django و SQLAlchemy و pytest از همین الگوها ساخته شده‌اند).

**۲.** یعنی الگوها را با همان تشریفاتی پیاده کنیم که زبان‌های بدون تابع درجه‌یک و Duck Typing مجبورند: اینترفیس و کلاس برای هر چیز کوچک. در پایتون بعضی الگوها آب می‌روند چون خود زبان مشکل را حل کرده. دو نمونه: **Strategy** ساده که به «پاس‌دادن تابع» تبدیل می‌شود، و **Builder** که آرگومان‌های نام‌دار با مقدار پیش‌فرض جایش را می‌گیرند. (نمونه‌ی سوم: Singleton که ماژول جایش را می‌گیرد.)

**۳.** بیماری «همه‌چیز را الگو کردن» بعد از یادگیری الگوهاست — اسکریپت ساده‌ای که زیر بار Factory و Observer ناخوانا می‌شود. سه قاعده: الگو **پاسخ به درد** است، اول درد را حس کنید؛ الگوها **واژگان‌اند نه چک‌لیست**؛ و **اول میان‌بُر پایتونی را چک کنید**.

**۴.** هر سه یک شیء را دور شیء دیگر می‌پیچند؛ تفاوت در نیت: **Adapter** اینترفیس را عوض می‌کند (ترجمه بین دو زبان ناسازگار)، **Decorator** همان اینترفیس را نگه می‌دارد و رفتار اضافه می‌کند (لاگ، retry)، **Proxy** همان اینترفیس را نگه می‌دارد و دسترسی را مدیریت می‌کند (کش، بارگذاری تنبل، مجوز).

**۵.** چون ماژول در پایتون فقط یک‌بار load و بعد کش می‌شود؛ هر `import` بعدی همان شیء را می‌دهد — دقیقاً معنای Singleton، بدون حتی یک خط کد اضافه. مشکل تست: Singleton حالت سراسری است؛ حالت باقی‌مانده از یک تست به تست بعدی نشت می‌کند و ترتیب اجرای تست‌ها روی نتیجه اثر می‌گذارد — کابوس «تستی که تنها پاس می‌شود و کنار بقیه می‌شکند». (فصل سیزدهم این را عملاً می‌بینید.)

---

## تمرین ۲ - تشخیص الگو

**۶.**

- الف) **Strategy** — و چون هر سه فرمول بدون حالت‌اند، میان‌بُر: سه تابع.
- ب) **Observer** (یا EventBus) — ثبت‌نام فقط رویداد منتشر می‌کند.
- ج) **Adapter** — مترجم `pay(amount, card)` به `do_payment(data_dict)`.
- د) **Template Method** — اسکلت در والد، مرحله‌ی سوم `@abstractmethod`. (میان‌بُر برای تک‌هوک: پاس‌دادن تابع.)
- ه) **Proxy** (نوع کش‌کننده) — همان اینترفیس، مدیریت دسترسی. میان‌بُر: `@cached_property` اگر داخل خود کلاس ممکن بود.
- و) **Null Object** — یک `NoDiscount` بی‌اثر بده تا `if`ها حذف شوند.

**۷.**

- الف) الگو لازم نیست؛ دردی وجود ندارد. همان چند تابع ساده خواناتر است. (تب الگو!)
- ب) Builder لازم نیست؛ آرگومان‌های نام‌دار با پیش‌فرض همین کار را می‌کنند: `Report(title=..., format=..., ...)`.
- ج) سلسله‌مراتب ABC لازم نیست؛ یک تابع ساده کافی است و اگر روزی استراتژی حالت‌دار آمد، آن موقع کلاس می‌شود.

**۸.** ترکیب **Facade** (یک در ساده — `checkout` — جلوی هماهنگی چند مؤلفه) با **تزریق وابستگی** (gateway و logger از بیرون می‌آیند — DIP فصل هشتم). ضعف احتمالی: اگر منطق هماهنگی رشد کند، `PaymentService` می‌تواند به God Object میل کند؛ باید مراقب مرز مسئولیتش بود. (اشاره به Adapter هم پذیرفتنی است اگر استدلال کنید gateway بیگانه است — مهم استدلال است.)

---

## تمرین ۳ - کدنویسی

**۹.**

```python
class TicketPricer:
    def __init__(self, strategy):
        self.strategy = strategy          # any callable: base -> price

    def price(self, base):
        return self.strategy(base)

def weekend(base):
    return int(base * 1.2)

def student(base):
    return base // 2

pricer = TicketPricer(weekend)
print(pricer.price(100_000))     # 120000
pricer.strategy = student        # swap at runtime
print(pricer.price(100_000))     # 50000
```

**۱۰.**

```python
class EventBus:
    def __init__(self):
        self._subscribers = {}

    def subscribe(self, event_name, handler):
        self._subscribers.setdefault(event_name, []).append(handler)

    def publish(self, event_name, data):
        for handler in self._subscribers.get(event_name, []):
            handler(data)

bus = EventBus()
bus.subscribe("user_registered", lambda d: print(f"welcome email to {d['email']}"))
bus.subscribe("user_registered", lambda d: print(f"profile created for {d['username']}"))

bus.publish("user_registered", {"username": "ali", "email": "ali@example.com"})
# welcome email to ali@example.com
# profile created for ali
```

**راه دیگر:** به‌جای `lambda`، توابع یا حتی متدهای کلاس‌ها را ثبت کنید — هر callable قبول است.

**۱۱.**

```python
class Exporter:
    registry = {}

    def __init_subclass__(cls, key=None, **kwargs):
        super().__init_subclass__(**kwargs)
        if key:
            Exporter.registry[key] = cls

class CsvExporter(Exporter, key="csv"):
    def export(self, data):
        return "csv output"

class JsonExporter(Exporter, key="json"):
    def export(self, data):
        return "json output"

def create_exporter(key):
    try:
        return Exporter.registry[key]()
    except KeyError:
        raise ValueError(f"unknown exporter: {key}") from None

print(create_exporter("csv").export([]))    # csv output
print(Exporter.registry)                    # {'csv': ..., 'json': ...}
```

**۱۲.**

```python
class NullNotifier:
    def send(self, message):
        pass                                # silently do nothing

class OrderService:
    def __init__(self, notifier=None):
        self.notifier = notifier or NullNotifier()

    def place(self, order_id):
        # no `if self.notifier is not None` anywhere:
        self.notifier.send(f"order {order_id} placed")
        return "ok"

print(OrderService().place(1))              # ok — no crash, no ifs
```

**۱۳.**

```python
class LegacyPrinter:                        # (the foreign class, for reference)
    def print_document(self, content: bytes, copies: int):
        return f"printing {copies} copies of {len(content)} bytes"

class PrinterAdapter:
    def __init__(self, legacy: LegacyPrinter):
        self._legacy = legacy

    def print_text(self, text: str):
        content = text.encode("utf-8")      # translate: str -> bytes
        return self._legacy.print_document(content, copies=1)

adapter = PrinterAdapter(LegacyPrinter())
print(adapter.print_text("hello"))          # printing 1 copies of 5 bytes
```

---

## تمرین ۴ - تحلیل و مقایسه

**۱۴.**

| درد | الگو | میان‌بُر پایتونی |
|------|------|------------------|
| رفتار تعویض‌پذیر | Strategy | پاس‌دادن تابع |
| سازنده‌ی شلوغ با ده پارامتر | Builder | آرگومان‌های نام‌دار با پیش‌فرض |
| دو اینترفیس ناسازگار | Adapter | Composition + Duck Typing (کلاس کوچک مترجم) |
| فقط یک نمونه از کلاس | Singleton | ماژول؛ و اغلب تزریق وابستگی به‌جایش |

**۱۵.** نسخه‌ی سبک (`Enum` + جدول گذارهای مجاز) وقتی کافی است که وضعیت‌ها عمدتاً «برچسب»‌اند و منطق هر وضعیت یکی‌دو شرط ساده است. الگوی State وقتی می‌ارزد که **رفتار چند متد** در هر وضعیت فرق کند و قوانین هر وضعیت آن‌قدر رشد کرده باشد که `if`های پراکنده در متدها تکرار شوند. معیار ارتقا: وقتی دیدید دارید همان `if status == ...` را در متد سوم و چهارم هم می‌نویسید، وقت کلاس‌های State است — هر وضعیت، خانه‌ی قوانین خودش.

---

## تمرین ۵ - پرسش تحلیلی

**۱۶.** ادعا وارونه‌ی حقیقت است. الگو «پاسخ به درد» است؛ کدی که دردی ندارد و الگو خورده، **پیچیدگی خودخواسته** دارد — لایه‌های انتزاعی که خواندن و دیباگ را کند می‌کنند بدون اینکه مشکلی حل کنند. اتفاقاً این نوع پیچیدگی از کد بی‌ساختار خطرناک‌تر است، چون پشت ظاهر «حرفه‌ای» پنهان می‌شود و نویسنده‌اش در برابر حذفش مقاومت می‌کند. معیار code review نه «تعداد الگوها» بلکه «تناسب راه‌حل با مسئله» است: کد پایتونی خوب اول میان‌بُرهای زبان را خرج می‌کند (تابع، دیکشنری، ماژول) و الگوی سنگین را برای درد واقعی نگه می‌دارد. جمله‌ی درست‌تر: «کد حرفه‌ای کدی است که به‌اندازه‌ی مسئله‌اش طراحی داشته باشد — نه کمتر، نه بیشتر.»
