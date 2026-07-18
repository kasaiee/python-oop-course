# فصل هشتم: اصول SOLID

---

## مقدمه

تا اینجا ابزارها (کلاس، ارث‌بری، Composition) و اصول پراکنده‌ای (قانون دیمتر، Tell Don't Ask) را دیدیم. اما یک سوال باقی است: **از کجا بفهمیم طراحی‌مان خوب است یا بد؟**

**SOLID** پاسخ می‌دهد. این پنج اصل، که رابرت سی. مارتین (معروف به Uncle Bob) آن‌ها را گردآوری و رواج داد، مجموعه‌ای از راهنماها برای ساختن طراحی تمیز، منعطف و قابل‌نگهداری‌اند. در صنعت نرم‌افزار، SOLID تقریباً استاندارد طلایی طراحی شیءگرا شناخته می‌شود.

هر حرف SOLID یک اصل است: **S**ingle Responsibility، **O**pen/Closed، **L**iskov Substitution، **I**nterface Segregation، **D**ependency Inversion. یکی‌یکی، با مثال‌های واقعی (نه حیوان و خودرو)، سراغشان می‌رویم.

---

## چرا باید SOLID را یاد بگیریم؟

کدی که کار می‌کند، لزوماً کد خوبی نیست. سوال واقعی این است: وقتی شش ماه بعد یک ویژگی جدید اضافه می‌شود، آیا کل سیستم به‌هم می‌ریزد؟ SOLID کمک می‌کند:

- کدی بنویسید که در برابر تغییر مقاوم باشد
- از وابستگی‌های پنهان که تغییر را خطرناک می‌کنند پرهیز کنید
- کد را طوری بسازید که دیگران بتوانند بی‌ترس گسترشش دهند
- طراحی خود را به سطح حرفه‌ای برسانید

اما یک هشدار از همین ابتدا: SOLID **قانون نیست، راهنماست**. گاهی رعایت افراطی یک اصل، اصل دیگری را زیر پا می‌گذارد یا کد را بی‌جهت پیچیده می‌کند. در پایان فصل به این Tradeoffها برمی‌گردیم.

---

## اصل اول — Single Responsibility (تک‌مسئولیتی)

### تعریف

**هر کلاس باید فقط یک دلیل برای تغییر داشته باشد.** یعنی یک مسئولیت منسجم. اگر یک کلاس چند کار نامرتبط انجام دهد، هر تغییر در هرکدام، خطر شکستن بقیه را دارد.

بهترین توصیفی که از SRP شنیده‌ام، از یک همکار قدیمی بود. در code review به نویسنده‌ی یک کلاس شلوغ گفت: «برای این کلاس یک جمله بگو که کارش را توضیح بدهد.» جوابش این بود: «کاربر را می‌سازد **و** ایمیلش را می‌فرستد **و** لاگ می‌کند و...». همکارم گفت: «هر جا مجبور شدی بگویی "و"، همان‌جا مرز کلاس بعدی است.» تست ساده‌ای است؛ از امروز روی کلاس‌های خودتان اجرایش کنید.

### چرا User نباید send_email داشته باشد؟

```python
# violating SRP: this class has two responsibilities
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def save(self):
        # responsibility 1: save to the database
        print(f"{self.name} saved to database")

    def send_email(self, message):
        # responsibility 2: send email — unrelated to user identity
        print(f"Email to {self.email}: {message}")
```

`User` دو دلیل مستقل برای تغییر دارد: تغییر نحوه‌ی ذخیره‌سازی (دیتابیس عوض شود)، و تغییر نحوه‌ی ارسال ایمیل (سرویس ایمیل عوض شود). این دو هیچ ربطی به هم ندارند و نباید در یک کلاس باشند.

```python
# following SRP: each responsibility, its own class
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserRepository:
    def save(self, user):
        print(f"{user.name} saved to database")

class EmailService:
    def send(self, user, message):
        print(f"Email to {user.email}: {message}")
```

حالا `User` فقط هویت کاربر است. ذخیره‌سازی و ایمیل، مسئولیت‌های جدا با کلاس‌های جدا. تغییر سرویس ایمیل، دیگر `User` را لمس نمی‌کند.

### چطور تشخیص دهیم یک کلاس بیش از یک مسئولیت دارد؟

نشانه‌ها: نام کلاس شامل «and» می‌شود («UserAndEmailManager»)؛ متدهایش به گروه‌های نامرتبط تقسیم می‌شوند؛ یا وقتی توصیفش می‌کنید مجبورید بگویید «این کلاس X را انجام می‌دهد **و همچنین** Y را». آن «و همچنین»، بوی نقض SRP می‌دهد.

---

## اصل دوم — Open/Closed (باز-بسته)

### تعریف

**کلاس‌ها باید برای توسعه باز، و برای تغییر بسته باشند.** یعنی باید بتوانید رفتار جدید **اضافه** کنید بی‌آنکه کد موجود را **ویرایش** کنید.

### مثال: افزودن نوع جدید پرداخت

```python
# violating OCP: every new method means editing this function
class PaymentProcessor:
    def process(self, method, amount):
        if method == "card":
            print(f"Card: {amount}")
        elif method == "wallet":
            print(f"Wallet: {amount}")
        # new method → add an elif → edit tested code (dangerous)
```

هر بار که روش پرداخت تازه‌ای می‌آید، باید این کلاس آزموده‌شده و پایدار را دستکاری کنید — و ریسک شکستن روش‌های موجود را بپذیرید.

```python
# following OCP: with abstraction and polymorphism
class PaymentMethod:
    def pay(self, amount): ...

class CardPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Card: {amount}")

class WalletPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Wallet: {amount}")

class PaymentProcessor:
    def process(self, method: PaymentMethod, amount):
        method.pay(amount)              # it doesn't know and has no need to know which method

# adding a new method: just a new class, without touching existing code
class CryptoPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Crypto: {amount}")

processor = PaymentProcessor()
processor.process(CardPayment(), 1000)
processor.process(CryptoPayment(), 2000)    # the old code stayed untouched
```

`PaymentProcessor` حالا «بسته برای تغییر» است (هرگز ویرایشش نمی‌کنید) اما «باز برای توسعه» (هر تعداد روش جدید می‌افزایید). این همان قدرت چندریختی فصل چهارم، در قالب یک اصل طراحی است.

---

## اصل سوم — Liskov Substitution (جانشینی لیسکوف)

### تعریف

**هر جا از یک کلاس والد استفاده می‌شود، باید بتوان زیرکلاسش را جایگزین کرد، بی‌آنکه رفتار درست برنامه بشکند.** به‌بیان ساده: زیرکلاس باید «قرارداد» والد را محترم بشمارد.

### مثال نقض

مثال کلاسیک، مستطیل و مربع است، اما یک نمونه‌ی نرم‌افزاری‌ترش را ببینیم:

```python
class FileStorage:
    def save(self, data):
        print(f"Saved: {data}")

    def delete(self, key):
        print(f"Deleted: {key}")

# a subclass that breaks the contract
class ReadOnlyStorage(FileStorage):
    def save(self, data):
        raise NotImplementedError("this storage is read-only!")   # violating LSP!
```

مشکل: هر کدی که با `FileStorage` کار می‌کند و انتظار دارد `save` کار کند، با `ReadOnlyStorage` به‌شکل غیرمنتظره می‌شکند. `ReadOnlyStorage` نمی‌تواند بی‌دردسر جایگزین `FileStorage` شود، پس LSP را نقض می‌کند.

### چطور بفهمیم زیرکلاس LSP را نقض می‌کند؟

نشانه‌ها: زیرکلاس یک متد والد را با `NotImplementedError` یا `pass` خالی «خنثی» می‌کند؛ زیرکلاس شرط‌های ورودی سخت‌گیرانه‌تری می‌گذارد؛ یا خروجی ضعیف‌تری برمی‌گرداند. اگر مجبورید قبل از استفاده از زیرکلاس، نوعش را چک کنید (`if isinstance(x, ReadOnly)`)، احتمالاً LSP نقض شده.

### راه‌حل

سلسله‌مراتب را طوری بازطراحی کنید که قرارداد رعایت شود. مثلاً به‌جای اینکه `ReadOnlyStorage` از `FileStorage` ارث ببرد، هر دو از یک پایه‌ی محدودتر ارث ببرند:

```python
class Readable:
    def read(self, key): ...

class Writable(Readable):
    def save(self, data): ...
    def delete(self, key): ...

class ReadOnlyStorage(Readable):     # only what it actually can
    def read(self, key):
        return "..."

class FullStorage(Writable):
    def read(self, key): return "..."
    def save(self, data): ...
    def delete(self, key): ...
```

حالا هیچ زیرکلاسی مجبور نیست متدی را که نمی‌تواند پیاده کند، به ارث ببرد. این ما را مستقیم به اصل بعدی می‌رساند.

---

## اصل چهارم — Interface Segregation (جداسازی اینترفیس)

### تعریف

**هیچ کلاسی نباید مجبور به وابستگی به متدهایی باشد که استفاده نمی‌کند.** اینترفیس‌های «چاق» را به اینترفیس‌های کوچک و متمرکز بشکنید.

### مثال

```python
# violating ISP: a fat interface
class Worker:
    def work(self): ...
    def eat(self): ...
    def sleep(self): ...

# a robot is forced to implement eat and sleep too — which is meaningless
class RobotWorker(Worker):
    def work(self): print("works")
    def eat(self): raise NotImplementedError    # a robot does not eat!
    def sleep(self): raise NotImplementedError  # a robot does not sleep!
```

`RobotWorker` مجبور شد به متدهایی وابسته شود که برایش بی‌معنا هستند. این هم ISP را نقض می‌کند و هم (چون `NotImplementedError` می‌اندازد) LSP را.

```python
# following ISP: small, independent interfaces
class Workable:
    def work(self): ...

class Eatable:
    def eat(self): ...

class Sleepable:
    def sleep(self): ...

class HumanWorker(Workable, Eatable, Sleepable):
    def work(self): print("works")
    def eat(self): print("eats")
    def sleep(self): print("sleeps")

class RobotWorker(Workable):        # only what it actually needs
    def work(self): print("works")
```

هر کلاس فقط اینترفیس‌هایی را می‌گیرد که واقعاً به آن‌ها نیاز دارد. `RobotWorker` دیگر مجبور نیست وانمود کند غذا می‌خورد.

### چطور در پایتون اینترفیس‌های کوچک بسازیم؟

در پایتون، اینترفیس‌ها را با ABC یا Protocol می‌سازیم (فصل بعد کامل می‌بینیم). قاعده: چند اینترفیس کوچک و متمرکز، بهتر از یک اینترفیس بزرگ و همه‌کاره است.

---

## اصل پنجم — Dependency Inversion (وارونگی وابستگی)

### تعریف

**ماژول‌های سطح‌بالا نباید به ماژول‌های سطح‌پایین وابسته باشند؛ هر دو باید به انتزاع وابسته باشند.** یعنی به‌جای وابستگی به یک پیاده‌سازی مشخص، به یک رابط انتزاعی وابسته شوید.

### مثال: وابستگی به جزئیات در برابر انتزاع

```python
# violating DIP: direct dependency on a specific implementation
class MySQLDatabase:
    def save(self, data):
        print(f"Saved in MySQL: {data}")

class UserService:
    def __init__(self):
        self.db = MySQLDatabase()       # nailed to a specific class
    def register(self, user):
        self.db.save(user)
```

مشکل: `UserService` به `MySQLDatabase` **میخکوب** شده. اگر بخواهید به PostgreSQL کوچ کنید، یا در تست از یک دیتابیس حافظه‌ای استفاده کنید، باید `UserService` را دستکاری کنید.

```python
# following DIP: depend on abstraction, not details
class Database:                          # abstraction (interface)
    def save(self, data): ...

class MySQLDatabase(Database):
    def save(self, data):
        print(f"Saved in MySQL: {data}")

class InMemoryDatabase(Database):        # suitable for testing
    def __init__(self):
        self.store = []
    def save(self, data):
        self.store.append(data)

class UserService:
    def __init__(self, db: Database):    # depends on the abstraction, not the implementation
        self.db = db                     # the implementation is "injected" from outside
    def register(self, user):
        self.db.save(user)

# in production:
service = UserService(MySQLDatabase())
# in tests:
test_service = UserService(InMemoryDatabase())
```

حالا `UserService` نمی‌داند و اهمیتی نمی‌دهد که کدام دیتابیس زیرش است؛ فقط به رابط `Database` وابسته است. پیاده‌سازی واقعی از بیرون **تزریق** می‌شود (Dependency Injection). این همان چیزی است که در فصل قبل، تست‌پذیری را ممکن کرد.

نکته: در پایتون، به‌لطف Duck Typing، این «انتزاع» حتی می‌تواند صرفاً یک قرارداد ضمنی باشد (هر شیئی که `save` دارد). ابزارهای رسمی‌ترش — ABC و Protocol — موضوع فصل بعدند.

### چیزهای دیگری شبیه SOLID

SOLID تنها مجموعه‌ی اصول نیست. چند اصل مکمل مهم که در همین دوره دیدیم یا می‌بینیم:

- **Composition over Inheritance** (فصل هفتم): ترکیب را بر ارث‌بری ترجیح بده.
- **Law of Demeter** (فصل هفتم): با غریبه‌ها حرف نزن.
- **DRY** (Don't Repeat Yourself): خودت را تکرار نکن.
- **YAGNI** (You Aren't Gonna Need It): پیش از نیاز، پیچیدگی نساز.
- **KISS** (Keep It Simple): ساده نگه دار.

جالب اینکه بعضی از این‌ها با هم در تنش‌اند — که ما را به Tradeoffها می‌رساند.

---

## Tradeoff: SOLID را کورکورانه دنبال نکنید

هر پنج اصل ارزشمندند، اما رعایت افراطی‌شان خطر دارد:

- **SRP افراطی** می‌تواند به انفجار کلاس‌های ریز کم‌محتوا منجر شود که دنبال‌کردنشان سخت است.
- **OCP افراطی** ممکن است لایه‌های انتزاعی زیادی بسازد که برای مسئله‌ای که هرگز تغییر نمی‌کند، بی‌جهت پیچیده‌اند (نقض YAGNI).
- **DIP افراطی** برای هر چیز کوچکی یک اینترفیس می‌سازد و کد را پر از غیرمستقیم‌سازی می‌کند.

قاعده‌ی خردمندانه: SOLID را وقتی به کار ببرید که مسئله‌ی واقعی‌ای (تغییرپذیری، تکرار، شکنندگی) حل می‌کند، نه چون «باید». یک کلاس ساده که هرگز تغییر نمی‌کند، نیازی به پنج لایه‌ی انتزاع ندارد. مثل همیشه، **قضاوت متناسب با موقعیت**، بالاتر از پیروی کورکورانه است.

---

## خلاصه

```
اصول SOLID
════════════════════════════════════════════════════
  S  Single Responsibility  یک کلاس، یک دلیل تغییر
  O  Open/Closed            باز برای توسعه، بسته برای تغییر
  L  Liskov Substitution    زیرکلاس، جانشین سالم والد
  I  Interface Segregation  اینترفیس کوچک، نه چاق
  D  Dependency Inversion   وابستگی به انتزاع، نه جزئیات
```

| اصل | نشانه‌ی نقض | راه‌حل |
|-----|-------------|--------|
| SRP | کلاسی که «X و همچنین Y» می‌کند | جداکردن مسئولیت‌ها |
| OCP | افزودن ویژگی = ویرایش `if/elif` | انتزاع + چندریختی |
| LSP | زیرکلاس با `NotImplementedError` | بازطراحی سلسله‌مراتب |
| ISP | کلاسی وابسته به متدهای بی‌استفاده | شکستن اینترفیس چاق |
| DIP | وابستگی میخکوب به پیاده‌سازی | تزریق وابستگی از بیرون |

**نکته‌ی طلایی:** SOLID یک قطب‌نماست، نه یک قفس. این اصول به شما جهت می‌دهند، اما مقصد را موقعیت تعیین می‌کند. برنامه‌نویس باتجربه می‌داند کی اصلی را کامل رعایت کند و کی به‌نفع سادگی از آن بگذرد. توانایی سنجیدن این Tradeoff، خودش نشانه‌ی بلوغ مهندسی است.

---

## پرسش‌های مصاحبه

SOLID را در هر مصاحبه‌ی طراحی می‌پرسند؛ تفاوت نمره در مثال‌هاست، نه تعریف‌ها.

**۱. «پنج اصل را با مثال نقض بگو، نه با تعریف.»** — SRP: کلاس User که ایمیل هم می‌فرستد. OCP: تابع پرداخت که برای هر روش جدید یک elif می‌گیرد. LSP: زیرکلاسی که متد والد را با «راه نمی‌دهم» بازنویسی می‌کند. ISP: اینترفیس چاقی که ربات را مجبور به پیاده‌سازی eat می‌کند. DIP: سرویسی که مستقیم `MySQLDatabase()` می‌سازد. اگر برای هرکدام «بو»یش را بگویید، از ۹۰٪ کاندیداها جلوترید.

**۲. «LSP در عمل یعنی چه؟ چطور نقضش را می‌فهمی؟»** — هر جا والد انتظار می‌رود، زیرکلاس باید بی‌غافلگیری جایگزین شود: نه شرط ورودی را سخت‌تر کند، نه تضمین خروجی را ضعیف‌تر، نه استثنای غیرمنتظره بدهد. تست عملی: اگر جایی مجبور شدید بنویسید `if isinstance(x, SubClass): # این یکی فرق دارد`، LSP همان‌جا مرده است.

**۳. «Dependency Injection چه ربطی به DIP دارد؟»** — DIP اصل است (به انتزاع وابسته باش، نه به جزئیات)؛ DI تکنیک رایج اجرای آن (وابستگی را از بیرون بده — معمولاً در سازنده). ثمره‌اش را هم بگویید: تعویض‌پذیری در production و تست‌پذیری با Fake — بدون patch.

**۴. «آیا می‌شود SOLID را زیادی رعایت کرد؟»** — بله، و اسمش over-engineering است: انتزاع برای چیزی که هرگز دومین پیاده‌سازی‌اش نیامد، شکستن کلاس‌ها تا حد گم‌شدن منطق. اصول برای مهار «هزینه‌ی تغییر»اند؛ وقتی تغییری در افق نیست، سادگی خودش یک اصل است (YAGNI). گفتن همین تعادل، بلوغ طراحی را نشان می‌دهد.

---

**در فصل بعد:** SOLID مدام از «انتزاع» و «اینترفیس» حرف زد. حالا سراغ ابزارهای رسمی انتزاع در پایتون می‌رویم: **کلاس‌های انتزاعی (ABC)، پروتکل‌ها (Protocol) و ژنریک‌ها (Generics)**.
