## فصل هفتم: ترکیب (Composition)، معماری و طراحی پیشرفته

### مقدمه

در دو فصل پیش، با ارث‌بری و کاربردهای پیشرفته‌ی آن آشنا شدیم. اما لازم است به یک حقیقت مهم اعتراف کنیم: ارث‌بری، در بسیاری از موارد، بهترین انتخاب برای طراحی نیست. در این فصل با رویکردی متفاوت و در عین حال بنیادین آشنا می‌شویم: **ترکیب (Composition)**. این رویکرد در بسیاری از سناریوهای واقعی، جایگزینی انعطاف‌پذیرتر و مقاوم‌تر در برابر تغییرات برای ارث‌بری محسوب می‌شود.

هدف این فصل، فراتر رفتن از بحث صرفِ «چگونگی» و ورود به «چرایی» و «چه زمانی» است. خواهیم آموخت که چرا اصل مشهور «ترکیب را بر ارث‌بری ترجیح بده» (Favor Composition over Inheritance) در دنیای طراحی نرم‌افزار تا این اندازه جدی گرفته می‌شود. همچنین با مجموعه‌ای از قوانین و اصول طراحی کلیدی آشنا خواهیم شد که به تنهایی، هر کدام می‌توانند کیفیت کد شما را به طرز چشمگیری ارتقا دهند: **قانون دیمتر**، اصل **Tell, Don't Ask**، و مفاهیم **معماری لایه‌ای**.

### ضرورت درک ترکیب و طراحی پیشرفته

ارث‌بری، رابطه‌ای از نوع **«هست» (Is-A)** ایجاد می‌کند؛ یعنی «کلاس B یک نوع از کلاس A است». اما در دنیای واقعی، بسیاری از روابط بین اشیاء از نوع **«دارد» (Has-A)** هستند: یک ماشین یک موتور دارد، یک سفارش یک روش پرداخت دارد، یک کاربر یک آدرس دارد. تحمیل رابطه‌ی «Is-A» بر روابطی که ذاتاً «Has-A» هستند، ساختاری شکننده و غیرطبیعی پدید می‌آورد که در برابر تغییرات به شدت آسیب‌پذیر است.

درک درست و استفاده از ترکیب به شما کمک می‌کند تا:

- کدی با **وابستگی کمتر (Loose Coupling)** بنویسید که تغییر در یک بخش، تأثیر کمی بر بخش‌های دیگر دارد.
- از **شکنندگی و پیچیدگی** سلسله‌مراتب‌های عمیق ارث‌بری بگریزید.
- کدی **تست‌پذیرتر** بسازید، زیرا می‌توانید هر جزء را به‌راحتی با نمونه‌های جایگزین (مانند Mock) تست کنید.
- معماری‌هایی **مقیاس‌پذیر و قابل‌نگهداری** طراحی کنید که با رشد پروژه، پیچیدگی آن‌ها به صورت خطی افزایش یابد، نه نمایی.

### ترکیب (Composition) چیست؟

ترکیب به زبان ساده یعنی **یک کلاس، نمونه‌هایی از کلاس‌های دیگر را در خود نگه دارد و بخشی از کار خود را با کمک آن‌ها انجام دهد**؛ درست برعکس ارث‌بری که در آن کلاس، خودِ والد می‌شود. رابطه‌ای که ترکیب می‌سازد، از نوع «داشتن» (Has-A) است.

بیایید با یک مثال ملموس، تفاوت را درک کنیم. فرض کنید می‌خواهیم کلاسی برای نمایش یک ماشین طراحی کنیم.

**طراحی اشتباه با ارث‌بری:**
```python
# wrong: a car is not a "type of" engine!
class Engine:
    def start(self):
        return "started"

class Car(Engine):  # Is-A is wrong here
    pass
```

در این طراحی، گفته می‌شود که «ماشین، یک نوع موتور است» که از نظر منطقی کاملاً نادرست است. این رویکرد، وابستگی‌های معنایی غلطی ایجاد می‌کند.

**طراحی درست با ترکیب:**
```python
# correct: a car "has" an engine
class Engine:
    def start(self):
        return "engine started"

class Car:
    def __init__(self):
        self.engine = Engine()  # Car has an Engine (Has-A)

    def start(self):
        return self.engine.start()  # delegates the work to the engine

c = Car()
print(c.start())  # engine started
```

در اینجا، کلاس `Car` یک موتور را در خود نگه داشته و کار راه‌اندازی را به آن واگذار (delegate) می‌کند. این طراحی، طبیعی‌تر، خواناتر و به مراتب منعطف‌تر است.

### چرا ترکیب اغلب بر ارث‌بری ترجیح دارد؟

برای درک عمیق‌تر این برتری، یک مثال واقعی‌تر را بررسی می‌کنیم. فرض کنید سیستمی برای مدیریت کارمندان طراحی می‌کنیم که کارمندان انواع مختلفی دارند (تمام‌وقت، پاره‌وقت) و هر کدام نیز روش دریافت حقوق متفاوتی دارند (از طریق بانک، نقدی).

**رویکرد ارث‌بری:**
در این رویکرد، مجبور خواهیم بود برای هر ترکیب ممکن از «نوع استخدام» و «روش دریافت حقوق»، یک کلاس مجزا تعریف کنیم:

```python
class Employee:
    pass

class FullTimeEmployee(Employee):
    pass

class PartTimeEmployee(Employee):
    pass

# Now if we want to add a second dimension — how salary is received
# we would have to build every combination:
class FullTimeBankEmployee(FullTimeEmployee):
    pass

class FullTimeCashEmployee(FullTimeEmployee):
    pass

class PartTimeBankEmployee(PartTimeEmployee):
    pass

class PartTimeCashEmployee(PartTimeEmployee):
    pass
# ... combinatorial explosion!
```
همان‌طور که می‌بینید، با افزودن هر بُعد جدید (مثلاً روش محاسبه‌ی حقوق)، تعداد کلاس‌ها به صورت **ضربدری** افزایش می‌یابد. این پدیده که **انفجار کلاس** (Class Explosion) نام دارد، یکی از بزرگترین دام‌های ارث‌بری نابجا است و به سرعت کد را به مجموعه‌ای غیرقابل مدیریت تبدیل می‌کند.

**رویکرد ترکیب (Composition):**
در این رویکرد، «روش دریافت حقوق» را به عنوان یک قابلیت مستقل (یک کلاس مجزا) طراحی می‌کنیم و سپس آن را به کارمند **تزریق** می‌کنیم:

```python
class PaymentMethod:
    def pay(self, amount):
        pass

class BankPayment(PaymentMethod):
    def pay(self, amount):
        return f"{amount} deposited to bank account"

class CashPayment(PaymentMethod):
    def pay(self, amount):
        return f"{amount} paid in cash"

class Employee:
    def __init__(self, name, payment_method):
        self.name = name
        self.payment_method = payment_method  # we "inject" the behavior

    def receive_salary(self, amount):
        return self.payment_method.pay(amount)

# any combination is possible, without a new class:
e1 = Employee("ali", BankPayment())
e2 = Employee("sara", CashPayment())
print(e1.receive_salary(5_000_000))  # 5000000 deposited to bank account
print(e2.receive_salary(3_000_000))  # 3000000 paid in cash

# you can even change behavior at runtime:
e1.payment_method = CashPayment()
print(e1.receive_salary(5_000_000))  # 5000000 paid in cash
```

با این طراحی، «نحوه‌ی پرداخت» به یک قطعه‌ی مستقل تبدیل شد که به کلاس `Employee` تزریق می‌شود. افزودن یک روش پرداخت جدید، صرفاً به معنای ایجاد یک کلاس جدید است و نیازی به تغییر در کلاس `Employee` یا سایر بخش‌های سیستم ندارد. چه بسا که می‌توان رفتار را حتی در **زمان اجرا** نیز تغییر داد؛ قابلیتی که در ساختار ایستای ارث‌بری وجود ندارد. (این الگو، در واقع همان **الگوی Strategy** است که یکی از الگوهای طراحی بنیادین محسوب می‌شود.)

### تفاوت‌های عملی ارث‌بری و ترکیب

| ویژگی | ارث‌بری (Is-A) | ترکیب (Has-A) |
| :--- | :--- | :--- |
| **رابطه** | «یک نوع از...» است | «... دارد» |
| **زمان اتصال** | ثابت، در زمان تعریف کلاس | منعطف، حتی قابل تغییر در زمان اجرا |
| **نوع وابستگی** | محکم (به کلاس والد و جزئیات آن) | سست (فقط به رابط یا قرارداد جزء) |
| **انفجار کلاس** | با هر بُعد جدید، تعداد کلاس‌ها ضربدری می‌شود | ندارد؛ با افزودن کلاس‌های جدید، ساختار منبسط می‌شود |
| **تست‌پذیری** | دشوارتر (به دلیل وابستگی به زنجیره‌ی والد) | آسان‌تر (قابلیت جایگزینی جزء با نمونه‌های ساختگی) |

### آیا ترکیب همیشه بهتر است؟ بررسی Tradeoffها

پاسخ صریح به این سوال، **خیر** است. ارث‌بری همچنان جایگاه ویژه‌ی خود را در طراحی نرم‌افزار دارد:

- ارث‌بری زمانی انتخاب درستی است که رابطه‌ی **واقعی «Is-A»** برقرار باشد و شما نیاز به **به ارث بردن کامل رفتار و رابط** والد داشته باشید. بهترین مثال برای این حالت، تعریف **استثناهای سفارشی** یا توسعه‌ی کلاس‌های پایه‌ای است که فریم‌ورک‌ها در اختیار شما قرار می‌دهند.
- ترکیب زمانی ارجحیت دارد که رابطه از نوع **«Has-A»** باشد، یا به **انعطاف‌پذیری و قابلیت جایگزینی** اجزا در زمان اجرا نیاز داشته باشید.

با این حال، ترکیب نیز هزینه‌هایی دارد. گاهی نوشتن کد ترکیب، اندکی بیشتر از ارث‌بری زمان می‌برد (زیرا باید متدها را به صورت دستی به جزء واگذار کنید) و یک لایه‌ی غیرمستقیم به معماری اضافه می‌کند. در مقابل، ارث‌بری مختصرتر است اما وابستگی محکم‌تری ایجاد می‌کند که تغییرات آتی را با چالش مواجه می‌سازد. همانند همیشه، هیچ انتخاب مطلقاً درستی وجود ندارد؛ باید رابطه‌ی واقعی در حوزه‌ی مسئله را بسنجید و آگاهانه تصمیم بگیرید.

### قوانین و اصول طراحی

#### قانون دیمتر: «با غریبه‌ها حرف نزن»

قانون دیمتر (Law of Demeter) که به «اصل کمترین آگاهی» نیز معروف است، می‌گوید که هر واحد (تابع یا متد) باید فقط با **«دوستان نزدیک»** خود در تعامل باشد و از صحبت با «غریبه‌ها» پرهیز کند. در عمل، این قانون به معنای پرهیز از **زنجیره‌های طولانی دسترسی به نقطه** (مانند `a.b.c.d`) است.

**نمونه‌ی نقض قانون:**
```python
# violating the Law of Demeter: a long chain, hidden dependency
class Order:
    def total_with_discount(self):
        # we became dependent on the internal structure of customer, wallet, account
        return self.customer.wallet.account.balance * 0.9
```
مشکل اساسی این کد، وابستگی پنهان کلاس `Order` به ساختار داخلی چندین لایه‌ی دیگر (`customer`، `wallet` و `account`) است. اگر کوچکترین تغییری در هر یک از این کلاس‌ها ایجاد شود، احتمال شکستن کلاس `Order` بسیار بالاست.

**راه‌حل پیروی از قانون:**
```python
# following the Law of Demeter: ask your direct friend, not its internals
class Customer:
    def available_balance(self):
        return self.wallet.balance()  # Customer manages its own details

class Order:
    def total_with_discount(self):
        return self.customer.available_balance() * 0.9
```
اکنون کلاس `Order` فقط با دوست مستقیم خود، یعنی `Customer`، صحبت می‌کند و صرفاً یک عدد ساده از او درخواست می‌کند. اگر ساختار داخلی `Customer` تغییر کند، کلاس `Order` کاملاً در امان خواهد ماند.

#### اصل Tell, Don't Ask

این اصل راهنما می‌گوید: **به جای اینکه از یک شیء داده‌ای بپرسید و سپس بر اساس آن، خودتان تصمیم بگیرید، به آن شیء بگویید که چه کاری باید انجام دهد و بگذارید خودش تصمیم بگیرد.**

**رویکرد Ask (پرسیدن):**
```python
# Ask: the outside decides — logic gets scattered
if account.balance >= amount:
    account.balance -= amount
else:
    raise ValueError("insufficient balance")
```
در اینجا، منطق بررسی موجودی و برداشت، بیرون از شیء `account` و در جای دیگری از کد پیاده‌سازی شده است.

**رویکرد Tell (گفتن):**
```python
# Tell: the object itself decides — logic stays centralized
account.withdraw(amount)
```
در این رویکرد، منطق کسب‌وکار (قانون موجودی) درون خود شیء `account` متمرکز می‌شود. مزیت این کار، جلوگیری از **پخش شدن منطق** در سراسر کد است و با اصل **کپسوله‌سازی** که در فصل چهارم با آن آشنا شدیم، همخوانی کامل دارد.

#### کلاس‌های Anemic در برابر Rich

- **کلاس Anemic (بی‌رفتار یا کم‌خون)**: کلاسی است که صرفاً **داده** دارد و هیچ رفتار و متد مرتبطی در خود تعریف نکرده است. تمام منطق مربوط به این داده‌ها، در جای دیگری (مانند سرویس‌ها) پیاده‌سازی می‌شود.
- **کلاس Rich (غنی)**: کلاسی است که هم **داده** و هم **رفتار** مرتبط با آن داده را در خود دارد. منطق، در نزدیکی داده‌ای که روی آن عمل می‌کند، قرار می‌گیرد.

```python
# Anemic — just a data container
class AnemicAccount:
    def __init__(self):
        self.balance = 0

# and the logic elsewhere:
def withdraw(account, amount):  # behavior is separated from data
    if amount > account.balance:
        raise ValueError()
    account.balance -= amount

# Rich — data and behavior together
class RichAccount:
    def __init__(self):
        self.balance = 0

    def withdraw(self, amount):  # behavior is close to the data
        if amount > self.balance:
            raise ValueError()
        self.balance -= amount
```

کلاس‌های Rich، به طور معمول، رویکردی شیء‌گراتر هستند و منطق را در یک جا متمرکز می‌کنند. با این حال، در برخی معماری‌ها مانند لایه‌های صرفاً داده‌ای یا DTOها، استفاده از کلاس‌های Anemic یک انتخاب عمدی و قابل‌قبول است. انتخاب بین این دو، همچنان به موقعیت و نیاز پروژه بستگی دارد.

**سناریوی استفاده از کلاس‌های Anemic به عنوان یک مزیت**: فرض کنید در یک سیستم، نیاز به انتقال داده بین لایه‌های مختلف دارید؛ برای نمونه، داده‌های یک کاربر را از پایگاه داده بخوانید، به لایه‌ی کسب‌وکار بفرستید و در نهایت به صورت JSON به کاربر نمایش دهید. در چنین مواردی، تعریف یک کلاس `UserDTO` (Data Transfer Object) که فقط فیلدهای `id`، `name` و `email` را نگهداری می‌کند و هیچ متد تجاری‌ای ندارد، کاری منطقی و مرسوم است. این کلاس به عنوان یک **حامل داده‌ی خام (Data Carrier)** عمل می‌کند و رفتارهای مربوط به اعتبارسنجی، محاسبه یا ذخیره‌سازی در لایه‌های دیگر (مانند سرویس‌ها) پیاده‌سازی می‌شوند. استفاده از کلاس‌های Anemic در این سناریو، وابستگی بین لایه‌ها را کاهش داده و تغییرات در هر لایه را مستقل‌تر می‌کند.

```python
# A typical DTO (Anemic) class in a layered architecture
class UserDTO:
    def __init__(self, user_id, name, email):
        self.id = user_id
        self.name = name
        self.email = email

# The logic is in a separate service layer
class UserService:
    def register(self, user_dto):
        # validation logic here
        if not user_dto.email:
            raise ValueError("Email is required")
        # save logic here
        self.repository.save(user_dto)
```

در اینجا، `UserDTO` یک کلاس صرفاً داده‌ای است که به راحتی می‌توان آن را سریالایز یا دی‌سریالایز کرد و در لایه‌های مختلف سیستم گرداند.

### معماری لایه‌ای (Layered Architecture)

معماری لایه‌ای، کد را به **لایه‌های افقی** با مسئولیت‌های کاملاً مجزا تقسیم می‌کند. رایج‌ترین شکل آن، شامل سه لایه‌ی اصلی است:

```
┌─────────────────────────────────┐
│   Presentation (UI/API)         │  ← ورودی/خروجی، نمایش داده‌ها
├─────────────────────────────────┤
│   Business Logic (Service)      │  ← قوانین اصلی برنامه
├─────────────────────────────────┤
│   Data Access (Repository)      │  ← ذخیره‌سازی و بازیابی داده
└─────────────────────────────────┘
```

**اصل اساسی** در این معماری، **جریان وابستگی از بالا به پایین** است. هر لایه فقط با لایه‌ی بلافصل زیرین خود ارتباط برقرار می‌کند.

بیایید این مفهوم را با یک مثال ساده پیاده‌سازی کنیم:

```python
# Data layer: only store and retrieve
class UserRepository:
    def __init__(self):
        self._users = {}

    def save(self, user):
        self._users[user.id] = user

    def get(self, user_id):
        return self._users.get(user_id)

# Logic layer: business rules
class UserService:
    def __init__(self, repository):
        self.repository = repository  # depends on the data layer

    def register(self, user):
        if self.repository.get(user.id):
            raise ValueError("duplicate user")
        self.repository.save(user)

# Presentation layer: interaction with the user
class UserController:
    def __init__(self, service):
        self.service = service  # depends on the logic layer

    def handle_register(self, user):
        try:
            self.service.register(user)
            return "registered successfully"
        except ValueError as e:
            return f"Error: {e}"
```

مزیت بزرگ این معماری، **استقلال** و **قابلیت تست** هر لایه به صورت جداگانه است. برای نمونه، می‌توانیم `UserRepository` حافظه‌ای را با یک نسخه‌ی مبتنی بر پایگاه‌داده جایگزین کنیم، بدون آنکه کوچکترین تغییری در کلاس `UserService` ایجاد شود.

**ارتباط معماری لایه‌ای با مفهوم ترکیب**: در نگاه اول، ممکن است معماری لایه‌ای صرفاً یک راه‌کار سازمان‌دهی کد به نظر برسد؛ اما در حقیقت، این معماری یک **نمونه‌ی عینی و توسعه‌یافته‌ی اصل ترکیب در مقیاس یک سیستم کامل** است. درست همان‌طور که یک کلاس (`Car`) با نگهداشتن یک نمونه از کلاس دیگر (`Engine`) کار خود را انجام می‌دهد، در معماری لایه‌ای نیز هر لایه، لایه‌ی زیرین خود را به عنوان یک «جزء» در خود نگه می‌دارد و وظایف مرتبط را به آن واگذار می‌کند. به بیان دیگر:

- **ترکیب در سطح کلاس** (نگهداشتن یک شیء درون شیء دیگر) → **معادل لایه‌بندی در سطح معماری** (نگهداشتن یک لایه‌ی زیرین به عنوان جزء) است.
- **تزریق وابستگی** (ارسال جزء در سازنده) در یک کلاس → **معادل ارتباط از بالا به پایین** بین لایه‌هاست که وابستگی را به‌صورت شفاف مدیریت می‌کند.
- **قابلیت جایگزینی** یک جزء با نمونه‌ای دیگر (مانند تغییر روش پرداخت) در یک کلاس → **معادل جایگزینی** یک لایه‌ی کامل (مثلاً تعویض پایگاه‌داده) بدون تأثیر بر لایه‌های دیگر است.

این شباهت نشان می‌دهد که ترکیب، صرفاً یک الگوی طراحی در سطح کد نیست، بلکه **اصلی بنیادین برای تفکر معماری** است که در تمام سطوح، از یک کلاس کوچک تا یک سیستم بزرگ، قابل اعمال است. به همین دلیل، معماری لایه‌ای به عنوان یک **مطالعه‌ی موردی (Case Study)** برای درک عینی‌تر و عمیق‌تر از مفاهیم انتزاعی ترکیب در این فصل گنجانده شده است.

**مقایسه با معماری هگزاگونال**: معماری لایه‌ای، ساده و آشنا است اما جریان وابستگی آن یک‌طرفه و سلسله‌مراتبی است. معماری هگزاگونال (Ports & Adapters) انعطاف‌پذیری بیشتری ارائه می‌دهد و تست‌پذیرتر است، اما در عوض پیچیدگی بیشتری نیز دارد. برای بسیاری از پروژه‌های کوچک و متوسط، معماری لایه‌ای انتخابی کاملاً کافی و به مراتب مناسب‌تر است.

### ترکیب در عمل و مزایای آن

#### مثالی که بدون شیءگرایی دشوار است

فرض کنید سیستمی برای ارسال اعلان (Notification) از طریق کانال‌های مختلف (ایمیل، اس ام اس، نوتیفیکیشن درون برنامه‌ای) طراحی می‌کنیم.

**رویکرد رویه‌ای (Procedural)** به سرعت به مجموعه‌ای از شرط‌های `if/elif` تبدیل می‌شود:
```python
# procedural: every new channel means a new elif everywhere
def send(channel, message):
    if channel == "email":
        print(f"Email: {message}")
    elif channel == "sms":
        print(f"SMS: {message}")
    elif channel == "push":
        print(f"Push: {message}")
    # a new channel → edit this function (and every similar function)
```
هر بار که یک کانال جدید اضافه می‌شود، باید تابع `send` (و تمام توابع مشابه آن) را ویرایش کنیم. این رویکرد، به شدت در برابر تغییرات آسیب‌پذیر است.

**بازنویسی شیءگرا با ترکیب و چندریختی (Polymorphism)** :
```python
class Channel:
    def send(self, message):
        pass

class EmailChannel(Channel):
    def send(self, message):
        print(f"Email: {message}")

class SMSChannel(Channel):
    def send(self, message):
        print(f"SMS: {message}")

class Notifier:
    def __init__(self, channels):
        self.channels = channels  # a combination of channels

    def broadcast(self, message):
        for channel in self.channels:
            channel.send(message)

notifier = Notifier([EmailChannel(), SMSChannel()])
notifier.broadcast("Hello")
# adding a new channel: just a new class, without changing Notifier
```
در این طراحی، افزودن یک کانال جدید به معنای **افزودن یک کلاس جدید** است، نه **ویرایش کدهای موجود**. کد قدیمی دست نخورده و در امان می‌ماند و این، یکی از بزرگترین دستاوردهای طراحی شیءگرا است.

#### چرا شیءگرایی ریفکتور را امن‌تر می‌کند؟

در کد رویه‌ای، منطق کسب‌وکار در سراسر توابع مختلف پخش شده است. تغییر یک قاعده، به معنای پیدا کردن و ویرایش تمام نقاطی است که آن قاعده در آن‌ها تکرار شده و جا ماندن یکی از آن‌ها، یعنی ایجاد یک باگ جدید.

در کد شیءگرا، هر قاعده در یک کلاس خاص متمرکز می‌شود. ریفکتور یعنی تغییر همان یک نقطه، و مرزهای شفاف کلاس‌ها از نشت خطا به سایر نقاط کد جلوگیری می‌کنند.

#### طراحی برای تست‌پذیری

ترکیب، به طور طبیعی، تست‌پذیری را به همراه می‌آورد، زیرا شما به راحتی می‌توانید اجزای یک کلاس را با نسخه‌های **ساختگی (Mock)** جایگزین کنید:

```python
class RealEmailChannel:
    def send(self, message):
        # actually sends email (slow, network-dependent)
        pass

class FakeChannel:  # test replacement
    def __init__(self):
        self.sent = []

    def send(self, message):
        self.sent.append(message)  # only records

# in tests:
fake = FakeChannel()
notifier = Notifier([fake])
notifier.broadcast("test")
assert fake.sent == ["test"]  # no network, fast and deterministic
```

چون کلاس `Notifier` به **رابط** (متد `send`) وابسته است، نه به کلاس مشخص، در محیط تست می‌توانیم به راحتی یک نسخه‌ی جعلی را تزریق کنیم. این همان مفهوم **تزریق وابستگی (Dependency Injection)** است که هسته‌ی اصلی **اصل وارونگی وابستگی (Dependency Inversion)** را در فصل بعد تشکیل خواهد داد.

#### مزیت اشیای بی‌حالت در طراحی

اشیای بی‌حالت (Stateless) اشیایی هستند که هیچ داده‌ی قابل تغییری را در خود نگهداری نمی‌کنند؛ یعنی پس از ساخته شدن، هیچ یک از ویژگی‌های آن‌ها تغییر نمی‌کند. این ویژگی ساده، مزیت‌های شگرفی در طراحی و معماری نرم‌افزار به همراه دارد:

**امنیت در محیط‌های چندنخی (Thread-Safety)**: اشیای بی‌حالت را می‌توان بدون هیچ نگرانی از تداخل یا شرایط رقابتی (Race Condition) بین چندین نخ (Thread) به اشتراک گذاشت. برای نمونه، یک کلاس `Calculator` که متدهای `add` و `subtract` را بدون نگهداری هیچ حالتی پیاده‌سازی کرده است، می‌تواند به طور همزمان توسط هزاران درخواست وب استفاده شود:

```python
class StatelessCalculator:
    # no instance variables, no shared state
    def add(self, a, b):
        return a + b

    def multiply(self, a, b):
        return a * b

# one instance can be shared safely across all threads
calc = StatelessCalculator()
# can be used in many threads concurrently without any synchronization
```

**قابلیت کش کردن و به اشتراک‌گذاری آسان**: اشیای بی‌حالت را می‌توان بدون نگرانی از تغییرات ناخواسته، در حافظه‌ی کش (Cache) نگهداری کرد یا در سراسر برنامه به اشتراک گذاشت. این امر به بهبود کارایی و کاهش مصرف حافظه کمک شایانی می‌کند.

**سادگی در تست**: آزمون یک شیء بی‌حالت بسیار ساده است، زیرا رفتار آن تنها به ورودی‌های متد وابسته است و نه به ترتیب فراخوانی متدها یا تاریخچه‌ی تعاملات. به عبارت دیگر، خروجی یک متد، صرفاً تابعی از ورودی‌های آن است (همانند توابع ریاضی).

**تفاوت با اشیای با حالت (Stateful)**: در مقابل، اشیایی که حالت داخلی را تغییر می‌دهند، پیچیدگی‌های خاص خود را دارند. برای نمونه، کلاس `Counter` که یک شمارنده را نگهداری می‌کند، نیازمند همگام‌سازی در محیط‌های چندنخی و توجه به ترتیب فراخوانی متدها در تست‌هاست:

```python
class StatefulCounter:
    def __init__(self):
        self.count = 0  # mutable state

    def increment(self):
        self.count += 1  # state changes over time
        return self.count

# this instance cannot be safely shared across threads without synchronization
# and its behavior depends on the order of method calls
counter = StatefulCounter()
print(counter.increment())  # 1
print(counter.increment())  # 2
```

**قاعده‌ی عملی**: هر جا که رفتاری واقعاً به حالت خاصی نیاز ندارد (یعنی می‌توان آن را به صورت تابعی محض نوشت)، طراحی آن به صورت بی‌حالت، انتخاب برتری است. بسیاری از کلاس‌های سرویس، ابزارهای کمکی (Utility) و کارخانه‌های سازنده (Factory) که صرفاً منطق را پیاده‌سازی می‌کنند، کاندیداهای مناسبی برای بی‌حالتی هستند.

### خلاصه

| مفهوم | خلاصه |
| :--- | :--- |
| **ترکیب (Composition)** | نگهداشتن اجزا و واگذاری کار به آن‌ها (Has-A) |
| **ترجیح بر ارث‌بری** | انعطاف بیشتر، وابستگی کمتر، جلوگیری از انفجار کلاس |
| **قانون دیمتر** | فقط با دوست مستقیم صحبت کن، نه با زنجیره‌های `a.b.c.d` |
| **Tell, Don't Ask** | به شیء بگو چه کاری انجام دهد، حالت درونی‌اش را نپرس |
| **Anemic در برابر Rich** | داده‌ی خالص در برابر داده و رفتار توأمان |
| **معماری لایه‌ای** | تفکیک به لایه‌های Presentation، Business و Data Access |
| **تست‌پذیری** | با تزریق اجزای ساختگی، به لطف وابستگی به رابط (Interface) |

**نکته‌ی طلایی این فصل:** هیچ راه‌حل مطلقاً درستی برای تمام مسائل وجود ندارد. ترکیب اغلب بهتر از ارث‌بری است، اما نه همیشه. معماری لایه‌ای عالی است، اما برای هر پروژه‌ای مناسب نیست. مهارت واقعی در طراحی، توانایی **سنجیدن Tradeoffها** و انتخاب آگاهانه‌ی بهترین گزینه برای موقعیت خاص است، نه پیروی کورکورانه از یک قاعده.

در فصل بعد، با ابزاری قدرتمند برای سنجش کیفیت طراحی‌های خود آشنا خواهیم شد: **اصول SOLID**؛ پنج اصل طلایی که به شما کمک می‌کنند تا نرم‌افزارهایی منعطف، قابل‌نگهداری و مقاوم در برابر تغییرات طراحی کنید.