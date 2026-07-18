# فصل بیست‌وچهارم — پروژه ششم: موتور اعلان Event-Driven

> آخرین پروژه‌ی دوره، سه رشته‌ی به‌ظاهر جدا را به هم می‌بافد: الگوی Observer (فصل ۱۲)، پروتکل‌ها (فصل ۹) و weakref (فصل ۱۴). نتیجه، معماری‌ای است که ستون سیستم‌های بزرگ واقعی است — از سیگنال‌های Django تا message brokerها — و در پایانش، دوره را رسماً می‌بندیم.

---

## مقدمه

در پروژه‌های قبل، هر بار که «بعد از فلان اتفاق، بهمان کار هم بشود» لازم شد، مستقیم صدایش زدیم: کتابخانه بعد از امانت، خودش رسید چاپ کرد؛ سیستم رزرو بعد از پرداخت، خودش وضعیت عوض کرد. برای آن مقیاس، درست هم بود.

اما سیستم‌های واقعی جای دیگری می‌شکنند: وقتی «بعد از فلان اتفاق»ها مدام زیاد می‌شوند. سفارش ثبت شد؟ ایمیل برود، پیامک برود، انبار به‌روز شود، آمار ثبت شود، کد تخفیف وفاداری حساب شود... و هر ماه یکی اضافه می‌شود. اگر همه‌ی این‌ها فراخوانی مستقیم باشند، کلاس سفارش کم‌کم به همه‌ی سیستم وابسته می‌شود — God Object فصل سوم، این‌بار از در وابستگی.

راه‌حل را در فصل دوازدهم دیدید: **رویداد منتشر کن، نگو چه کسی بشنود.** در این پروژه آن ایده را به یک موتور اعلان کامل قابل‌استفاده در production تبدیل می‌کنیم: با کانال‌های افزودنی، مقاوم در برابر خطای شنونده‌ها، و — بخش خاصش — بدون نشت حافظه.

---

## نیاز کارفرما

مدیر محصول یک فروشگاه اینترنتی رو به رشد:

> «سیستم اعلان‌هایمان قصه شده. الان کد ثبت سفارش مستقیماً ایمیل می‌فرستد؛ تیم مارکتینگ پیامک خواست، وسط همان کد اضافه شد؛ حالا اپ موبایل push می‌خواهد و من می‌ترسم باز همان‌جا دست بزنیم. می‌خواهم یک سیستم اعلان مرتب: هر اتفاق مهم (ثبت سفارش، ارسال، تأخیر) یک‌جا اعلام شود و کانال‌ها جدا باشند — امروز ایمیل و پیامک، فردا هر چیز دیگر، بدون دست‌زدن به کد اصلی. آهان، و دو چیز: اگر سرویس پیامک قطع بود، نباید ایمیل‌ها هم بمیرند؛ و کاربرها باید بتوانند بگویند از کدام کانال چه نوع خبری می‌خواهند.»

---

## تحلیل نیازمندی‌ها

**شما:** «"اتفاق مهم" دقیقاً چند نوع است و چه داده‌ای همراهش است؟»
**مدیر:** «فعلاً سه نوع: ثبت سفارش، ارسال سفارش، تأخیر. هرکدام شناسه‌ی سفارش و کاربر و یک متن.»

**شما:** «اگر ارسال به یک کانال شکست خورد چه؟ تلاش مجدد می‌خواهید؟»
**مدیر:** «یک‌بار تلاش مجدد کافی است. مهم‌تر این است که شکست یک کانال بقیه را متوقف نکند و جایی ثبت شود.»

**شما:** «ترجیحات کاربر کجا اعمال شود؟ داخل هر کانال یا جای مرکزی؟»
**مدیر:** «تو بگو — فقط نمی‌خواهم منطق "این کاربر پیامک نمی‌خواهد" ده جا کپی شود.»

**شما:** «مشترک‌ها (شنونده‌ها) در طول عمر برنامه می‌آیند و می‌روند؟ مثلاً داشبورد ادمین که فقط وقتی باز است باید آمار بگیرد؟»
**مدیر:** «دقیقاً همین‌طور است. قبلاً یک‌بار سر همین، سیستم بعد از یک هفته سنگین شد و کسی نفهمید چرا.»

آن جمله‌ی آخر را قاب کنید — «سنگین شد و کسی نفهمید چرا». این بوی **نشت حافظه از مسیر Observer** می‌دهد: شنونده‌هایی که مرده‌اند اما چون ناشر ارجاعشان را نگه داشته، هرگز جمع نمی‌شوند. فصل چهاردهم دقیقاً برای امروز بود.

## جمع‌بندی نیازمندی‌ها

| نیاز | جزئیات |
|------|--------|
| رویدادها | سه نوع + قابل‌گسترش؛ حامل داده‌ی سفارش/کاربر/متن |
| انتشار مرکزی | ناشر هیچ کانالی را نمی‌شناسد (Observer) |
| کانال‌های افزودنی | ایمیل/پیامک امروز؛ push فردا بدون تغییر هسته (OCP) |
| مقاومت در برابر خطا | شکست یک شنونده، بقیه را متوقف نکند؛ یک retry + ثبت خطا |
| ترجیحات کاربر | فیلتر مرکزی، نه کپی در هر کانال |
| بدون نشت حافظه | شنونده‌ی مرده، خودکار حذف شود (weakref) |

## پیش‌نیازهای دانشی

- **[فصل ۱۲: الگوهای طراحی](12-design-patterns.md)** — Observer/EventBus و Decorator (برای retry).
- **[فصل ۹: پروتکل‌ها](09-interfaces-protocols-generics.md)** — قرارداد کانال‌ها با `Protocol`.
- **[فصل ۱۴: مدیریت حافظه](14-memory-management.md)** — weakref و نشت حافظه‌ی Observer.
- **[فصل ۱۰: مدل‌های داده](10-modern-data-models.md)** — رویدادها با `dataclass` و `Enum`.
- **[فصل ۱۳: تست‌نویسی](13-testing-oop.md)** — تست موتور با Fakeها.

---

## پیاده‌سازی

### گام ۱: رویدادها — زبان مشترک سیستم

رویداد فقط حامل داده است؛ `dataclass` منجمد (فصل ۱۰) انتخاب طبیعی است — تغییرناپذیر، چون رویدادی که رخ داده نباید عوض شود:

```python
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum

class EventType(Enum):
    ORDER_PLACED = "order_placed"
    ORDER_SHIPPED = "order_shipped"
    ORDER_DELAYED = "order_delayed"

@dataclass(frozen=True)
class Event:
    type: EventType
    user_id: int
    order_id: int
    message: str
    occurred_at: datetime = field(default_factory=datetime.now)
```

### گام ۲: قرارداد کانال — Protocol به‌جای ارث‌بری

کانال‌ها را با `Protocol` (فصل ۹) قرارداد می‌کنیم، نه ABC. دلیل انتخاب: کانال فردا شاید کلاسی از یک کتابخانه‌ی بیرونی باشد که نمی‌توانیم وادار به ارث‌بری‌اش کنیم؛ Duck Typing رسمی‌شده اینجا آزادی می‌دهد:

```python
from typing import Protocol

class Channel(Protocol):
    name: str
    def deliver(self, user_id: int, text: str) -> None: ...

# two real channels. no inheritance — they just fit the shape:
class EmailChannel:
    name = "email"
    def deliver(self, user_id, text):
        print(f"[email to user {user_id}] {text}")

class SmsChannel:
    name = "sms"
    def deliver(self, user_id, text):
        print(f"[sms to user {user_id}] {text}")
```

### گام ۳: هسته — EventBus مقاوم در برابر خطا

قلب سیستم. نسخه‌ی فصل ۱۲ را برمی‌داریم و صنعتی‌اش می‌کنیم: خطای هر شنونده **قرنطینه** می‌شود و در گزارش می‌نشیند، نه اینکه انتشار را بکشد:

```python
class NotificationError(Exception):
    """base error of the notification engine"""

class EventBus:
    def __init__(self, logger=None):
        self._subscribers = {}                 # EventType -> list of handlers
        self._logger = logger or (lambda msg: None)   # Null Object, ch 12

    def subscribe(self, event_type: EventType, handler):
        self._subscribers.setdefault(event_type, []).append(handler)

    def publish(self, event: Event) -> int:
        """Deliver to all subscribers. One failure never stops the rest."""
        delivered = 0
        for handler in list(self._subscribers.get(event.type, [])):
            try:
                handler(event)
                delivered += 1
            except Exception as e:             # quarantine, log, continue
                self._logger(f"handler {handler!r} failed: {e}")
        return delivered
```

دو ظرافت: روی **کپی** لیست حلقه می‌زنیم (`list(...)`) تا اگر شنونده‌ای وسط کار subscribe/unsubscribe کرد، حلقه خراب نشود — باگی که فقط در production خودش را نشان می‌دهد. و `except Exception` اینجا — برخلاف قاعده‌ی معمول «استثنای پهن نگیر» — عمداً پهن است: مرز بین سیستم و کد غریبه دقیقاً جایی است که باید همه‌چیز را گرفت، ثبت کرد و زنده ماند. استثنا در قاعده هم درس است.

### گام ۴: بخش خاص — اشتراک بدون نشت حافظه

حالا همان مشکلی که سیستم قبلی کارفرما را «سنگین» کرد. وقتی داشبورد ادمین subscribe می‌کند و بعد بسته می‌شود، ارجاع داخل `_subscribers` آن را برای همیشه زنده نگه می‌دارد — Garbage Collector (فصل ۱۴) چیزی را که ارجاع دارد جمع نمی‌کند. راه‌حل: **weakref** — اشاره‌ای که مانع مرگ نمی‌شود:

```python
import weakref

class WeakSubscription:
    """Holds a bound method weakly; reports if its owner has died."""
    def __init__(self, bound_method):
        self._ref = weakref.WeakMethod(bound_method)   # ch 14

    def __call__(self, event):
        method = self._ref()               # try to revive the reference
        if method is None:
            raise _DeadSubscriber          # the owner is gone
        method(event)

class _DeadSubscriber(Exception):
    pass
```

و `publish` را یاد می‌دهیم که مرده‌ها را همان‌جا **خاک کند**:

```python
# add inside EventBus:
    def subscribe_weak(self, event_type: EventType, bound_method):
        self.subscribe(event_type, WeakSubscription(bound_method))

    def publish(self, event: Event) -> int:
        delivered = 0
        handlers = self._subscribers.get(event.type, [])
        for handler in list(handlers):
            try:
                handler(event)
                delivered += 1
            except _DeadSubscriber:
                handlers.remove(handler)   # bury it — no leak, no noise
            except Exception as e:
                self._logger(f"handler {handler!r} failed: {e}")
        return delivered
```

ببینیم واقعاً کار می‌کند — این تست، عین سناریوی «یک هفته بعد سنگین شد» است، فشرده در پنج خط:

```python
class AdminDashboard:
    def on_event(self, event):
        print(f"[dashboard] {event.message}")

bus = EventBus()
dash = AdminDashboard()
bus.subscribe_weak(EventType.ORDER_PLACED, dash.on_event)

e = Event(EventType.ORDER_PLACED, user_id=1, order_id=100, message="order placed")
print(bus.publish(e))     # [dashboard] order placed  →  1

del dash                  # the dashboard window is closed
print(bus.publish(e))     # 0 — and the dead subscription was auto-removed
```

چرا `WeakMethod` و نه `weakref.ref` ساده؟ ظرافتی از جنس فصل ۱۵: `dash.on_event` هر بار یک bound method **تازه** می‌سازد (`__get__`!)؛ `weakref.ref` به آن شیء لحظه‌ای، بلافاصله می‌میرد. `WeakMethod` مخصوص همین ساخته شده: شیء و تابع را جدا و ضعیف نگه می‌دارد. این‌جور جزئیات را فقط کسی می‌داند که زیر کاپوت را دیده — یعنی حالا، شما.

### گام ۵: کانال‌ها + ترجیحات + retry — سوارکردن قطعات

حالا قطعات بیرونی را به هم وصل کنیم. ترجیحات کاربر یک فیلتر مرکزی است و retry یک **Decorator** (فصل ۱۲) دور هر کانال — نه داخل هیچ‌کدامشان:

```python
class WithRetry:
    """Decorator pattern: any channel, one extra attempt."""
    def __init__(self, channel, attempts=2):
        self.name = channel.name
        self._channel = channel
        self._attempts = attempts

    def deliver(self, user_id, text):
        last_error = None
        for _ in range(self._attempts):
            try:
                return self._channel.deliver(user_id, text)
            except ConnectionError as e:
                last_error = e
        raise NotificationError(f"{self.name} failed after retries") from last_error

class Preferences:
    """user_id -> set of allowed channel names. Central, not copied per channel."""
    def __init__(self):
        self._allowed = {}

    def allow(self, user_id, *channel_names):
        self._allowed[user_id] = set(channel_names)

    def accepts(self, user_id, channel_name):
        return channel_name in self._allowed.get(user_id, set())

class ChannelSubscriber:
    """Glue: an event arrives -> if the user accepts this channel, deliver."""
    def __init__(self, channel, preferences):
        self._channel = channel
        self._prefs = preferences

    def __call__(self, event):                 # __call__: usable as a handler (ch 5)
        if self._prefs.accepts(event.user_id, self._channel.name):
            self._channel.deliver(event.user_id, event.message)
```

### گام ۶: همه‌چیز کنار هم — روز اول در production

```python
prefs = Preferences()
prefs.allow(1, "email", "sms")     # user 1 wants everything
prefs.allow(2, "email")            # user 2: email only

bus = EventBus(logger=print)
email = ChannelSubscriber(WithRetry(EmailChannel()), prefs)
sms = ChannelSubscriber(WithRetry(SmsChannel()), prefs)

for event_type in EventType:                   # all channels hear all events
    bus.subscribe(event_type, email)
    bus.subscribe(event_type, sms)

bus.publish(Event(EventType.ORDER_PLACED, user_id=1, order_id=100,
                  message="your order was placed"))
# [email to user 1] your order was placed
# [sms to user 1] your order was placed

bus.publish(Event(EventType.ORDER_SHIPPED, user_id=2, order_id=101,
                  message="your order was shipped"))
# [email to user 2] your order was shipped      ← no sms: user 2 declined it
```

و خواسته‌ی آخر کارفرما — «فردا push، بدون دست‌زدن به کد اصلی»؟ سه خط، صفر تغییر در هسته:

```python
class PushChannel:
    name = "push"
    def deliver(self, user_id, text):
        print(f"[push to user {user_id}] {text}")

push = ChannelSubscriber(WithRetry(PushChannel()), prefs)
for event_type in EventType:
    bus.subscribe(event_type, push)
prefs.allow(1, "email", "sms", "push")
```

Open/Closed (فصل ۸)، در گوشت و خون.

### گام ۷: تست — با همان Fakeهای فصل سیزدهم

موتور به هیچ سرویس واقعی‌ای وصل نیست، پس تستش لذت‌بخش است:

```python
class FakeChannel:
    name = "fake"
    def __init__(self):
        self.delivered = []
    def deliver(self, user_id, text):
        self.delivered.append((user_id, text))

def test_publish_respects_preferences():
    prefs = Preferences()
    prefs.allow(1, "fake")             # user 1 accepts, user 2 does not
    fake = FakeChannel()
    bus = EventBus()
    bus.subscribe(EventType.ORDER_PLACED, ChannelSubscriber(fake, prefs))

    bus.publish(Event(EventType.ORDER_PLACED, 1, 100, "hi"))
    bus.publish(Event(EventType.ORDER_PLACED, 2, 101, "hi"))

    assert fake.delivered == [(1, "hi")]

def test_one_broken_handler_does_not_stop_others():
    bus = EventBus()
    seen = []
    def broken(event): raise RuntimeError("boom")
    def healthy(event): seen.append(event.order_id)
    bus.subscribe(EventType.ORDER_PLACED, broken)
    bus.subscribe(EventType.ORDER_PLACED, healthy)

    delivered = bus.publish(Event(EventType.ORDER_PLACED, 1, 100, "hi"))

    assert delivered == 1 and seen == [100]    # the healthy one still ran
```

---

## جمع‌بندی پروژه

| بخش | مفهوم | فصل |
|-----|-------|-----|
| `Event` منجمد + `EventType` | `dataclass(frozen)`, `Enum` | ۱۰ |
| `EventBus` | Observer، قرنطینه‌ی خطا | ۱۲ |
| `Channel` | Protocol، Duck Typing رسمی | ۹ |
| `WeakSubscription` | weakref، `WeakMethod`، نشت حافظه | ۱۴، ۱۵ |
| `WithRetry` | الگوی Decorator | ۱۲ |
| `ChannelSubscriber.__call__` | شیء فراخوانی‌پذیر | ۵ |
| `logger` پیش‌فرض بی‌صدا | Null Object | ۱۲ |
| افزودن `PushChannel` | Open/Closed | ۸ |
| تست با `FakeChannel` | تست مستقل از دنیا | ۱۳ |

**درس اصلی:** این معماری — رویداد تغییرناپذیر، ناشر کور، شنونده‌های قرنطینه‌شده، اشتراک weak — دقیقاً همان استخوان‌بندی سیگنال‌های Django، سیستم‌های pub/sub و message brokerهاست. تفاوت سیستم اسباب‌بازی و سیستم production در سه سوال «بدبینانه» بود: شنونده خراب شود چه؟ شنونده بمیرد چه؟ وسط انتشار، لیست عوض شود چه؟ مهندس باتجربه را از روی سوال‌هایش می‌شناسند، نه جواب‌هایش.

**تمرین پیشنهادی:** (۱) صف رویدادهای شکست‌خورده (dead-letter) اضافه کنید تا خطاها بعداً قابل‌بازپخش باشند؛ (۲) `unsubscribe` صریح بنویسید؛ (۳) نسخه‌ی async از `EventBus` با `asyncio.gather` (فصل ۱۷) بسازید تا کانال‌ها همزمان تحویل بگیرند.

---

## پایان دوره

تبریک — رسیدید.

از فصل صفر شروع کردید، جایی که هنوز `return` و `print` جای بحث داشت. حالا در کارنامه‌تان یک ORM هست که با descriptor و registry کار می‌کند، یک فریمورک تست که تزریق fixture را با introspection انجام می‌دهد، و یک موتور اعلان که نشت حافظه‌ی Observer را با weakref می‌بندد. بین آن نقطه و این نقطه، نوزده فصل مفهوم و شش پروژه‌ی کامل است — و مهم‌تر از همه‌شان، یک تغییر ذهنیت.

چون مهم‌ترین درس کل دوره یک جمله بود که بارها تکرار شد: **هیچ راه‌حل مطلقاً درستی وجود ندارد.** ارث‌بری یا Composition؟ ABC یا Protocol؟ متاکلاس یا `__init_subclass__`؟ الگوی سنگین یا میان‌بُر پایتونی؟ پاسخ همیشه «بستگی دارد» بود — و حالا شما ابزار سنجیدن این «بستگی» را دارید: Tradeoffها را می‌شناسید، هزینه‌ها را می‌بینید، و می‌توانید از انتخاب‌هایتان در جلسه‌ی طراحی و مصاحبه دفاع کنید.

از اینجا سه مسیر طبیعی پیش رویتان است: فریمورک‌های وب (Django/FastAPI — که حالا سازوکار درونی‌شان برایتان آشناست)، معماری‌های بزرگ‌تر (Clean Architecture و DDD — که ادامه‌ی منطقی فصل‌های ۷ و ۸اند)، یا عمق بیشتر خود زبان (CPython و کتابخانه‌نویسی — که فصل‌های ۱۴ تا ۱۷ درش را باز کردند).

هر کدام را انتخاب کنید، یک چیز عوض نمی‌شود: شما دیگر «کاربر» پایتون نیستید. بروید و بسازید — و ابزار بعدی را که «جادویی» به نظر رسید، لبخند بزنید و بپرسید: «کدام پروتکل‌ها زیرش است؟»

و یک همراه آخر برای مسیر حرفه‌ای‌تان: [پیوست «۱۰ اشتباه مرگبار در کد شیءگرا»](appendix-10-deadly-mistakes.md) را دم دست نگه دارید — چک‌لیستی که در هر code review به‌کارتان می‌آید.
