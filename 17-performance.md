# فصل هفدهم: Performance و بهینه‌سازی پیشرفته

---

## مقدمه

به آخرین فصلِ مفهومیِ دوره رسیدیم. تا اینجا یاد گرفتیم کدِ **درست، تمیز و قابل‌نگهداری** بنویسیم. اما یک بُعدِ دیگر هم هست: **سرعت**.

این فصل با فصل چهاردهم (مدیریت حافظه) تفاوت دارد. آنجا تمرکز بر مصرفِ حافظه بود؛ اینجا بر **سرعتِ اجرا و مقیاس‌پذیری**. یاد می‌گیریم چه چیزی کدِ شیءگرا را کند می‌کند، چطور با کش و بارگذاریِ تنبل سریع‌ترش کنیم، چطور با چندنخی و چندپردازشی کار کنیم، و مهم‌تر از همه — چطور **اندازه‌گیری** کنیم تا کورکورانه بهینه نکنیم.

یک هشدارِ مقدماتی که تمِ کلِ فصل است: **بهینه‌سازیِ زودرس، ریشه‌ی بسیاری از بدی‌هاست** (جمله‌ی معروفِ دانلد کنوت). اول درست بنویسید، بعد اندازه بگیرید، و فقط گلوگاه‌های واقعی را بهینه کنید.

---

## چرا بهینه‌سازی Performance را یاد بگیریم؟

کدِ شیءگرای خوش‌طراحی ممکن است در تولید، با حجمِ بالای داده، کند شود. یادگیریِ این تکنیک‌ها کمک می‌کند:

- کدِ شیءگرا را برای سرعتِ بالاتر بهینه کنید
- از کش و بارگذاریِ تنبل استفاده کنید
- بینِ خوانایی و سرعت، آگاهانه Tradeoff کنید
- گلوگاه‌های عملکردی را شناسایی و رفع کنید

---

## چه چیزی کد شیءگرا را کند می‌کند؟

شیءگرایی یک هزینه‌ی پنهان دارد. دو عاملِ اصلی:

**۱. جست‌وجوی attribute (Attribute Lookup):** هر بار که `obj.x` می‌نویسید، پایتون باید `x` را جست‌وجو کند — اول در `__dict__`ِ شیء، بعد در کلاس، بعد در والدها (طبقِ MRO). این زنجیره، در حلقه‌های داغ (میلیون‌ها تکرار) هزینه دارد.

**۲. سربارِ فراخوانیِ متد (Method Call Overhead):** هر فراخوانیِ متد، از فراخوانیِ یک تابعِ ساده گران‌تر است، چون `self` باید پاس داده شود و متد در MRO یافت شود.

```python
import time

class Point:
    def __init__(self, x):
        self.x = x
    def get_x(self):
        return self.x

p = Point(5)

# repeated attribute lookup in a hot loop
start = time.perf_counter()
total = 0
for _ in range(10_000_000):
    total += p.x          # an attribute lookup each time
print(f"attribute: {time.perf_counter() - start:.3f}s")
```

نکته: این هزینه‌ها معمولاً ناچیزند و فقط در کدِ واقعاً داغ اهمیت می‌یابند. **پیش از بهینه‌سازی، مطمئن شوید این‌جا واقعاً گلوگاه است.**

---

## تکنیک‌های بهینه‌سازی

### __slots__ برای سرعت

در فصل چهاردهم دیدیم `__slots__` حافظه را کم می‌کند. اما مزیتِ دومی هم دارد: **دسترسیِ سریع‌ترِ attribute**. چون attributeها در ساختارِ ثابت (نه دیکشنری) نگه‌داری می‌شوند، جست‌وجویشان کمی سریع‌تر است:

```python
class WithSlots:
    __slots__ = ("x", "y")
    def __init__(self, x, y):
        self.x = x
        self.y = y

class WithoutSlots:
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

در ساختِ میلیون‌ها شیء و دسترسیِ مکرر، `WithSlots` هم حافظه‌ی کمتر و هم سرعتِ کمی بیشتر دارد. **Tradeoff:** همان محدودیت‌های فصل چهاردهم (بی‌انعطافی) پابرجاست.

### Lazy Loading با @property

**بارگذاریِ تنبل** یعنی محاسبه‌ی یک مقدارِ گران را تا لحظه‌ی واقعیِ نیاز به تعویق بیندازید. اگر هرگز لازم نشد، هرگز محاسبه نمی‌شود:

```python
from functools import cached_property

class Report:
    def __init__(self, data):
        self.data = data

    @cached_property
    def summary(self):
        print("Heavy computation...")       # only on the first access
        return sum(self.data)             # and only once

r = Report(list(range(1000)))
# so far summary has not been computed
print(r.summary)      # Heavy computation... \n 499500
print(r.summary)      # 499500 — from cache, without recomputing
```

`cached_property` (فصل پنجم) هم بارگذاریِ تنبل است و هم کش. مقدار را تا لحظه‌ی نیاز به تعویق می‌اندازد و بعد نگهش می‌دارد.

### Caching با functools.lru_cache

برای متدها و توابعی که با ورودیِ یکسان، همیشه خروجیِ یکسان می‌دهند (توابعِ خالص)، `lru_cache` نتیجه را کش می‌کند:

```python
from functools import lru_cache

class MathService:
    @lru_cache(maxsize=128)
    def fibonacci(self, n):
        if n < 2:
            return n
        return self.fibonacci(n - 1) + self.fibonacci(n - 2)

m = MathService()
print(m.fibonacci(30))     # fast — intermediate results are cached
```

بدونِ کش، `fibonacci(30)` بازگشتیِ نمایی و بسیار کند است؛ با کش، هر مقدار فقط یک‌بار محاسبه می‌شود. **Tradeoff:** کش حافظه مصرف می‌کند (`maxsize` را کنترل کنید) و فقط برای توابعِ خالص امن است — اگر خروجی به حالتِ متغیر بستگی داشته باشد، کش نتایجِ غلط می‌دهد.

### Descriptor برای دسترسی بهینه

Descriptorها (فصل پنجم و پانزدهم) می‌توانند منطقِ دسترسیِ بهینه و مشترک را در یک‌جا متمرکز کنند و از تکرار جلوگیری کنند — به‌خصوص وقتی همان اعتبارسنجی یا تبدیل روی چند فیلد لازم است.

### __dict__ در برابر __slots__: مقایسه

| | `__dict__` (پیش‌فرض) | `__slots__` |
|---|----------------------|-------------|
| حافظه | بیشتر | کمتر |
| سرعتِ دسترسی | کمی کندتر | کمی سریع‌تر |
| انعطاف | attribute پویا مجاز | فقط attributeهای اعلام‌شده |
| مناسبِ | کلاس‌های معمولی | میلیون‌ها شیءِ ثابت |

قاعده: `__slots__` را فقط وقتی به‌کار ببرید که پروفایلینگ نشان دهد حافظه یا سرعتِ دسترسی واقعاً گلوگاه است.

---

## چندنخی و چندپردازشی

### GIL و اثرش

پایتون یک قفلِ سراسری به نامِ **GIL** (Global Interpreter Lock) دارد که اجازه می‌دهد در هر لحظه فقط **یک نخ** کدِ پایتون اجرا کند. نتیجه‌ی مهم:

- برای کارهای **پردازشی (CPU-bound)** — محاسباتِ سنگین — چندنخی (threading) سرعتی نمی‌آورد، چون GIL نخ‌ها را نوبتی می‌کند. اینجا باید **چندپردازشی (multiprocessing)** استفاده کنید که پردازش‌های جدا (بدونِ GILِ مشترک) می‌سازد.
- برای کارهای **ورودی/خروجی (I/O-bound)** — انتظار برای شبکه یا دیسک — چندنخی مفید است، چون هنگامِ انتظار، GIL آزاد می‌شود و نخِ دیگر کار می‌کند.

```python
# CPU-bound → multiprocessing
from multiprocessing import Pool

def heavy(n):
    return sum(i * i for i in range(n))

if __name__ == "__main__":
    with Pool(4) as pool:
        results = pool.map(heavy, [10**6, 10**6, 10**6, 10**6])
    # four truly parallel processes, without the GIL limitation
```

**قاعده:** CPU-bound → multiprocessing؛ I/O-boundِ کم‌تعداد → threading؛ I/O-boundِ انبوه → asyncio (بخشِ بعد).

### کلاس‌های thread-safe

یک کلاس **thread-safe** است اگر چند نخ بتوانند همزمان با آن کار کنند بدونِ خرابیِ داده. مشکلِ اصلی، **شرایطِ مسابقه** (Race Condition) است: وقتی دو نخ همزمان یک حالتِ مشترک را تغییر می‌دهند.

```python
import threading

# not thread-safe — race condition
class UnsafeCounter:
    def __init__(self):
        self.count = 0
    def increment(self):
        self.count += 1        # this operation is not atomic! two threads can interfere

# thread-safe — with a lock
class SafeCounter:
    def __init__(self):
        self.count = 0
        self._lock = threading.Lock()
    def increment(self):
        with self._lock:       # only one thread at a time
            self.count += 1
```

`SafeCounter` با یک قفل (`Lock`) تضمین می‌کند که تغییرِ `count` هر بار توسطِ فقط یک نخ انجام شود. **Tradeoff:** قفل، سرباری اضافه می‌کند و اگر بد استفاده شود می‌تواند به بن‌بست (Deadlock) منجر شود. برای همین، طراحیِ **بی‌حالت** یا **تغییرناپذیر** (فصل‌های سوم و چهاردهم) که اصلاً حالتِ مشترکِ متغیر ندارد، اغلب راهِ ساده‌تر و امن‌تری برای thread-safety است.

### شیءگرایی در دنیای async

برای I/O-bound یک راهِ سومِ مدرن هم هست که آگهی‌های استخدام پُرش کرده‌اند: **asyncio**. به‌جای چند نخ، یک نخ داریم و یک حلقه‌ی رویداد که هزاران کارِ «منتظر» را جابه‌جا می‌کند — هر جا `await` ببیند، می‌داند می‌تواند سراغِ کارِ بعدی برود. FastAPI و aiohttp روی همین سوارند.

خبرِ خوب برای شما: دنیای async همان پروتکل‌های شیءگرایی است که بلدید، فقط با پیشوندِ `a`. کلاس‌ها `async def` متد می‌گیرند، context manager می‌شود `__aenter__`/`__aexit__` و iterator می‌شود `__aiter__`/`__anext__`:

```python
import asyncio

class AsyncDatabase:
    async def __aenter__(self):                  # async version of __enter__
        await asyncio.sleep(0.01)                # pretend: opening a connection
        print("connection opened")
        return self

    async def __aexit__(self, exc_type, exc, tb):
        await asyncio.sleep(0.01)
        print("connection closed")               # guaranteed, like before

    async def fetch_users(self, count):          # an async generator
        for i in range(1, count + 1):
            await asyncio.sleep(0.01)            # pretend: one row arrives
            yield f"user-{i}"

async def main():
    async with AsyncDatabase() as db:            # __aenter__ / __aexit__
        async for user in db.fetch_users(3):     # __aiter__ / __anext__
            print(user)

asyncio.run(main())
# connection opened
# user-1
# user-2
# user-3
# connection closed
```

اگر فصل پنجم را خوب خوانده باشید، اینجا هیچ‌چیزِ واقعاً جدیدی نیست: همان قراردادِ «متدهای جادویی، رفتارِ زبان را تعریف می‌کنند»، این‌بار با نسخه‌های awaitable. این یکی از پاداش‌های یادگیریِ عمیقِ پروتکل‌هاست — فریمورکِ بعدی که بیاید هم، باز همین قاعده‌ها را بازی می‌کند.

دو هشدارِ مهمِ طراحی در کلاس‌های async: اول، **`__init__` نمی‌تواند `async` باشد** — اگر ساختِ شیء به I/O نیاز دارد (مثل بازکردنِ اتصال)، از یک factoryِ کلاسی استفاده کنید (`await Database.connect(...)` — همان الگوی `classmethod` فصل پنجم، در لباسِ async). دوم، قفل‌های `threading` را در async به‌کار نبرید؛ `asyncio.Lock` معادلِ سازگارش است. و قاعده‌ی نهاییِ انتخاب حالا کامل می‌شود: **CPU-bound → multiprocessing؛ I/O-boundِ کم‌تعداد → threading؛ I/O-boundِ انبوه (صدها اتصالِ همزمان) → asyncio.**

---

## اندازه‌گیری: Profiling

مهم‌ترین بخشِ کلِ فصل: **قبل از بهینه‌سازی، اندازه بگیرید.** حدس‌زدنِ گلوگاه تقریباً همیشه اشتباه است.

### ابزارهای profiling

پایتون ابزارهای داخلی دارد:

```python
import cProfile

def slow_function():
    total = 0
    for i in range(1_000_000):
        total += i ** 2
    return total

cProfile.run("slow_function()")
# Output: number of calls and time spent in each function
```

`cProfile` نشان می‌دهد کدام تابع چقدر وقت می‌گیرد. ابزارهای دیگر: `timeit` (برای زمان‌سنجیِ قطعاتِ کوچک)، `line_profiler` (خط‌به‌خط)، و `memory_profiler` (مصرفِ حافظه).

```python
import timeit

# comparing two approaches concretely
t1 = timeit.timeit("[i*2 for i in range(100)]", number=10000)
t2 = timeit.timeit("list(map(lambda i: i*2, range(100)))", number=10000)
print(f"list comprehension: {t1:.3f}")
print(f"map: {t2:.3f}")
```

با اندازه‌گیریِ واقعی تصمیم بگیرید، نه با حدس. اغلب اوقات، گلوگاهِ واقعی جایی است که انتظارش را نداشتید.

---

## ضدالگوهای عملکردی

چند ضدالگوی رایج که هم طراحی و هم عملکرد را خراب می‌کنند:

**God Object (فصل سوم):** کلاسِ همه‌کاره که علاوه بر بدیِ طراحی، بهینه‌سازی و کش‌کردنش هم سخت است، چون همه‌چیز در آن گره خورده.

**Lava Flow:** کدِ مرده یا قدیمی که کسی جرئتِ حذفش را ندارد، پس در پروژه می‌ماند، حافظه و زمانِ بارگذاری می‌گیرد، و خواندنش را سخت می‌کند.

**بهینه‌سازیِ زودرس:** پیچیده‌کردنِ کد برای سرعتی که نیازش نیست. کدِ پیچیده‌ی «بهینه» که ۰.۱٪ زمانِ اجرا را می‌گیرد، فقط بدهیِ فنی است.

**N+1 Query:** در کدی که با دیتابیس کار می‌کند، به‌جای یک کوئریِ دسته‌ای، درونِ حلقه برای هر شیء یک کوئریِ جدا زدن — که در مقیاس فاجعه است.

---

## خلاصه

```
مسیرِ درستِ بهینه‌سازی
════════════════════════════════════════════════
  ۱. کدِ درست و خوانا بنویس
        │
        ▼
  ۲. اندازه بگیر (cProfile, timeit)
        │
        ▼
  ۳. گلوگاهِ واقعی را پیدا کن
        │
        ▼
  ۴. فقط همان‌جا را بهینه کن
        │
        ▼
  ۵. دوباره اندازه بگیر (آیا بهتر شد؟)
```

| تکنیک | کاربرد |
|-------|--------|
| `__slots__` | حافظه‌ی کمتر + دسترسیِ سریع‌تر |
| `cached_property` | بارگذاریِ تنبل + کش |
| `lru_cache` | کشِ توابعِ خالص |
| multiprocessing | کارهای CPU-bound (دور زدنِ GIL) |
| threading / asyncio | کارهای I/O-bound |
| `Lock` | thread-safety (یا بهتر: طراحیِ بی‌حالت) |
| `cProfile` / `timeit` | اندازه‌گیریِ گلوگاه |

**نکته‌ی طلایی:** این فصل یک پیامِ اصلی دارد که مهم‌تر از همه‌ی تکنیک‌هاست: **اول اندازه بگیر، بعد بهینه کن.** کدِ خوانا و درست که کمی کند است، تقریباً همیشه بهتر از کدِ سریعِ نامفهوم است — چون اکثرِ کد در مسیرِ داغ نیست و بهینه‌سازی‌اش هیچ سودی ندارد جز پیچیدگی. Tradeoffِ خوانایی در برابر سرعت را فقط جایی بپذیرید که اندازه‌گیری ثابت کرده ارزشش را دارد.

---

## پرسش‌های مصاحبه

**۱. «GIL چیست و چه چیزی را واقعاً محدود می‌کند؟»** — قفلِ سراسریِ مفسر: در هر لحظه یک نخ بایت‌کد اجرا می‌کند. فقط **CPU-bound چندنخی** را عقیم می‌کند؛ I/O-bound سود می‌برد چون GIL هنگامِ انتظار آزاد می‌شود. نقشه‌ی تصمیم: CPU → multiprocessing؛ I/O کم → threading؛ I/O انبوه → asyncio.

**۲. «برنامه کند است — از کجا شروع می‌کنی؟»** — جوابِ غلط: «بهینه‌سازی می‌کنم». جوابِ درست: «اندازه می‌گیرم» — cProfile برای پیداکردنِ گلوگاه، timeit برای مقایسه‌ی راه‌حل‌ها؛ بعد فقط همان گلوگاه. حدسِ انسان درباره‌ی گلوگاه تقریباً همیشه غلط است و بهینه‌سازیِ زودرس، خواناییِ کد را بی‌دلیل قربانی می‌کند.

**۳. «race condition چیست و چطور جلویش را می‌گیری؟»** — دو نخ روی حالتِ مشترکِ تغییرپذیر، با عملیاتِ غیراتمی (`count += 1` = خواندن+جمع+نوشتن). راه‌ها به ترتیبِ ترجیح: حذفِ حالتِ مشترک (بی‌حالتی/تغییرناپذیری)، ساختارهای صف‌محور، و در نهایت Lock — با هزینه‌ی سربار و خطرِ deadlock.

**۴. «async چه فرقی با نخ دارد و کلاس‌هایت چطور async می‌شوند؟»** — یک نخ + حلقه‌ی رویداد؛ جابه‌جایی فقط سرِ `await`های داوطلبانه، پس هزاران کارِ همزمانِ I/O بدونِ هزینه‌ی نخ‌ها. کلاس‌ها با همان پروتکل‌های آشنا در نسخه‌ی a-دار: `__aenter__`/`__aexit__`، `__aiter__`/`__anext__` — و چون `__init__` نمی‌تواند async باشد، ساختِ نیازمندِ I/O می‌رود در factoryِ کلاسی (`await Db.connect()`).

---

**در فصل بعد:** آخرین قطعه‌ی پازل قبل از پروژه‌ها: **سازمان‌دهیِ کد در ماژول‌ها و پکیج‌ها** — چطور کلاس‌هایی را که ساخته‌ایم در ساختارِ یک پروژه‌ی واقعی بچینیم.
