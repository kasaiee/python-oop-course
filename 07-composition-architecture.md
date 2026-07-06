# فصل هفتم: Composition، معماری و طراحی پیشرفته

---

## مقدمه

دو فصل را صرفِ ارث‌بری کردیم. اما یک اعتراف: ارث‌بری، در بسیاری از موارد، **انتخابِ اول نیست**.

در این فصل با **Composition (ترکیب)** آشنا می‌شویم — رویکردی که در بیشتر سناریوهای واقعی، جایگزینِ بهتری برای ارث‌بری است. یاد می‌گیریم چرا اصلِ مشهورِ «ترکیب را بر ارث‌بری ترجیح بده» این‌قدر جدی گرفته می‌شود، و چند اصلِ طراحیِ کلیدی — قانون دیمتر، Tell Don't Ask، و معماری لایه‌ای — که کد شما را از «کارکننده» به «قابل‌نگهداری» می‌رسانند.

---

## چرا Composition و طراحی پیشرفته را یاد بگیریم؟

ارث‌بری یک رابطه‌ی **Is-A** («یک نوعِ...») می‌سازد. اما بیشتر روابطِ دنیای واقعی از نوع **Has-A** («دارد») هستند: ماشین موتور **دارد**، سفارش پرداخت **دارد**، کاربر آدرس **دارد**. تحمیلِ ارث‌بری به این روابط، کد را شکننده می‌کند.

استفاده‌ی درست از Composition به شما کمک می‌کند:

- کدی با وابستگیِ کمتر (Loose Coupling) بنویسید
- از شکنندگیِ سلسله‌مراتب‌های ارث‌بری فرار کنید
- کدِ تست‌پذیرتر بسازید (چون اجزا را می‌توان جدا جایگزین کرد)
- معماریِ مقیاس‌پذیر و قابل‌نگهداری طراحی کنید

---

## Composition — ترکیب

### Composition چیست؟

**Composition** یعنی یک کلاس، نمونه‌هایی از کلاس‌های دیگر را **در خود نگه دارد** و کارش را با کمکِ آن‌ها انجام دهد — به‌جای اینکه از آن‌ها ارث ببرد. رابطه‌ای که می‌سازد **Has-A** است.

مثال: یک ماشین. با ارث‌بری، طراحیِ اشتباه این است:

```python
# wrong: a car is not a "type of" engine!
class Engine:
    def start(self):
        return "started"

class Car(Engine):        # Is-A wrong: a Car is not an Engine
    pass
```

با Composition، طراحیِ درست:

```python
# correct: a car "has" an engine
class Engine:
    def start(self):
        return "engine started"

class Car:
    def __init__(self):
        self.engine = Engine()       # Car has an Engine (Has-A)

    def start(self):
        return self.engine.start()   # delegates the work to the engine

c = Car()
print(c.start())   # engine started
```

`Car` موتور را **در خود دارد** و کار را به آن **واگذار (delegate)** می‌کند. این طبیعی‌تر و منعطف‌تر است.

### چرا ترکیب اغلب بر ارث‌بری ترجیح دارد؟

بیایید یک مثال واقعی را در دو رویکرد ببینیم تا تفاوت ملموس شود. فرض کنید انواعِ کارمند با روش‌های مختلفِ پرداخت داریم.

**رویکرد ارث‌بری** — به‌سرعت منفجر می‌شود:

```python
class Employee: ...
class FullTimeEmployee(Employee): ...
class PartTimeEmployee(Employee): ...
# now if we want to add a second dimension — how salary is received (bank/cash) —
# we would have to build every combination:
# FullTimeBankEmployee, FullTimeCashEmployee,
# PartTimeBankEmployee, PartTimeCashEmployee ...
# combinatorial explosion!
```

هر بُعدِ جدید، تعدادِ کلاس‌ها را ضربدری می‌کند. این «انفجارِ کلاس» یکی از بزرگ‌ترین دام‌های ارث‌بری است.

**رویکرد Composition** — منعطف و بی‌انفجار:

```python
class PaymentMethod:
    def pay(self, amount): ...

class BankPayment(PaymentMethod):
    def pay(self, amount):
        return f"{amount} deposited to bank account"

class CashPayment(PaymentMethod):
    def pay(self, amount):
        return f"{amount} paid in cash"

class Employee:
    def __init__(self, name, payment_method):
        self.name = name
        self.payment_method = payment_method    # we "inject" the behavior

    def receive_salary(self, amount):
        return self.payment_method.pay(amount)

# any combination is possible, without a new class:
e1 = Employee("ali", BankPayment())
e2 = Employee("sara", CashPayment())
print(e1.receive_salary(5_000_000))   # 5000000 deposited to bank account
print(e2.receive_salary(3_000_000))   # 3000000 paid in cash

# you can even change behavior at runtime:
e1.payment_method = CashPayment()
print(e1.receive_salary(5_000_000))   # 5000000 paid in cash
```

با Composition، «نحوه‌ی پرداخت» یک قطعه‌ی مستقل شد که به کارمند تزریق می‌شود. اضافه‌کردن روش پرداختِ جدید فقط یک کلاس تازه است، بدون دست‌زدن به `Employee`. و می‌توان رفتار را حتی در زمان اجرا عوض کرد — چیزی که ارث‌بری هرگز اجازه نمی‌دهد. (این دقیقاً الگوی Strategy است.)

### تفاوت عملی ارث‌بری و Composition

| | ارث‌بری (Is-A) | Composition (Has-A) |
|---|----------------|----------------------|
| رابطه | «یک نوعِ...» | «...دارد» |
| زمان اتصال | ثابت، در زمان تعریف | منعطف، حتی در زمان اجرا |
| وابستگی | محکم (به کل والد) | سست (فقط به رابط جزء) |
| انفجار کلاس | با هر بُعدِ جدید | ندارد |
| تست | سخت‌تر (والد را هم می‌کشد) | آسان (جزء را mock می‌کنی) |

### آیا Composition همیشه بهتر است؟ Tradeoffها

نه. ارث‌بری هنوز جای خودش را دارد:

- **ارث‌بری** وقتی درست است که رابطه‌ی واقعیِ Is-A باشد و بخواهید رفتار و رابطِ والد را کاملاً به ارث ببرید (مثل استثناهای سفارشی فصل چهارم، یا کلاس‌های پایه‌ی فریمورک).
- **Composition** وقتی بهتر است که رابطه Has-A باشد، یا بخواهید انعطاف و جایگزینیِ اجزا را داشته باشید.

**Tradeoff:** Composition گاهی کدِ بیشتری می‌طلبد (باید متدها را دستی واگذار کنید) و یک لایه‌ی غیرمستقیم اضافه می‌کند. ارث‌بری مختصرتر است اما وابستگیِ محکم‌تری می‌سازد. مثل همیشه: هیچ راهِ مطلقاً درستی نیست؛ رابطه‌ی واقعیِ دامنه را بسنجید و انتخاب کنید.

---

## قوانین و اصول طراحی

### قانون دیمتر: «با غریبه‌ها حرف نزن»

**قانون دیمتر** (Law of Demeter) می‌گوید هر واحد باید فقط با «دوستانِ نزدیکش» حرف بزند، نه با غریبه‌ها. عملاً یعنی: از زنجیره‌های طولانیِ نقطه (`a.b.c.d`) پرهیز کنید.

```python
# violating the Law of Demeter: a long chain, hidden dependency
class Order:
    def total_with_discount(self):
        # we became dependent on the internal structure of customer, then wallet, then balance
        return self.customer.wallet.account.balance * 0.9
```

مشکل: `Order` حالا به ساختارِ داخلیِ `customer`، `wallet` و `account` وابسته است. اگر هرکدام تغییر کند، `Order` می‌شکند. این وابستگیِ پنهان، شکننده است.

```python
# following the Law of Demeter: ask your direct friend, don't ask for its internals
class Customer:
    def available_balance(self):
        return self.wallet.balance()     # Customer manages its own details

class Order:
    def total_with_discount(self):
        return self.customer.available_balance() * 0.9
```

حالا `Order` فقط با `Customer` حرف می‌زند (دوستِ مستقیم) و از او یک عددِ ساده می‌خواهد. اگر ساختار داخلیِ `Customer` عوض شود، `Order` دست‌نخورده می‌ماند.

### Tell, Don't Ask

اصلِ **Tell, Don't Ask** («بگو، نپرس») می‌گوید: به‌جای اینکه داده‌ی یک شیء را بپرسی و بیرون تصمیم بگیری، به شیء **بگو** چه کند و بگذار خودش تصمیم بگیرد.

```python
# Ask: the outside decides — logic gets scattered
if account.balance >= amount:
    account.balance -= amount
else:
    raise ValueError("insufficient balance")

# Tell: the object itself decides — logic stays centralized
account.withdraw(amount)   # the balance rule is inside account itself
```

مزیت رویکردِ Tell: منطقِ کسب‌وکار (قانونِ موجودی) درون خودِ شیء متمرکز می‌ماند، نه پخش در سراسر کد. این با کپسول‌سازیِ فصل چهارم هم‌راستاست.

### کلاس‌های Anemic در برابر Rich

- **کلاس Anemic (کم‌خون/بی‌رفتار):** فقط داده دارد، بدون رفتار. منطق در جای دیگری (سرویس‌ها) پخش می‌شود.
- **کلاس Rich (غنی):** داده و رفتارِ مربوط را با هم دارد. منطق، نزدیکِ داده‌اش می‌ماند.

```python
# Anemic — just a data container
class AnemicAccount:
    def __init__(self):
        self.balance = 0
# and the logic elsewhere:
def withdraw(account, amount):        # behavior is separated from data
    if amount > account.balance: raise ValueError()
    account.balance -= amount

# Rich — data and behavior together
class RichAccount:
    def __init__(self):
        self.balance = 0
    def withdraw(self, amount):        # behavior close to the data
        if amount > self.balance: raise ValueError()
        self.balance -= amount
```

**Tradeoff:** کلاس‌های Rich معمولاً شیءگراییِ بهتری‌اند و منطق را جمع نگه می‌دارند. اما در بعضی معماری‌ها (مثلاً لایه‌های داده‌ی خالص، یا DTOها) کلاس Anemic عمداً انتخاب می‌شود. مدل‌های داده‌ی مدرن مثل `dataclass` (فصل دهم) گاهی عامدانه Anemic‌اند. باز هم بستگی به موقعیت دارد.

### معماری لایه‌ای

**معماری لایه‌ای** (Layered Architecture) کد را به لایه‌های افقی با مسئولیت‌های مجزا تقسیم می‌کند. رایج‌ترین شکلش سه لایه دارد:

```
┌─────────────────────────────────┐
│  Presentation (رابط کاربر/API)    │  ← ورودی/خروجی، نمایش
├─────────────────────────────────┤
│  Business Logic (منطق کسب‌وکار)    │  ← قوانین اصلی برنامه
├─────────────────────────────────┤
│  Data Access (دسترسی به داده)     │  ← ذخیره و بازیابی
└─────────────────────────────────┘
        جریان وابستگی: بالا → پایین
```

هر لایه فقط با لایه‌ی زیرینش حرف می‌زند. مثال ساده:

```python
# data layer: only store and retrieve
class UserRepository:
    def __init__(self):
        self._users = {}
    def save(self, user):
        self._users[user.id] = user
    def get(self, user_id):
        return self._users.get(user_id)

# logic layer: business rules
class UserService:
    def __init__(self, repository):
        self.repository = repository        # depends on the data layer
    def register(self, user):
        if self.repository.get(user.id):
            raise ValueError("duplicate user")
        self.repository.save(user)

# presentation layer: interaction with the user (simplified here)
class UserController:
    def __init__(self, service):
        self.service = service              # depends on the logic layer
    def handle_register(self, user):
        try:
            self.service.register(user)
            return "registered successfully"
        except ValueError as e:
            return f"Error: {e}"
```

مزیت: هر لایه مستقل تست و جایگزین می‌شود. می‌توانید `UserRepository` (مثلاً حافظه‌ای) را با نسخه‌ی دیتابیسی عوض کنید، بی‌آنکه `UserService` تغییر کند.

**در مقایسه با معماری هگزاگونال:** معماری لایه‌ای ساده و آشناست اما جریانِ وابستگی‌اش یک‌طرفه و سلسله‌مراتبی است. معماری هگزاگونال (Ports & Adapters) انعطاف بیشتری می‌دهد — هسته‌ی منطق در مرکز، و همه‌ی وابستگی‌های بیرونی از طریق «پورت»های انتزاعی. هگزاگونال تست‌پذیرتر است اما پیچیده‌تر. برای پروژه‌های کوچک و متوسط، معماری لایه‌ای اغلب کافی و مناسب‌تر است. باز هم یک Tradeoff.

---

## مقایسه و مثال‌ها

### مثالی که بدون شیءگرایی سخت است

فرض کنید یک سیستم اعلان (Notification) داریم که باید از چند کانال پیام بفرستد. **رویکرد رویه‌ای** به‌سرعت به `if/elif` تبدیل می‌شود:

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

**بازنویسیِ شیءگرا** با Composition و چندریختی:

```python
class Channel:
    def send(self, message): ...

class EmailChannel(Channel):
    def send(self, message):
        print(f"Email: {message}")

class SMSChannel(Channel):
    def send(self, message):
        print(f"SMS: {message}")

class Notifier:
    def __init__(self, channels):
        self.channels = channels        # a combination of channels

    def broadcast(self, message):
        for channel in self.channels:
            channel.send(message)

notifier = Notifier([EmailChannel(), SMSChannel()])
notifier.broadcast("Hello")
# adding a new channel: just a new class, without changing Notifier
```

چرا شیءگرایی اینجا راحت‌تر کرد؟ چون افزودنِ کانالِ جدید دیگر یعنی **ویرایشِ کدِ موجود** نیست، بلکه **افزودنِ کدِ تازه** است. کدِ قدیم دست‌نخورده و امن می‌ماند.

### چرا شیءگرایی ریفکتور را امن‌تر می‌کند؟

در کدِ رویه‌ای، منطق در سراسر توابع پخش است؛ تغییرِ یک قاعده یعنی پیداکردن و ویرایشِ همه‌ی جاهایی که آن قاعده تکرار شده — و جاماندنِ یکی، یعنی باگ. در کدِ شیءگرا، هر قاعده در **یک** کلاس متمرکز است؛ ریفکتور یعنی تغییرِ همان یک‌جا، و مرزهای روشنِ کلاس‌ها جلوی نشتِ خطا را می‌گیرند.

### طراحی برای تست‌پذیری

Composition به‌طور طبیعی تست‌پذیری می‌آورد، چون می‌توانید اجزا را با نسخه‌های ساختگی (mock) جایگزین کنید:

```python
class RealEmailChannel:
    def send(self, message):
        # actually sends email (slow, network-dependent)
        ...

class FakeChannel:                       # test replacement
    def __init__(self):
        self.sent = []
    def send(self, message):
        self.sent.append(message)        # only records

# in tests:
fake = FakeChannel()
notifier = Notifier([fake])
notifier.broadcast("test")
assert fake.sent == ["test"]              # no network, fast and deterministic
```

چون `Notifier` به **رابط** (متد `send`) وابسته است نه به کلاسِ مشخص، در تست می‌توانیم نسخه‌ی جعلی تزریق کنیم. این همان **Dependency Injection** است که هسته‌ی اصلِ Dependency Inversion در فصل بعد را می‌سازد.

### مزیت اشیای بی‌حالت در طراحی

اشیای بی‌حالت (Stateless) — که در فصل دوم دیدیم — در طراحی مزیت دارند: چون حالتِ مشترکِ تغییرپذیر ندارند، ذاتاً امن‌تر (به‌خصوص چندنخی)، ساده‌تر برای تست، و قابلِ‌اشتراک‌گذاری بدون خطرند. هر جا رفتاری واقعاً به حالت نیاز ندارد، بی‌حالت نگهش دارید.

---

## خلاصه

```
ارث‌بری در برابر Composition
════════════════════════════════════════
  ارث‌بری     ──►  «Car یک Vehicle است»   (Is-A)
  Composition ──►  «Car یک Engine دارد»    (Has-A)

  قاعده‌ی طلایی:
  «Composition را بر Inheritance ترجیح بده»
  (مگر رابطه واقعاً Is-A باشد)
```

| اصل | خلاصه |
|-----|-------|
| Composition | نگه‌داشتن اجزا و واگذاریِ کار (Has-A) |
| ترجیح بر ارث‌بری | انعطاف بیشتر، وابستگیِ کمتر، بدون انفجارِ کلاس |
| قانون دیمتر | با دوستِ مستقیم حرف بزن، نه زنجیره‌ی `a.b.c.d` |
| Tell, Don't Ask | به شیء بگو چه کند، حالتش را نپرس |
| Anemic در برابر Rich | داده‌ی خالص در برابر داده+رفتار |
| معماری لایه‌ای | Presentation / Business / Data Access |
| تست‌پذیری | تزریقِ اجزای جعلی، به‌لطف وابستگی به رابط |

**نکته‌ی طلایی:** هیچ راه‌حلِ مطلقاً درستی وجود ندارد — این جمله را در این فصل بیش از هر جای دیگر جدی بگیرید. Composition اغلب بهتر است، اما نه همیشه. معماریِ لایه‌ای عالی است، اما نه برای هر پروژه. مهارتِ واقعیِ طراحی، سنجیدنِ Tradeoffها و انتخابِ مناسبِ موقعیت است، نه پیرویِ کورکورانه از یک قاعده.

---

## پرسش‌های مصاحبه

**۱. «Composition را به ارث‌بری ترجیح بده — یعنی همیشه؟»** — نه؛ یعنی «به‌طورِ پیش‌فرض اول Composition را بسنج». ارث‌بری برای رابطه‌ی Is-Aی واقعی و پایدار عالی است؛ Composition برای Has-A و ترکیبِ رفتارها. جوابِ ممتاز نشانه‌ی خطر می‌دهد: «هر جا برای رسیدن به یک قابلیت مجبورم چیزهای بی‌ربط را هم به ارث ببرم، ارث‌بری غلط است.»

**۲. «قانون دیمتر چیست؟ زنجیره‌ی a.b.c.d چه اشکالی دارد؟»** — «با غریبه‌ها حرف نزن»: هر شیء فقط با همسایه‌های مستقیمش کار کند. زنجیره‌ی طولانی یعنی کدِ شما به ساختارِ داخلیِ چند لایه آن‌طرف‌تر جوش خورده — یک تغییرِ ساختاری در انتهای زنجیره، همه‌ی مصرف‌کننده‌ها را می‌شکند. راه‌حل: متدی در همسایه که کار را «انجام دهد» به‌جای اینکه اندرونی‌اش را بیرون بدهد (Tell, Don't Ask).

**۳. «معماری لایه‌ای را توضیح بده و بگو منطق کجا می‌نشیند.»** — سه لایه: Presentation (تعامل)، Business/Service (قوانین)، Data (ذخیره‌سازی). قانونِ طلایی: وابستگی فقط رو به پایین؛ منطقِ کسب‌وکار در Service — نه در UI و نه پخش در کوئری‌ها. اگر گفتید «لایه‌ی Service همان Facade روی زیرسیستم‌هاست»، پیوندتان با الگوها هم روشن می‌شود.

---

**در فصل بعد:** حالا که ابزارها و اصولِ طراحی را داریم، سراغِ چارچوبی می‌رویم که کیفیتِ طراحی را می‌سنجد: **اصول SOLID** — پنج اصلِ طلاییِ طراحیِ شیءگرا.
