# فصل چهاردهم: مدیریت حافظه و بهینه‌سازی

---

## مقدمه

تا اینجا روی طراحی و معماری تمرکز کردیم. اما یک جنبه‌ی حیاتیِ دیگر هست که در پروژه‌های واقعی — به‌خصوص در مقیاسِ بزرگ — تعیین‌کننده می‌شود: **مدیریت حافظه**.

در این فصل به زیرِ پوستِ پایتون می‌رویم: اشیاء در حافظه چطور نگه‌داری و آزاد می‌شوند، `__slots__` چطور مصرف حافظه را کم می‌کند، `weakref` چطور از نشتِ حافظه جلوگیری می‌کند، و چطور اشیاء را درست کپی و سریال کنیم. این مفاهیم، تفاوتِ کدی که در لپ‌تاپ کار می‌کند و کدی که در تولید با میلیون‌ها شیء دوام می‌آورد را می‌سازند.

---

## چرا مدیریت حافظه را یاد بگیریم؟

خیلی از برنامه‌نویسان کدی می‌نویسند که کار می‌کند، اما در مقیاسِ بزرگ به دیوار می‌خورد: مصرفِ حافظه منفجر می‌شود، اشیاء آزاد نمی‌شوند، و برنامه به‌مرور کند و پرمصرف می‌شود. یادگیری این مباحث کمک می‌کند:

- از مصرفِ بی‌رویه‌ی حافظه جلوگیری کنید
- کدی بنویسید که در Production دوام بیاورد
- نشتِ حافظه (Memory Leak) را شناسایی و رفع کنید
- بینِ خوانایی و کارایی، آگاهانه Tradeoff کنید

---

## مدیریت حافظه در پایتون

### اشیاء، Heap و ارجاع

در پایتون، هر شیء در بخشی از حافظه به نام **Heap** ساخته می‌شود. متغیرها خودِ شیء را نگه نمی‌دارند؛ فقط یک **ارجاع** (اشاره‌گر) به آن شیء روی Heap‌اند. این همان نکته‌ای است که در فصل دوم درباره‌ی `self` گفتیم.

```python
a = [1, 2, 3]      # a list is created on the Heap, a points to it
b = a              # b points to the same list — no copy is made!
b.append(4)
print(a)           # [1, 2, 3, 4] — because a and b are one single object
print(a is b)      # True
```

این «اشتراکِ ارجاع» منبعِ خیلی از باگ‌هاست: تغییرِ `b` روی `a` هم اثر گذاشت، چون هر دو به یک شیء اشاره می‌کنند. درکِ این مدل، برای مدیریتِ حافظه ضروری است.

### Mutable در برابر Immutable درون self

این تمایز، وقتی داده‌ای درونِ `self` نگه می‌دارید، اهمیتِ ویژه پیدا می‌کند. یک دامِ کلاسیک، آرگومانِ پیش‌فرضِ تغییرپذیر است:

```python
class ShoppingCart:
    def __init__(self, items=[]):      # trap! the default list becomes shared
        self.items = items

c1 = ShoppingCart()
c2 = ShoppingCart()
c1.items.append("Book")
print(c2.items)      # ['Book'] — disaster! c2 saw it too
```

چرا؟ چون لیستِ پیش‌فرض **یک‌بار** هنگام تعریفِ تابع ساخته می‌شود و بینِ همه‌ی فراخوانی‌ها مشترک می‌ماند. راهِ درست:

```python
class ShoppingCart:
    def __init__(self, items=None):
        self.items = items if items is not None else []   # a fresh list each time

c1 = ShoppingCart()
c2 = ShoppingCart()
c1.items.append("Book")
print(c2.items)      # [] — correct
```

قاعده‌ی طلایی: **هرگز شیءِ تغییرپذیر (list، dict، set) را به‌عنوان مقدارِ پیش‌فرضِ پارامتر نگذارید.** از `None` استفاده کنید و درون تابع بسازید.

### Reference Counting و Garbage Collector

پایتون دو مکانیزم برای آزادکردنِ حافظه دارد:

**۱. شمارشِ ارجاع (Reference Counting):** هر شیء یک شمارنده دارد که می‌گوید چند ارجاع به آن اشاره می‌کند. وقتی این شمارنده به صفر برسد، شیء **فوراً** آزاد می‌شود.

```python
import sys
a = [1, 2, 3]
print(sys.getrefcount(a))   # number of references (a bit more, because getrefcount itself creates one)
b = a
print(sys.getrefcount(a))   # increased by one
del b
print(sys.getrefcount(a))   # decreased again
```

**۲. Garbage Collector (GC):** شمارشِ ارجاع یک نقطه‌ضعف دارد: **ارجاعِ حلقوی** (Cyclic Reference). اگر دو شیء به هم اشاره کنند، شمارنده‌شان هرگز صفر نمی‌شود، حتی اگر هیچ‌کس دیگری به آن‌ها اشاره نکند:

```python
class Node:
    def __init__(self):
        self.partner = None

a = Node()
b = Node()
a.partner = b      # a points to b
b.partner = a      # b points to a — a cycle!
del a
del b
# neither counter reached zero, but nobody else can access them
```

اینجا GC وارد می‌شود: به‌صورتِ دوره‌ای، حلقه‌های ارجاعیِ غیرقابل‌دسترس را پیدا و آزاد می‌کند. تفاوت: شمارشِ ارجاع **فوری** است اما حلقه‌ها را نمی‌گیرد؛ GC **دوره‌ای** است و حلقه‌ها را می‌گیرد.

```python
import gc
gc.collect()       # manual GC call (rarely needed)
```

### __del__ و چرا تقریباً هیچ‌کس از آن استفاده نمی‌کند

`__del__` متدی است که هنگامِ آزادشدنِ شیء صدا زده می‌شود (به‌اصطلاح finalizer). اما استفاده‌اش خطرناک و نامطمئن است:

```python
class Resource:
    def __del__(self):
        print("released")     # when? unknown!

r = Resource()
del r                        # maybe here, maybe later
```

چرا از آن پرهیز می‌شود؟ **اول**، زمانِ دقیقِ اجرایش تضمین نیست (به‌خصوص با حلقه‌های ارجاعی یا هنگامِ خروجِ برنامه). **دوم**، اگر داخلش خطا رخ دهد، بی‌سروصدا نادیده گرفته می‌شود. **سوم**، می‌تواند حلقه‌های ارجاعی را برای GC پیچیده کند.

جایگزینِ درست برای آزادکردنِ منابع: **Context Manager** (`__enter__`/`__exit__` از فصل پنجم) که زمانِ آزادسازی را قطعی و صریح می‌کند.

### الگوی Flyweight

**Flyweight** یک الگوی طراحی برای صرفه‌جویی در حافظه است: به‌جای ساختنِ هزاران شیءِ تکراری، اشیاءِ مشترک را **بازاستفاده** می‌کنید. خودِ پایتون این کار را برای اعداد کوچک و رشته‌های کوتاه انجام می‌دهد:

```python
a = 256
b = 256
print(a is b)      # True — Python caches small integers (built-in Flyweight)

x = 1000
y = 1000
print(x is y)      # may be False — large integers are not cached
```

پیاده‌سازیِ دستیِ Flyweight برای اشیای سنگین:

```python
class Color:
    _cache = {}

    def __new__(cls, name):
        if name not in cls._cache:
            instance = super().__new__(cls)
            instance.name = name
            cls._cache[name] = instance      # reuse instead of building again
        return cls._cache[name]

red1 = Color("red")
red2 = Color("red")
print(red1 is red2)      # True — the same object, not two instances
```

اینجا `__new__` (که در فصل دوم دیدیم) به کار می‌آید: کنترلِ اینکه شیءِ تازه بسازیم یا موجود را برگردانیم.

---

## __slots__

### __slots__ چیست و چطور حافظه را کم می‌کند؟

به‌طور پیش‌فرض، هر شیء در پایتون attributeهایش را در یک دیکشنریِ داخلی به نام `__dict__` نگه می‌دارد. این دیکشنری انعطاف می‌دهد (می‌توانید هر attribute تازه‌ای اضافه کنید) اما **حافظه‌ی زیادی** می‌گیرد — به‌خصوص وقتی میلیون‌ها شیء دارید.

`__slots__` به پایتون می‌گوید attributeها ثابت‌اند، پس به‌جای دیکشنری، آن‌ها را در ساختارِ فشرده‌تری نگه می‌دارد:

```python
class PointNormal:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class PointSlots:
    __slots__ = ("x", "y")       # attributes are declared in advance
    def __init__(self, x, y):
        self.x = x
        self.y = y

import sys
p1 = PointNormal(1, 2)
p2 = PointSlots(1, 2)
print(hasattr(p1, "__dict__"))   # True — it has a dictionary
print(hasattr(p2, "__dict__"))   # False — it doesn't, more compact
```

در ساختِ میلیون‌ها شیء، `__slots__` می‌تواند مصرفِ حافظه را چشمگیر کاهش دهد (اغلب ۳۰ تا ۵۰ درصد).

### تأثیر بر __dict__ و محدودیت‌ها

با `__slots__`، شیء دیگر `__dict__` ندارد. نتیجه‌ی مهم: **نمی‌توانید attribute تازه‌ای که در slots اعلام نشده اضافه کنید:**

```python
p = PointSlots(1, 2)
# p.z = 3      # AttributeError — z is not in __slots__
```

### چه زمانی از __slots__ استفاده نکنیم؟

`__slots__` معایبی دارد:

- **انعطاف را از بین می‌برد:** نمی‌توانید attribute پویا اضافه کنید.
- **با ارث‌بریِ چندگانه دردسر دارد.**
- **با بعضی ابزارها** (که به `__dict__` تکیه می‌کنند) ناسازگار است.

پس فقط وقتی به کار ببرید که: تعدادِ اشیاء بسیار زیاد است، attributeها ثابت‌اند، و حافظه واقعاً گلوگاه است. برای کلاس‌های معمولی با تعدادِ کمِ نمونه، `__slots__` بهینه‌سازیِ زودرس است و فقط انعطاف را می‌گیرد. **Tradeoff:** حافظه در برابر انعطاف.

### __slots__ و property با هم

می‌توان `__slots__` و property را با هم داشت، اما با دقت: نامِ property نباید در `__slots__` باشد (وگرنه تصادم می‌شود). معمولاً یک slot با نامِ داخلی می‌گذارید و property رویش:

```python
class Temperature:
    __slots__ = ("_celsius",)        # slot with an internal name

    def __init__(self, c):
        self._celsius = c

    @property
    def celsius(self):
        return self._celsius
```

---

## weakref

### weakref چیست؟

اول بگویم چرا این بخش را جدی بگیرید. یکی از سمج‌ترین باگ‌هایی که دیده‌ام، سرویسی بود که هر چند روز یک‌بار حافظه‌اش پر می‌شد و ری‌استارت می‌خواست. مقصر؟ یک کشِ ساده — دیکشنری‌ای که اشیای کاربران را نگه می‌داشت و چون «ارجاعِ قوی» داشت، هیچ‌کدام از آن اشیاء هرگز آزاد نمی‌شدند؛ کش، قبرستانِ اشیای زنده‌به‌گور شده بود. راه‌حلش همان چیزی است که الان یاد می‌گیرید.

یک **ارجاعِ ضعیف** (weak reference) به شیئی اشاره می‌کند **بدون اینکه** شمارنده‌ی ارجاعش را بالا ببرد. یعنی اگر تنها ارجاع‌های باقی‌مانده به یک شیء، ضعیف باشند، شیء می‌تواند آزاد شود.

```python
import weakref

class Data:
    def __init__(self, value):
        self.value = value

d = Data(42)
weak = weakref.ref(d)          # weak reference
print(weak())                  # <__main__.Data ...> — still alive
del d                          # the only strong reference was removed
print(weak())                  # None — the object was freed
```

### کاربرد: Cache و Observer

`weakref` دو کاربردِ کلیدی دارد. **اول، کش:** می‌خواهید اشیاء را کش کنید، اما نمی‌خواهید کش خودش مانعِ آزادشدنشان شود.

```python
import weakref

class ImageCache:
    def __init__(self):
        self._cache = weakref.WeakValueDictionary()   # weak values

    def get(self, key):
        return self._cache.get(key)

    def put(self, key, image):
        self._cache[key] = image
```

با `WeakValueDictionary`، وقتی هیچ‌کس دیگری به یک تصویر اشاره نکند، خودکار از کش حذف می‌شود — بدونِ نشتِ حافظه.

**دوم، الگوی Observer:** یک ناظر (Observer) که به یک سوژه گوش می‌دهد، نباید مانعِ آزادشدنِ سوژه شود؛ ارجاعِ ضعیف این را حل می‌کند.

### weakref چطور از Memory Leak جلوگیری می‌کند؟

نشتِ حافظه اغلب وقتی رخ می‌دهد که یک ساختار (مثل کش یا فهرستِ ناظران) ارجاع‌های قوی نگه می‌دارد و مانعِ آزادشدنِ اشیائی می‌شود که دیگر لازم نیستند. با ارجاعِ ضعیف، آن اشیاء به‌محضِ بی‌استفاده‌شدن آزاد می‌شوند، حتی اگر هنوز در کش «فهرست» شده باشند.

### رابطه __slots__ و weakref

نکته‌ی ظریف: اگر از `__slots__` استفاده کنید، شیء به‌طور پیش‌فرض قابلِ weakref نیست (چون slotِ لازم را ندارد). برای اینکه هم `__slots__` و هم weakref داشته باشید، باید `'__weakref__'` را صراحتاً به slots اضافه کنید:

```python
class Node:
    __slots__ = ("value", "__weakref__")     # __weakref__ is required

    def __init__(self, value):
        self.value = value

n = Node(1)
import weakref
w = weakref.ref(n)      # now it works
print(w().value)        # 1
```

بدونِ `'__weakref__'` در slots، تلاش برای weakref خطا می‌دهد.

---

## کپی کردن اشیاء

### Shallow Copy در برابر Deep Copy

- **کپیِ سطحی (Shallow):** یک شیءِ جدید می‌سازد، اما attributeهای درونی‌اش هنوز به **همان** اشیای اصلی اشاره می‌کنند.
- **کپیِ عمیق (Deep):** همه‌چیز را بازگشتی کپی می‌کند؛ اشیای درونی هم نسخه‌ی مستقلِ خودشان را می‌گیرند.

```python
import copy

class Order:
    def __init__(self, items):
        self.items = items

original = Order(["Book", "pen"])

shallow = copy.copy(original)
shallow.items.append("notebook")
print(original.items)      # ['Book', 'pen', 'notebook'] — the list is shared!

original2 = Order(["Book", "pen"])
deep = copy.deepcopy(original2)
deep.items.append("notebook")
print(original2.items)     # ['Book', 'pen'] — independent, no effect
```

کپیِ سطحی، لیستِ `items` را کپی نکرد؛ فقط ارجاعش را کپی کرد. کپیِ عمیق، لیستِ مستقل ساخت.

### __copy__ و __deepcopy__

می‌توانید رفتارِ کپی را برای کلاسِ خودتان سفارشی کنید:

```python
import copy

class Connection:
    def __init__(self, host, cache):
        self.host = host
        self.cache = cache

    def __copy__(self):
        # custom shallow copy
        return Connection(self.host, self.cache)

    def __deepcopy__(self, memo):
        # custom deep copy
        return Connection(self.host, copy.deepcopy(self.cache, memo))
```

چرا لازم است؟ گاهی بعضی attributeها را نباید کپی کرد (مثلاً یک اتصالِ شبکه‌ی زنده) یا کپی نیاز به منطقِ خاص دارد. `memo` در `__deepcopy__` برای جلوگیری از کپیِ بی‌نهایتِ ارجاع‌های حلقوی است.

### مزیت اشیای Immutable در کپی

اشیای تغییرناپذیر (مثل `tuple`، `frozenset`، یا اشیای شما که عمداً بی‌تغییرند) مزیتِ بزرگی دارند: **نیازی به کپی ندارند.** چون تغییر نمی‌کنند، اشتراکِ ارجاع کاملاً امن است. این یکی از دلایلِ محبوبیتِ طراحیِ Immutable است — هم ساده‌تر، هم امن‌تر در برنامه‌های چندنخی.

---

## سریال‌سازی (مرور از منظرِ حافظه)

سریال‌سازی را در فصل یازدهم کامل دیدیم. اینجا فقط از منظرِ حافظه و حالتِ شیء مرورش می‌کنیم، چون این دو ابزار دقیقاً با همان «حالتِ درونِ حافظه» کار می‌کنند:

- **`json`**: به فرمتِ متنیِ استاندارد؛ امن و بین‌زبانی، اما فقط انواعِ ساده.
- **`pickle`**: به بایتِ باینریِ مخصوصِ پایتون؛ هر شیئی را ذخیره می‌کند اما **ناامن** است.

```python
import json

class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def to_dict(self):
        return {"name": self.name, "age": self.age}

u = User("ali", 30)
text = json.dumps(u.to_dict(), ensure_ascii=False)
print(text)      # {"name": "ali", "age": 30}
```

متدهای `__getstate__` و `__setstate__` (فصل یازدهم) دقیقاً همین «حالتِ روی Heap» را بسته‌بندی و بازسازی می‌کنند — حالا که سازوکارِ حافظه را می‌شناسید، معنیِ عمیق‌ترِ آن متدها هم روشن‌تر است: آن‌ها مرزِ بینِ شیءِ زنده در حافظه و بایت‌های مرده روی دیسک‌اند.

---

## خلاصه

```
مدیریت حافظه در پایتون
════════════════════════════════════════════════
  Reference Counting  ──► فوری، اما حلقه‌ها را نمی‌گیرد
         +
  Garbage Collector   ──► دوره‌ای، حلقه‌های ارجاعی را می‌گیرد

  ابزارهای بهینه‌سازی:
    __slots__  ──► حافظه‌ی کمتر (در ازای انعطاف)
    weakref    ──► ارجاعِ بدون‌مالکیت (ضدِ نشت)
    Flyweight  ──► بازاستفاده به‌جای ساختِ تکراری
```

| مفهوم | نکته‌ی کلیدی |
|-------|--------------|
| Heap و ارجاع | متغیرها به شیء اشاره می‌کنند، نه کپی |
| پیش‌فرضِ تغییرپذیر | هرگز `def f(x=[])`؛ از `None` استفاده کن |
| Ref Counting + GC | صفرشدنِ شمارنده / گرفتنِ حلقه‌ها |
| `__del__` | نامطمئن؛ به‌جایش Context Manager |
| `__slots__` | حافظه‌ی کمتر، انعطافِ کمتر |
| `weakref` | ارجاعِ ضعیف؛ کش و Observer بدونِ نشت |
| Shallow/Deep copy | اشتراکِ ارجاع در برابر کپیِ مستقل |
| Immutable | نیازی به کپی ندارد، امن‌تر |

**نکته‌ی طلایی:** بیشترِ این ابزارها (`__slots__`، `weakref`، Flyweight) بهینه‌سازی‌اند — و بهینه‌سازیِ زودرس، ریشه‌ی خیلی از بدی‌هاست. اول کدِ درست و خوانا بنویسید؛ فقط وقتی اندازه‌گیری نشان داد حافظه واقعاً گلوگاه است، سراغِ این ابزارها بروید. اما دام‌های حافظه (مثلِ پیش‌فرضِ تغییرپذیر) را از همان اول بشناسید، چون آن‌ها باگ‌اند، نه بهینه‌سازی.

---

## پرسش‌های مصاحبه

**۱. «مدیریت حافظه‌ی CPython را توضیح بده.»** — دو سازوکارِ مکمل: Reference Counting (هر شیء شمارنده‌ی ارجاع دارد؛ صفر شد، همان لحظه آزاد) + Garbage Collectorِ نسلی برای **چرخه‌ها** — جایی که شمارنده هرگز صفر نمی‌شود (a→b→a). اگر گفتید GC فقط برای چرخه‌هاست نه همه‌چیز، فهمِ درست را نشان داده‌اید.

**۲. «__slots__ چه می‌دهد و چه می‌گیرد؟»** — می‌دهد: حذفِ `__dict__` هر نمونه → حافظه‌ی به‌مراتب کمتر و دسترسیِ کمی سریع‌تر؛ برای میلیون‌ها شیءِ کوچک تعیین‌کننده. می‌گیرد: افزودنِ attributeِ پویا، سازگاریِ راحت با وراثتِ چندگانه، و به‌طورِ پیش‌فرض `__weakref__`. قاعده: ابزارِ بهینه‌سازیِ اندازه‌گیری‌شده است، نه پیش‌فرضِ هر کلاس.

**۳. «weakref کجا به دادت رسیده؟»** — کش‌ها و Observerها: جایی که می‌خواهید «اشاره داشته باشید بی‌آنکه زنده نگه دارید». مثالِ کلاسیک: registryای که اشیای مرده را با ارجاعِ قوی برای همیشه حبس می‌کند — همان «برنامه بعد از یک هفته سنگین شد». `WeakValueDictionary`/`WeakMethod` راه‌حل‌اند.

**۴. «فرق copy و deepcopy؟ کِی هرکدام؟»** — shallow فقط پوسته‌ی بیرونی را می‌سازد و اندرونی‌ها مشترک می‌مانند (تغییرِ تو در توی کپی = تغییرِ اصل)؛ deep همه‌ی گراف را بازمی‌سازد و گران است. جوابِ ممتاز: «اول می‌پرسم اصلاً چرا کپی؟ طراحی با اشیای تغییرناپذیر خیلی از کپی‌ها را حذف می‌کند.»

---

**در فصل بعد:** حالا که می‌دانیم اشیاء در حافظه چطور زندگی می‌کنند، به زیرِ کاپوتِ خودِ attributeها می‌رویم: **Descriptorها** — سازوکاری که `@property`، `classmethod` و فیلدهای ORMها روی آن ساخته شده‌اند.
