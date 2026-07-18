# فصل دوم: مفاهیم بنیادین شی‌گرایی

## مقدمه

در فصل قبل با فلسفه و تاریخچه‌ی شیءگرایی آشنا شدیم — فهمیدیم این پارادایم از کجا آمد و چه مشکلی را حل می‌کند. حالا بالاخره وقت نوشتن **اولین کد شیءگرا**ست.

این فصل، پایه‌ی همه‌چیز است. هر مفهومی که در فصل‌های بعد می‌آید — تفکر شیءگرا، ارث‌بری، Polymorphism، متاکلاس — روی چهار مفهوم همین فصل ساخته می‌شود: **کلاس، شیء، `self` و سازنده**. اگر این چهار چیز را خوب بفهمید، بقیه‌ی دوره روان پیش می‌رود. اگر سطحی رد شوید، بعدها به باگ‌های گیج‌کننده برمی‌خورید که ریشه‌شان همین‌جاست.

چون این نخستین رویارویی شما با سینتکس شیءگرایی است، از صفر و با حوصله پیش می‌رویم: اول کالبدشکافی یک کلاس ساده، بعد مفاهیم عمیق‌تر `self` و سازنده. به‌خصوص بخش `self` را با دقت بخوانید، چون بیشترین سوءتفاهم‌ها آنجاست.

## کالبدشکافی مفاهیم، یکی‌یکی

### کلاس و شیء: قالب و نمونه

**کلاس** یک قالب است؛ نقشه‌ای که می‌گوید موجودیت‌هایی از این نوع چه داده‌ای دارند و چه کارهایی می‌کنند. کلاس، خودش یک موجودیت زنده در حافظه نیست — مثل نقشه‌ی یک خانه که خودش خانه نیست.

**شیء** (که به آن نمونه یا instance هم گفته می‌شود) یک ساخت واقعی از آن قالب در حافظه است. از روی یک نقشه می‌توان صدها خانه ساخت؛ از روی یک کلاس می‌توان هزاران شیء ساخت که هرکدام داده‌ی خودش را دارد.

```python
class User:                       # class: template
    def __init__(self, username):
        self.username = username

u1 = User("ali")                  # first object (an instance of User)
u2 = User("sara")                 # second object, independent of the first

print(u1.username)   # ali
print(u2.username)   # sara
print(type(u1))      # <class '__main__.User'>
print(u1 is u2)      # False — two separate objects in memory
```

`u1` و `u2` هر دو از کلاس `User` ساخته شده‌اند، اما دو موجود کاملاً مستقل‌اند؛ تغییر `u1` هیچ اثری بر `u2` ندارد.

### کالبدشکافی یک کلاس: از تعریف تا استفاده

چون این نخستین کلاسی است که می‌نویسید، بیایید نحو (syntax) آن را تکه‌تکه بشکافیم. یک کلاس با کلیدواژه‌ی `class`، یک نام، و یک دونقطه شروع می‌شود؛ بدنه‌اش با **تورفتگی** (indentation) مشخص می‌شود:

```python
class Message:                     # the class keyword + name + colon
    def __init__(self, text):      # constructor: runs when the object is created
        self.text = text           # sets an attribute on the object itself

    def preview(self):             # a method (behavior)
        return self.text[:10]      # has access to the object's own data
```

چند نکته‌ی نحوی که همین حالا باید بدانید:

- نام کلاس‌ها طبق قرارداد با حرف بزرگ شروع می‌شود (`Message`)، برخلاف متغیرها و توابع که با حرف کوچک نوشته می‌شوند.
- هر خطی که یک سطح **تورفته‌تر** از `class` باشد، «عضو» آن کلاس است (اینجا `__init__` و `preview`).
- `def` درون یک کلاس یک **متد** می‌سازد، و اولین پارامترش تقریباً همیشه `self` است (چرایی‌اش را در بخش بعد می‌بینیم).

حالا از این قالب یک شیء بسازیم و از آن استفاده کنیم:

```python
m = Message("Hello, this is a test message")   # creating the object: class name + parentheses

print(m.text)          # Hello, this is a test message   ← reading an attribute with a dot
print(m.preview())     # Hello, this    ← calling a method with a dot and parentheses
```

به این سه گام دقت کنید، چون **کل کدنویسی شیءگرا** روی همین الگو بنا شده:

1. **تعریف:** کلاس را با `class` می‌سازید (قالب).
2. **ساخت:** با `ClassName(...)` یک شیء از روی قالب می‌سازید. پرانتز باعث می‌شود پایتون خودکار `__init__` را صدا بزند و شیء را مقداردهی کند.
3. **استفاده:** با نقطه (`.`) به attributeها (`m.text`) و متدها (`m.preview()`) دسترسی می‌گیرید — تفاوت ظاهری‌شان فقط پرانتز متد است.

اگر این الگوی «تعریف → ساخت با `()` → دسترسی با `.`» را در ذهن داشته باشید، بقیه‌ی فصل فقط عمیق‌تر کردن همین سه گام است.

### self دقیقاً چیست؟

اینجا مهم‌ترین بخش فصل است. بیایید سوءتفاهم رایج را از بین ببریم.

وقتی متدی روی یک شیء صدا زده می‌شود، پایتون باید بداند آن متد روی **کدام** شیء کار می‌کند. `self` دقیقاً همان است: **ارجاعی به شیئی که متد روی آن فراخوانی شده.**

`self` یک کلمه‌ی کلیدی جادویی نیست؛ فقط یک قرارداد نام‌گذاری برای **اولین پارامتر** هر متد است. پایتون به‌طور خودکار شیء را به‌عنوان این پارامتر پاس می‌دهد.

```python
class User:
    def greet(self):
        return f"Hello, I am {self.username}"

    def __init__(self, username):
        self.username = username

u = User("ali")
print(u.greet())          # Hello, I am ali
# behind the scenes it is equivalent to:
print(User.greet(u))      # Hello, I am ali — u itself was passed as self
```

این دو خط آخر دقیقاً یک کار می‌کنند. `u.greet()` فقط یک میان‌بُر است برای `User.greet(u)`. حالا معلوم می‌شود چرا `self` اولین پارامتر است: چون همان شیء `u` است که پایتون خودکار جای `self` می‌گذارد.

**یک مدل ذهنی دقیق‌تر:** در پایتون، متغیرها «برچسب»‌اند نه «جعبه». وقتی `u = User("ali")` می‌نویسید، شیء در بخشی از حافظه به نام Heap ساخته می‌شود و `u` فقط یک ارجاع (اشاره‌گر) به آن است. `self` هم دقیقاً همان ارجاع است — به همان شیء روی Heap اشاره می‌کند. برای همین وقتی داخل متد `self.username` را تغییر می‌دهید، تغییر روی همان شیء واقعی اعمال می‌شود، نه روی یک کپی.

```python
def rename(self, new_name):
    self.username = new_name    # the same object on the Heap is changed

u.rename("reza")
print(u.username)   # reza — the change persists, because self was not a copy
```

### Attribute چیست؟

**Attribute** (ویژگی/فیلد) داده‌ای است که به یک شیء یا کلاس چسبیده. در مثال‌های بالا، `username` یک attribute است. با نقطه به آن دسترسی می‌گیریم: `u.username`.

پایتون بسیار پویاست؛ حتی می‌توانید بعد از ساخت شیء، attribute جدید به آن اضافه کنید (هرچند معمولاً بهتر است همه را در سازنده تعریف کنید):

```python
u = User("ali")
u.email = "ali@example.com"    # new attribute, only on this object
print(u.email)                 # ali@example.com
```

### Class Variable و Instance Variable

در شی‌گرایی دو نوع متغیر داریم:

- **Instance Variable:** به هر شیء جداگانه تعلق دارد. معمولاً درون `__init__` با `self.x = ...` تعریف می‌شود. هر شیء نسخه‌ی خودش را دارد.
- **Class Variable:** به خود کلاس تعلق دارد و **بین همه‌ی اشیاء مشترک** است. مستقیماً در بدنه‌ی کلاس (بیرون از متدها) تعریف می‌شود.

```python
class Employee:
    company = "Acme"          # Class Variable — shared among all

    def __init__(self, name):
        self.name = name      # Instance Variable — specific to each object

e1 = Employee("ali")
e2 = Employee("sara")

print(e1.company, e2.company)   # Acme Acme — both see the same shared value
print(e1.name, e2.name)         # ali sara — each its own

Employee.company = "Globex"     # a change on the class affects all
print(e1.company, e2.company)   # Globex Globex
```

قاعده: **Class Variable برای داده‌ای که واقعاً باید مشترک باشد** (مثل نام شرکت، یک شمارنده‌ی کل، تنظیمات پیش‌فرض). برای داده‌ی مختص هر شیء، همیشه از instance variable و `self` در `__init__` استفاده کنید.

یک دام رایج: Class Variableهای تغییرپذیر (مثل لیست) که سهواً مشترک می‌شوند.

```python
class Team:
    members = []             # trap! this list is shared among all
    def add(self, name):
        self.members.append(name)

t1 = Team()
t2 = Team()
t1.add("ali")
print(t2.members)   # ['ali'] — leak! t2 saw it too

# correct:
class Team:
    def __init__(self):
        self.members = []    # each team has its own list
```

### متد در برابر تابع

**متد** تابعی است که درون یک کلاس تعریف شده و به شیء گره خورده. تفاوت اصلی با تابع معمولی: متد به‌طور خودکار `self` (شیء صاحب) را دریافت می‌کند و می‌تواند به حالت شیء دسترسی داشته باشد.

```python
def area_func(width, height):        # standalone function
    return width * height

class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):                  # method: has access to the object's own data
        return self.width * self.height

print(area_func(3, 4))               # 12 — data must be passed in
r = Rectangle(3, 4)
print(r.area())                      # 12 — data comes from the object itself
```

### سازنده: متد __init__

**سازنده** (Constructor) متدی است که هنگام ساخت شیء، به‌طور خودکار اجرا می‌شود تا آن را **مقداردهی اولیه** کند. در پایتون این متد `__init__` نام دارد (دو زیرخط در ابتدا و انتها — به این‌ها «dunder» می‌گویند).

```python
class Product:
    def __init__(self, name, price, stock=0):
        self.name = name
        self.price = price
        self.stock = stock
        print(f"product {name} created")

p = Product("Keyboard", 500_000)   # __init__ is called automatically
# Output: product Keyboard created
print(p.stock)                    # 0 — default value
```

نکته‌ی دقیق: `__init__` شیء را **نمی‌سازد**؛ شیء پیش از فراخوانی `__init__` ساخته شده و کار `__init__` فقط **پرکردن** آن با داده‌ی اولیه است. اولین پارامترش هم `self` است — یعنی همان شیء تازه‌ساخته‌شده که قرار است مقداردهی شود.

### __new__ در برابر __init__

اگر `__init__` شیء را نمی‌سازد، پس چه چیزی می‌سازد؟ پاسخ: `__new__`.

چرخه‌ی حیات ساخت یک شیء دو مرحله دارد:

1. `__new__` فراخوانی می‌شود و **شیء خام را می‌سازد** و برمی‌گرداند.
2. `__init__` روی آن شیء اجرا می‌شود و **مقداردهی‌اش می‌کند**.

```python
class Demo:
    def __new__(cls, *args, **kwargs):
        print("1) __new__: raw object is created")
        instance = super().__new__(cls)
        return instance

    def __init__(self, value):
        print("2) __init__: object is initialized")
        self.value = value

d = Demo(42)
# Output:
# 1) __new__: raw object is created
# 2) __init__: object is initialized
```

تفاوت کلیدی: `__new__` پارامتر `cls` (خود کلاس) می‌گیرد و باید شیء را **برگرداند**؛ `__init__` پارامتر `self` (شیء ساخته‌شده) می‌گیرد و چیزی برنمی‌گرداند.

در ۹۹٪ موارد فقط `__init__` را می‌نویسید و `__new__` را دست نمی‌زنید. اما در چند حالت خاص `__new__` لازم می‌شود: ساختن الگوی Singleton، ارث‌بری از انواع تغییرناپذیر مثل `int` و `tuple` (که مقدارشان باید در زمان ساخت قطعی شود)، و بعضی کارهای پیشرفته‌ی متاکلاسی (فصل شانزدهم).

```python
# real example: why we must touch __new__ to inherit from tuple
class Point(tuple):
    def __new__(cls, x, y):
        return super().__new__(cls, (x, y))   # the tuple value is finalized here

p = Point(3, 4)
print(p)        # (3, 4)
print(p[0])     # 3
```

### اشیای بی‌حالت و باحالت

- **باحالت (Stateful):** شیئی که داده‌ای در خود نگه می‌دارد و در طول زمان تغییر می‌کند. مثل حساب بانکی که موجودی‌اش کم و زیاد می‌شود.
- **بی‌حالت (Stateless):** شیئی که حالت داخلی متغیری ندارد و رفتارش فقط به ورودی‌ها بستگی دارد. مثل یک اعتبارسنج که فقط ورودی را بررسی می‌کند.

```python
# stateful — the result depends on the history of calls
class Accumulator:
    def __init__(self):
        self.total = 0
    def add(self, x):
        self.total += x
        return self.total

# stateless — the same input always the same output
class PriceFormatter:
    def format(self, amount):
        return f"{amount:,} Toman"
```

اشیای بی‌حالت مزیت مهمی دارند: چون حالت مشترک تغییرپذیر ندارند، امن‌تر (به‌خصوص در برنامه‌های چندنخی) و ساده‌تر برای تست‌اند. **Tradeoff:** اما همه‌چیز را نمی‌توان بی‌حالت کرد؛ خیلی از موجودیت‌های دنیای واقعی ذاتاً حالت دارند. انتخاب بین این دو، یکی از تصمیم‌های همیشگی طراحی است.

## مثال کامل: طراحی یک BankAccount کاربردی

بیایید همه‌ی مفاهیم فصل را در یک کلاس واقعی جمع کنیم. این کلاس را قدم‌به‌قدم می‌سازیم.

نسخه‌ی اول، ساده:

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner          # instance variable
        self.balance = balance      # instance variable (state)

    def deposit(self, amount):
        self.balance += amount

    def withdraw(self, amount):
        self.balance -= amount
```

مشکل: هیچ اعتبارسنجی‌ای نیست. می‌شود مبلغ منفی واریز کرد یا بیش از موجودی برداشت. نسخه‌ی دوم را مقاوم‌تر می‌کنیم:

```python
class BankAccount:
    bank_name = "Sample Bank"        # Class Variable — shared among all accounts
    _next_id = 1000                 # Class Variable — shared counter for the ID

    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance
        self.account_id = BankAccount._next_id   # read from the shared counter
        BankAccount._next_id += 1                # and increment it

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("deposit amount must be positive")
        self.balance += amount
        return self.balance

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("withdrawal amount must be positive")
        if amount > self.balance:
            raise ValueError("insufficient balance")
        self.balance -= amount
        return self.balance

a1 = BankAccount("ali", 1000)
a2 = BankAccount("sara")

print(a1.account_id, a2.account_id)   # 1000 1001 — unique IDs
a1.deposit(500)
print(a1.balance)                     # 1500
print(a1.bank_name, a2.bank_name)     # Sample Bank Sample Bank — shared
```

چند نکته‌ی طراحی که همه‌ی فصل را جمع می‌کنند: `owner` و `balance` **instance variable**‌اند چون مختص هر حساب‌اند. `bank_name` و `_next_id` **class variable**‌اند چون واقعاً باید مشترک باشند — همه‌ی حساب‌ها در یک بانک‌اند و شمارنده‌ی شناسه باید یکتا و سراسری باشد. `deposit` و `withdraw` **رفتار**‌اند که **حالت** (`balance`) را با اعتبارسنجی تغییر می‌دهند. و `BankAccount` یک شیء **باحالت** است، چون موجودی‌اش در طول زمان عوض می‌شود.

## خلاصه

| مفهوم | تعریف کوتاه |
|-------|-------------|
| کلاس | قالب/نقشه برای ساختن اشیاء |
| شیء (instance) | نمونه‌ی واقعی کلاس در حافظه |
| `self` | ارجاع به همان شیئی که متد رویش صدا زده شده |
| attribute | داده‌ی چسبیده به شیء یا کلاس |
| Instance Variable | مختص هر شیء (`self.x` در `__init__`) |
| Class Variable | مشترک بین همه‌ی اشیاء (در بدنه‌ی کلاس) |
| متد | تابعی که به شیء گره خورده و `self` می‌گیرد |
| `__init__` | سازنده؛ شیء ساخته‌شده را مقداردهی می‌کند |
| `__new__` | شیء خام را می‌سازد (به‌ندرت لازم) |
| باحالت / بی‌حالت | حالت‌دار متغیر / بدون حالت داخلی |

```
چرخه‌ی حیات ساخت یک شیء
═══════════════════════════════════
  Product("کیبورد", 500)
        │
        ▼
  __new__(cls)  ──►  شیء خام روی Heap ساخته می‌شود
        │
        ▼
  __init__(self, ...) ──►  شیء با داده پر می‌شود
        │
        ▼
  ارجاع به شیء برگردانده می‌شود ──► p
```

**نکته‌ی طلایی:** هر بار که به `self` نگاه می‌کنید، به‌خاطر بسپارید که این فقط «همان شیء» است — نه چیزی مرموز. و هر بار که متغیری در کلاس تعریف می‌کنید، از خودتان بپرسید: «این باید مختص هر شیء باشد یا مشترک بین همه؟» پاسخ این سوال، تعیین می‌کند instance variable بنویسید یا class variable.

## پرسش‌های مصاحبه

این فصل پرسوال‌ترین فصل مصاحبه‌های جونیور است؛ این چهار را بی‌تپق جواب بدهید.

**۱. «self دقیقاً چیست؟»** — ارجاع به همان شیئی که متد رویش صدا زده شده؛ نه کلمه‌ی کلیدی است نه جادو — فقط پارامتر اول متد که پایتون خودکار پُرش می‌کند: `u.greet()` همان `User.greet(u)` است. گفتن همین هم‌ارزی، قلب جواب کامل است.

**۲. «فرق class variable و instance variable؟ یک دام رایجش را بگویید.»** — اولی روی کلاس و مشترک بین همه؛ دومی روی هر شیء و مختص خودش. دام: class variable تغییرپذیر (لیست مشترکی که همه‌ی اشیاء سهواً در آن می‌ریزند) و دام ظریف‌تر: `self.count += 1` که به‌جای تغییر متغیر کلاس، یک instance variable سایه‌انداز می‌سازد.

**۳. «__init__ سازنده است؟»** — سوال تله‌دار. جواب دقیق: نه — شیء را `__new__` می‌سازد؛ `__init__` شیء ساخته‌شده را مقداردهی می‌کند. در ۹۹٪ موارد فقط با `__init__` کار دارید، اما تمایزش در Singleton و ارث‌بری از انواع تغییرناپذیر (`tuple`، `int`) عملی می‌شود.

**۴. «چرا `def f(x, items=[])` خطرناک است؟»** — مقدار پیش‌فرض فقط **یک‌بار** هنگام تعریف تابع ساخته می‌شود و بین همه‌ی فراخوانی‌ها مشترک می‌ماند؛ لیست کم‌کم پُر می‌شود. راه درست: `items=None` و ساخت لیست داخل تابع. (همین دام در فصل دهم به `field(default_factory=list)` می‌رسد.)

**در فصل بعد:** حالا که بلدیم کلاس و شیء بسازیم، یک قدم عقب می‌رویم و می‌پرسیم: *کی* و *چرا* باید کلاس بسازیم؟ در فصل «تفکر شیءگرا» یاد می‌گیریم چطور یک مسئله‌ی واقعی را به کلاس‌ها و مسئولیت‌ها ترجمه کنیم — یعنی مثل یک طراح فکر کنیم.
