# فصل سیزدهم: تست‌نویسی برای کد شیءگرا

---

## مقدمه

در فصل‌های گذشته بارها جمله‌ای تکرار شد: «این طراحی تست‌پذیرتر است.» در فصل هفتم برای تست‌پذیری، وابستگی تزریق کردیم؛ در فصل هشتم DIP را با همین وعده خواندید؛ در فصل قبل دیدید Singleton چطور تست را زهر می‌کند. حالا وقتش رسیده که بدهی‌مان را صاف کنیم: **خودِ تست‌نویسی.**

بگذارید از یک صحنه‌ی واقعی شروع کنم. برنامه‌نویسی را دیده‌ام که از دست‌زدن به کدِ خودش می‌ترسید. سیستمِ تخفیفِ فروشگاه را شش ماه قبل نوشته بود، حالا مدیر یک قانونِ جدید می‌خواست، و او ساعت‌ها به صفحه زل زده بود — چون هیچ راهی نداشت بفهمد تغییرش کجاها را می‌شکند جز اینکه «امیدوار باشد». این حالت اسم دارد: **فلجِ ریفکتور**. و درمانش دقیقاً همین فصل است: مجموعه‌ای از تست‌ها که مثل تورِ ایمنیِ بندباز، زیرِ کدتان کشیده شده‌اند. با تور، حرکت‌های جسورانه ممکن می‌شود؛ بی‌تور، فقط می‌شود بی‌حرکت ایستاد.

در این فصل با `pytest` تست می‌نویسیم، با fixtureها تکرار را حذف می‌کنیم، با تزریقِ وابستگی و Mock کد را از دنیای بیرون جدا می‌کنیم، و مهم‌تر از همه: یاد می‌گیریم **چه چیزی را تست کنیم و چه چیزی را نه**.

---

## چرا تست‌نویسی مهارتِ درجه‌یکِ استخدام است؟

آگهی‌های استخدامِ پایتون را باز کنید؛ کنارِ Django و SQL تقریباً همیشه این را می‌بینید: «آشنا با تستِ خودکار / pytest». دلیلش اقتصادی است، نه سلیقه‌ای:

- **هزینه‌ی باگ در production ده‌ها برابرِ هزینه‌ی باگ در توسعه است.** تست، باگ را قبل از مشتری می‌گیرد.
- **کدِ بدونِ تست، قابلِ‌تغییر نیست.** و کدی که نشود تغییرش داد، دارایی نیست؛ بدهی است.
- **تست، مستنداتِ زنده است.** تستِ خوب نشان می‌دهد کلاس چطور قرار است استفاده شود — مستنداتی که هرگز از کد عقب نمی‌افتند، چون اگر عقب بیفتند قرمز می‌شوند.
- و در مصاحبه‌ها: «برای این کلاس چه تست‌هایی می‌نویسی؟» یکی از پرتکرارترین سوال‌های عملی است.

یک باورِ غلط را هم همین اول کنار بگذاریم: «تست‌نویسی وقتِ اضافه می‌خواهد.» تست‌نویسی وقتِ **جلوتر** می‌خواهد و وقتِ **بعداً** را چند برابر پس می‌دهد — هر بار که با خیالِ راحت ریفکتور می‌کنید، هر بار که باگ قبل از deploy می‌میرد، هر بار که همکارِ جدید از روی تست‌ها می‌فهمد سیستم چطور کار می‌کند.

---

## اولین تست: pytest در شصت ثانیه

پایتون در کتابخانه‌ی استاندارد ماژولِ `unittest` را دارد، اما صنعت سال‌هاست به `pytest` کوچ کرده: کم‌تشریفات‌تر، خواناتر و اکوسیستمِ غنی‌تر. نصبش:

```bash
pip install pytest
```

قرارداد ساده است: فایل‌های تست با `test_` شروع می‌شوند، توابعِ تست هم همین‌طور، و داخلشان `assert`ِ خالیِ خودِ پایتون می‌نویسید — نه متدهای خاص:

```python
# file: shipping.py — the code under test
def shipping_cost(weight_kg: float) -> int:
    if weight_kg <= 0:
        raise ValueError("weight must be positive")
    if weight_kg <= 1:
        return 30_000
    return 30_000 + int((weight_kg - 1) * 10_000)
```

```python
# file: test_shipping.py
from shipping import shipping_cost

def test_light_package_has_base_cost():
    assert shipping_cost(0.5) == 30_000

def test_heavy_package_costs_more():
    assert shipping_cost(3) == 50_000
```

اجرا:

```bash
pytest
# ================= 2 passed in 0.01s =================
```

همین. `pytest` خودش فایل‌ها و توابعِ تست را پیدا می‌کند (کشفِ خودکار)، اجرایشان می‌کند و اگر `assert`ی شکست بخورد، با جزئیاتِ کامل نشانتان می‌دهد — مقدارِ واقعی، مقدارِ انتظاری، و خطِ دقیق.

به نام‌گذاریِ تست‌ها دقت کنید: `test_light_package_has_base_cost` یک **جمله‌ی خبری** است. تستِ خوب اسمش رفتار را توصیف می‌کند؛ وقتی قرمز شود، از خودِ اسم می‌فهمید *چه قولی* زیرِ پا گذاشته شده.

---

## تست کلاس‌ها: الگوی سه‌مرحله‌ای AAA

برای تستِ کلاس‌ها یک الگوی ذهنیِ جهانی وجود دارد — **Arrange، Act، Assert** (آماده‌سازی، اقدام، ادعا):

```python
# file: bank.py — remember BankAccount from chapter 2? now with tests, finally.
class InsufficientBalanceError(Exception):
    pass

class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("deposit must be positive")
        self.balance += amount

    def withdraw(self, amount):
        if amount > self.balance:
            raise InsufficientBalanceError(f"balance is {self.balance}")
        self.balance -= amount
```

```python
# file: test_bank.py
import pytest
from bank import BankAccount, InsufficientBalanceError

def test_deposit_increases_balance():
    account = BankAccount("ali", balance=100)   # Arrange
    account.deposit(50)                          # Act
    assert account.balance == 150                # Assert

def test_new_account_starts_at_zero():
    account = BankAccount("sara")
    assert account.balance == 0
```

هر تست **یک رفتار** را می‌سنجد و مستقل از بقیه است — شیءِ تازه‌ی خودش را می‌سازد. این استقلال مقدس است: تستی که به تستِ قبلی وابسته باشد، با عوض‌شدنِ ترتیبِ اجرا می‌شکند و ساعت‌ها وقتتان را برای پیداکردنِ «باگی که وجود ندارد» می‌سوزاند. (حالا دلیلِ واقعیِ دشمنی فصل قبل با Singleton را می‌بینید: حالتِ سراسری، استقلالِ تست‌ها را می‌کُشد.)

### تست استثناها

رفتارِ خطا هم رفتار است و تست می‌خواهد — استثناهای سفارشی که در فصل یازدهم طراحی کردید، اینجا ارزششان را نشان می‌دهند:

```python
def test_withdraw_more_than_balance_fails():
    account = BankAccount("ali", balance=100)
    with pytest.raises(InsufficientBalanceError):
        account.withdraw(500)

def test_negative_deposit_is_rejected():
    account = BankAccount("ali")
    with pytest.raises(ValueError, match="positive"):
        account.deposit(-10)
```

`pytest.raises` یک context manager است (همان پروتکلِ `__enter__`/`__exit__` فصل پنجم!): اگر داخلِ بلاک، استثنای موردِانتظار رخ ندهد، تست شکست می‌خورد. پارامترِ `match` متنِ پیام را هم می‌سنجد.

### تست‌های پارامتری: یک تست، ده سناریو

وقتی همان منطق باید با ورودی‌های مختلف سنجیده شود، کپی‌کردنِ تست ممنوع — پارامتری کنید:

```python
import pytest
from shipping import shipping_cost

@pytest.mark.parametrize("weight, expected", [
    (0.5, 30_000),      # light
    (1,   30_000),      # exactly at the boundary
    (2,   40_000),      # heavy
    (10,  120_000),     # very heavy
])
def test_shipping_cost(weight, expected):
    assert shipping_cost(weight) == expected
```

pytest این را **چهار تستِ جدا** حساب می‌کند و اگر فقط یکی بشکند، دقیقاً می‌گوید کدام ورودی. به سطرِ دوم دقت کنید: مقدارِ مرزی (`weight == 1`). تجربه می‌گوید باگ‌ها عاشقِ مرزها هستند — صفر، یک، آخرین عضو، لیستِ خالی. تست‌های مرزی را هیچ‌وقت جا نیندازید.

---

## Fixture: آماده‌سازیِ مشترک، بدون تکرار

وقتی ده تست همه به «یک حسابِ پُر» نیاز دارند، آماده‌سازی را در یک **fixture** جمع می‌کنیم:

```python
import pytest
from bank import BankAccount

@pytest.fixture
def rich_account():
    """Any test that asks for this name gets a fresh account with 1000."""
    return BankAccount("ali", balance=1000)

def test_withdraw(rich_account):
    rich_account.withdraw(300)
    assert rich_account.balance == 700

def test_two_operations(rich_account):
    rich_account.deposit(500)
    rich_account.withdraw(200)
    assert rich_account.balance == 1300
```

سازوکارش را ببینید: تابعِ تست، fixture را با **نامِ پارامتر** صدا می‌زند و pytest خودش آن را می‌سازد و تزریق می‌کند — آشناست؟ این همان **تزریقِ وابستگی** فصل هشتم است که pytest به سطحِ فریمورک برده. و هر تست fixtureِ **تازه‌ی** خودش را می‌گیرد؛ استقلالِ تست‌ها حفظ می‌شود.

fixtureهای آماده‌ی خودِ pytest هم طلایی‌اند. مهم‌ترینشان `tmp_path` است: یک پوشه‌ی موقتِ واقعی که بعد از تست خودش تمیز می‌شود — برای تستِ کدهایی که فایل می‌نویسند (مثل ذخیره‌سازیِ JSON پروژه‌ی اول):

```python
import json

def save_users(users: list, path):
    path.write_text(json.dumps(users), encoding="utf-8")

def test_save_users_writes_valid_json(tmp_path):
    target = tmp_path / "users.json"        # a real temp file, auto-cleaned
    save_users([{"name": "ali"}], target)
    assert json.loads(target.read_text(encoding="utf-8")) == [{"name": "ali"}]
```

---

## جداسازی از دنیای بیرون: Fake و Mock

تا اینجا کلاس‌هایی را تست کردیم که «خودکفا» بودند. اما کلاسِ واقعی به دنیا وصل است: دیتابیس، درگاهِ پرداخت، سرویسِ ایمیل، ساعتِ سیستم. تستی که واقعاً ایمیل بفرستد یا کارت بکشد، نه سریع است، نه تکرارپذیر، نه بی‌خطر.

راه‌حل را از فصل هفتم و هشتم دارید: **وابستگی را تزریق کن.** حالا میوه‌اش را می‌چینیم.

### Fake: بدلِ دست‌ساز

```python
# order_service.py
class OrderService:
    def __init__(self, payment_gateway, notifier):
        self.gateway = payment_gateway        # injected (DIP, ch 8)
        self.notifier = notifier

    def checkout(self, order_id, amount, card):
        receipt = self.gateway.charge(card, amount)
        self.notifier.send(f"order {order_id} paid: {receipt}")
        return receipt
```

```python
# test_order_service.py — hand-made fakes, no library needed
class FakeGateway:
    def __init__(self):
        self.charged = []                     # records what happened

    def charge(self, card, amount):
        self.charged.append((card, amount))
        return f"receipt-{len(self.charged)}"

class FakeNotifier:
    def __init__(self):
        self.sent = []

    def send(self, message):
        self.sent.append(message)

def test_checkout_charges_and_notifies():
    gateway, notifier = FakeGateway(), FakeNotifier()
    service = OrderService(gateway, notifier)

    receipt = service.checkout(1, 90_000, "1234")

    assert receipt == "receipt-1"
    assert gateway.charged == [("1234", 90_000)]
    assert notifier.sent == ["order 1 paid: receipt-1"]
```

Fake یک پیاده‌سازیِ ساده‌ی واقعی است که رفتارش را ضبط می‌کند. به‌لطفِ Duck Typing حتی لازم نیست از کلاسی ارث ببرد — فقط باید متدِ درست را داشته باشد. تستِ بالا **بدونِ اینترنت، بدونِ بانک، در چند میلی‌ثانیه** کلِ منطقِ checkout را می‌سنجد.

### Mock: بدلِ آماده از کتابخانه‌ی استاندارد

برای بدل‌های یک‌بارمصرف، `unittest.mock` بدلِ آماده می‌دهد:

```python
from unittest.mock import Mock

def test_checkout_with_mocks():
    gateway = Mock()
    gateway.charge.return_value = "receipt-42"    # program the answer
    notifier = Mock()

    service = OrderService(gateway, notifier)
    receipt = service.checkout(7, 50_000, "9876")

    assert receipt == "receipt-42"
    gateway.charge.assert_called_once_with("9876", 50_000)
    notifier.send.assert_called_once_with("order 7 paid: receipt-42")
```

`Mock` هر attributeی که بخواهید در لحظه می‌سازد (با همان `__getattr__` جادوییِ فصل پنجم)، هر فراخوانی را ضبط می‌کند، و متدهای `assert_called_*` برای بازجویی می‌دهد. با `side_effect` می‌شود خطا هم شبیه‌سازی کرد — سناریوهایی که با سرویسِ واقعی تقریباً غیرقابلِ‌ساخت‌اند:

```python
def test_checkout_when_gateway_is_down():
    gateway = Mock()
    gateway.charge.side_effect = ConnectionError("bank is down")
    service = OrderService(gateway, Mock())

    import pytest
    with pytest.raises(ConnectionError):
        service.checkout(1, 10_000, "1111")
```

### patch: وقتی تزریق ممکن نیست

گاهی کدی که تست می‌کنید وابستگی‌اش را تزریق نگرفته، بلکه مستقیم صدا می‌زند (مثلاً `datetime.now()` یا `requests.get`). `patch` موقتاً آن نام را در محلِ استفاده‌اش عوض می‌کند:

```python
from unittest.mock import patch

# code under test (no injection — the hard-to-test style):
def greeting():
    from datetime import datetime
    hour = datetime.now().hour
    return "good morning" if hour < 12 else "good evening"

def test_greeting_in_the_morning():
    with patch("datetime.datetime") as fake_dt:
        fake_dt.now.return_value.hour = 9
        assert greeting() == "good morning"
```

اما یک اعترافِ حرفه‌ای: **هر بار که مجبور به `patch` می‌شوید، طراحی دارد به شما طعنه می‌زند.** کدِ خوش‌طراحی، وابستگی‌هایش را تزریق می‌گیرد و به `patch` نیاز پیدا نمی‌کند. در پروژه‌ی سومِ دوره همین را دیدید: `BookingService` تابعِ زمان (`now_fn`) را تزریق می‌گرفت و تستِ انقضای رزرو، بدونِ هیچ patchی، فقط با پاس‌دادنِ زمانِ ساختگی انجام شد. قاعده‌ی سرانگشتی: **اول تزریق، بعد Fake، و patch فقط برای کدی که نمی‌توانید تغییرش دهید.**

---

## چه چیزی را تست کنیم؟ (و چه چیزی را نه)

هنرِ تست‌نویسی در انتخاب است. چند قاعده از جنسِ تجربه:

**رفتار را تست کنید، نه پیاده‌سازی را.** تستِ خوب می‌گوید «بعد از برداشتِ ۳۰۰، موجودی ۷۰۰ است»؛ تستِ بد می‌گوید «متدِ `_recalculate` باید دقیقاً دو بار صدا شود». اولی با ریفکتورِ داخلی سبز می‌ماند؛ دومی با هر جابه‌جاییِ بی‌ضرر می‌شکند و کم‌کم آن‌قدر مزاحم می‌شود که تیم تست‌ها را «خاموش» می‌کند. Mockِ افراطی دقیقاً همین بیماری است: وقتی تک‌تکِ فراخوانی‌های داخلی را `assert_called` می‌کنید، دیگر رفتار را تست نمی‌کنید؛ کپیِ کد را تست می‌کنید.

**منطق را تست کنید، نه زبان را.** تستی که فقط می‌سنجد «dataclass فیلدهایش را ست می‌کند» یا «property مقدار برمی‌گرداند»، دارد خودِ پایتون را تست می‌کند. وقتِ محدودتان را بگذارید روی منطقِ شرطی، محاسبات، مرزها و خطاها.

**اولویت با پرریسک‌هاست.** منطقِ پول و پرداخت، قوانینِ کسب‌وکار، هر جایی که باگش خبرساز می‌شود. جمله‌ی معروفی در صنعت هست: پوششِ تستِ ۱۰۰٪ نه ممکن است برای همه، نه هدف؛ **اطمینانِ ۱۰۰٪ به بخش‌های حیاتی** هدف است.

برای دیدنِ اینکه چقدر از کدتان زیرِ تور است:

```bash
pip install pytest-cov
pytest --cov=myproject
# TOTAL ...... 87%
```

عدد را راهنما بگیرید، نه بت. کدی با پوششِ ۱۰۰٪ که فقط happy path را می‌سنجد، از کدی با ۷۰٪ که مرزها و خطاها را می‌سنجد ضعیف‌تر است.

---

## تست‌پذیری: آزمونِ نهاییِ طراحی

این فصل را با یک جمع‌بندیِ مهم ببندیم که کلِ نیمه‌ی دومِ دوره را به هم می‌دوزد:

**تست‌پذیری، لَخته‌سنجِ طراحی است.** اگر برای تستِ یک کلاس مجبورید دیتابیسِ واقعی بالا بیاورید، پنج شیءِ دیگر بسازید و سه چیز را patch کنید — تست‌ها دارند فریاد می‌زنند که طراحی مشکل دارد: وابستگیِ پنهان (نقضِ DIP)، مسئولیتِ زیاد (نقضِ SRP)، جفت‌شدگیِ سفت (فقدانِ Composition). برعکس، کلاسی که در سه خط تست می‌شود، تقریباً همیشه خوش‌طراحی است.

برای همین تیم‌های حرفه‌ای تست را همزمان با کد (یا حتی قبلش — TDD) می‌نویسند: نه فقط برای گرفتنِ باگ، بلکه چون تستِ سخت‌نویس، **زودترین هشدارِ طراحیِ بد** است. ارزان‌ترین زمان برای شنیدنِ این هشدار، قبل از production است.

---

## خلاصه

| مفهوم | یک‌خطی |
|-------|---------|
| pytest | فریمورکِ استانداردِ صنعت: کشفِ خودکار، `assert` ساده |
| AAA | ساختارِ هر تست: Arrange، Act، Assert |
| `pytest.raises` | تستِ رفتارِ خطا و استثناهای سفارشی |
| `parametrize` | یک منطقِ تست، چند سناریو — مرزها را فراموش نکنید |
| fixture | آماده‌سازیِ مشترکِ قابلِ‌تزریق؛ هر تست نسخه‌ی تازه |
| Fake | بدلِ دست‌سازِ ساده که رفتار را ضبط می‌کند |
| Mock | بدلِ آماده‌ی `unittest.mock` با ضبط و بازجوییِ فراخوانی‌ها |
| patch | تعویضِ موقتِ وابستگیِ تزریق‌نشده — نشانه‌ی طعنه‌ی طراحی |
| رفتار نه پیاده‌سازی | تستِ شکننده = تستی که به جزئیاتِ داخلی چسبیده |

**نکته‌ی طلایی:** تست ننوشتن، صرفه‌جویی نیست؛ قرض‌گرفتن از آینده با بهره‌ی سنگین است. و اگر تستی سخت نوشته می‌شود، قبل از جنگیدن با تست، به طراحیِ کد مشکوک شوید.

---

## پرسش‌های مصاحبه

**۱. «برای کلاس BankAccount چه تست‌هایی می‌نویسید؟»** — پاسخِ ممتاز، دسته‌بندی دارد: happy path (واریز/برداشتِ عادی)، مرزها (برداشتِ دقیقاً برابرِ موجودی، واریزِ صفر)، خطاها (برداشتِ بیش از موجودی → استثنای سفارشی، مبلغِ منفی)، و حالتِ اولیه (حسابِ تازه). گفتنِ «مرزها و خطاها» شما را از کسانی که فقط happy path می‌گویند جدا می‌کند.

**۲. «فرق Mock و Fake (و Stub) چیست؟»** — Fake پیاده‌سازیِ ساده‌ی واقعی است (مثل gateway ای که در لیست ضبط می‌کند)، Stub فقط جوابِ ازپیش‌تعیین‌شده می‌دهد، Mock علاوه بر جواب، **فراخوانی‌ها را ضبط می‌کند و انتظارات را می‌سنجد** (`assert_called_once_with`). در عمل `unittest.mock.Mock` هر سه نقش را بازی می‌کند؛ مهم این است که بدانید کجا فقط جواب لازم دارید و کجا بازجوییِ تعامل.

**۳. «تست شکننده (brittle) یعنی چه و چطور از آن دوری می‌کنید؟»** — تستی که با ریفکتورِ بی‌ضررِ داخلی می‌شکند، چون به پیاده‌سازی (ترتیب/تعدادِ فراخوانی‌های داخلی، ساختارِ private) چسبیده نه به رفتارِ قابلِ‌مشاهده. دوری: تستِ خروجی و تغییرِ حالتِ عمومی، Mock فقط در مرزِ سیستم (I/O، شبکه)، نه برای اشیای داخلیِ خودتان.

**۴. «چرا تزریق وابستگی تست را آسان می‌کند؟ یک مثال بزنید.»** — چون نقطه‌ی تعویضِ دنیای واقعی با بدل را از قبل در سازنده گذاشته: `OrderService(gateway, notifier)` در تست همان امضا را با Fake می‌گیرد، بدونِ patch. مثالِ ساعت هم طلایی است: تزریقِ `now_fn` به‌جای فراخوانیِ مستقیمِ `datetime.now()`، تستِ منطقِ انقضا را از «انتظارِ واقعی» به «پاس‌دادنِ زمانِ ساختگی» تبدیل می‌کند.

---

**در فصل بعد:** از دنیای طراحی و کیفیت، به زیرِ سطح می‌رویم: **مدیریت حافظه** — اشیاء کجا زندگی می‌کنند، چه کسی می‌میراندشان، `__slots__` چطور میلیون‌ها شیء را سبک می‌کند و weakref چطور جلوی نشتِ حافظه را می‌گیرد. از اینجا به بعد، وارد عمقِ پایتون می‌شویم.
