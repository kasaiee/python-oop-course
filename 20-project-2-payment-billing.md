# فصل بیستم — پروژه دوم: سیستم پرداخت و صورت‌حساب آنلاین

> مقاله‌ی **آموزش با مثال**. این پروژه بر چندریختی، الگوی Strategy و اصول SOLID تمرکز دارد.

---

## مقدمه

در پروژه‌ی اول، یک سیستم داده‌محور (کتابخانه) ساختیم. حالا سراغ چالشی می‌رویم که در قلب آن **انعطاف‌پذیری** است: یک سیستم پرداخت که باید با روش‌های مختلف پرداخت کار کند و برای افزودن روش‌های تازه، **باز** باشد.

این دقیقاً همان مسئله‌ای است که در فصل‌های ۴، ۷ و ۸ بارها به آن اشاره کردیم. حالا آن را کامل و واقعی پیاده می‌کنیم و می‌بینیم چطور SOLID یک سیستم پرداخت حرفه‌ای می‌سازد.

---

## نیاز کارفرما

کارفرما صاحب یک فروشگاه اینترنتی است:

> «سایت فروشگاهی دارم و می‌خواهم مشتری‌ها بتوانند پرداخت کنند. الان فقط درگاه بانکی دارم، ولی می‌خواهم کیف پول داخلی هم اضافه کنم، و شاید بعداً پرداخت با رمزارز. برای هر خرید باید یک فاکتور صادر شود با جزئیات اقلام، مالیات و تخفیف. آها، بعضی مشتری‌ها کد تخفیف دارند. می‌خواهم سیستم طوری باشد که وقتی روش پرداخت جدیدی خواستم، کل کد را به‌هم نریزد.»

نکته‌ی طلایی این کارفرما جمله‌ی آخر است: «وقتی روش جدیدی خواستم، کل کد به‌هم نریزد.» این، تعریف دقیق اصل **Open/Closed** (فصل ۸) است — کارفرما بدون اینکه اسمش را بداند، دقیقاً همان را می‌خواهد.

---

## تحلیل نیازمندی‌ها

سوال‌هایی که از کارفرما می‌پرسیم:

**درباره‌ی روش‌های پرداخت:**
- «هر روش پرداخت چه اطلاعاتی لازم دارد؟» → «بانکی: شماره‌ی کارت. کیف پول: شناسه‌ی کاربر. رمزارز: آدرس کیف‌پول.»
- «آیا همه‌ی روش‌ها قابلیت بازگشت وجه (refund) دارند؟» → «بانکی و کیف‌پول بله، رمزارز فعلاً نه.»
  - نکته‌ی مهم: این یعنی نباید همه‌ی روش‌ها را وادار به `refund` کنیم — یاد اصل **Interface Segregation** (فصل ۸).

**درباره‌ی فاکتور:**
- «مالیات چند درصد است؟» → «۹ درصد.»
- «تخفیف چطور اعمال می‌شود؟» → «کد تخفیف یا درصدی است یا مبلغ ثابت.»
- «ترتیب اعمال: اول تخفیف بعد مالیات، یا برعکس؟» → «اول تخفیف روی جمع اقلام، بعد مالیات روی باقی‌مانده.»

**درباره‌ی خطاها:**
- «اگر پرداخت ناموفق بود؟» → «باید دلیلش مشخص باشد: موجودی ناکافی، کارت منقضی و... .»

---

## جمع‌بندی نیازمندی‌ها

**موجودیت‌ها:**

```
Invoice (فاکتور)
├── LineItem (قلم فاکتور: نام، قیمت واحد، تعداد)
├── Discount (تخفیف: درصدی یا ثابت)  ← Strategy
└── total() → با احتساب تخفیف و مالیات

PaymentMethod (انتزاع)  ← Strategy / DIP
├── BankPayment    (pay + refund)
├── WalletPayment  (pay + refund)
└── CryptoPayment  (فقط pay)

PaymentProcessor (هماهنگ‌کننده)
```

**قوانین کسب‌وکار:**

| قانون | مقدار |
|-------|-------|
| نرخ مالیات | ۹٪ |
| ترتیب محاسبه | (جمع اقلام − تخفیف) × (۱ + مالیات) |
| بازگشت وجه | فقط بانکی و کیف‌پول |

**اصول طراحی که هدف‌گذاری می‌کنیم:**
- **OCP:** افزودن روش پرداخت بدون تغییر کد موجود.
- **ISP:** جداکردن قابلیت `refund` از `pay`.
- **DIP:** وابستگی `PaymentProcessor` به انتزاع، نه پیاده‌سازی.

---

## پیش‌نیازهای دانشی

- **[فصل ۴: اصول چهارگانه](04-four-pillars.md)** — چندریختی، ارث‌بری، استثناهای سفارشی.
- **[فصل ۵: Property و متدهای جادویی](05-property-decorators-magic-methods.md)** — `@property`, `__str__`.
- **[فصل ۷: Composition و معماری](07-composition-architecture.md)** — الگوی Strategy، تزریق وابستگی.
- **[فصل ۸: SOLID](08-solid.md)** — هر پنج اصل، به‌ویژه OCP، ISP و DIP.
- **[فصل ۹: اینترفیس‌ها و پروتکل‌ها](09-interfaces-protocols-generics.md)** — ABC و Protocol برای تعریف رابط.
- **[فصل ۱۰: مدل‌های داده مدرن](10-modern-data-models.md)** — `dataclass`, `Enum`.

---

## پیاده‌سازی

### گام ۱: فاکتور و اقلام

از `dataclass` (فصل ۱۰) برای قلم فاکتور استفاده می‌کنیم:

```python
from dataclasses import dataclass, field

@dataclass
class LineItem:
    name: str
    unit_price: int
    quantity: int = 1

    @property
    def subtotal(self):                  # read-only (Ch. 5)
        return self.unit_price * self.quantity

@dataclass
class Invoice:
    items: list = field(default_factory=list)   # safe default (Ch. 14)
    tax_rate: float = 0.09

    def add_item(self, item):
        self.items.append(item)

    def subtotal(self):
        return sum(item.subtotal for item in self.items)
```

### گام ۲: تخفیف با الگوی Strategy

تخفیف دو نوع دارد: درصدی و ثابت. به‌جای `if/elif`، از **Strategy** (فصل ۷) استفاده می‌کنیم — هر نوع تخفیف یک کلاس. رابط را با ABC (فصل ۹) تعریف می‌کنیم:

```python
from abc import ABC, abstractmethod

class Discount(ABC):
    @abstractmethod
    def apply(self, amount: int) -> int:
        """returns the discount amount"""

class PercentageDiscount(Discount):
    def __init__(self, percent):
        self.percent = percent
    def apply(self, amount):
        return int(amount * self.percent / 100)

class FixedDiscount(Discount):
    def __init__(self, value):
        self.value = value
    def apply(self, amount):
        return min(self.value, amount)      # discount should not exceed the amount

class NoDiscount(Discount):
    def apply(self, amount):
        return 0                            # Null Object pattern
```

`NoDiscount` یک ترفند تمیز است (الگوی Null Object): به‌جای بررسی `if discount is None`، همیشه یک تخفیف داریم که صفر برمی‌گرداند. این کد را ساده‌تر می‌کند.

حالا `Invoice` را کامل می‌کنیم:

```python
@dataclass
class Invoice:
    items: list = field(default_factory=list)
    tax_rate: float = 0.09
    discount: Discount = field(default_factory=NoDiscount)

    def add_item(self, item):
        self.items.append(item)

    def subtotal(self):
        return sum(item.subtotal for item in self.items)

    def total(self):
        sub = self.subtotal()
        discounted = sub - self.discount.apply(sub)     # discount first
        return int(discounted * (1 + self.tax_rate))    # after tax

    def __str__(self):                                  # readable representation (Ch. 5)
        lines = [f"  {i.name}: {i.subtotal:,} ({i.quantity}×{i.unit_price:,})"
                 for i in self.items]
        return ("Invoice:\n" + "\n".join(lines) +
                f"\n  Subtotal: {self.subtotal():,}"
                f"\n  Payable: {self.total():,} Toman")

inv = Invoice(tax_rate=0.09, discount=PercentageDiscount(10))
inv.add_item(LineItem("Keyboard", 500_000))
inv.add_item(LineItem("mouse", 200_000, quantity=2))
print(inv)
print("Final amount:", inv.total())
```

توجه کنید که `total()` هیچ `if`ی برای نوع تخفیف ندارد؛ فقط `self.discount.apply(sub)` را صدا می‌زند و هر نوع تخفیفی به‌شکل خودش پاسخ می‌دهد — **چندریختی** (فصل ۴) در عمل.

### گام ۳: روش‌های پرداخت با Interface Segregation

اینجا اصل **ISP** (فصل ۸) را رعایت می‌کنیم: چون همه‌ی روش‌ها `refund` ندارند، آن را از `pay` جدا می‌کنیم. از دو رابط کوچک به‌جای یک رابط چاق استفاده می‌کنیم:

```python
from abc import ABC, abstractmethod

# payment exceptions (Ch. 4 and 11)
class PaymentError(Exception): pass
class InsufficientFundsError(PaymentError): pass
class CardExpiredError(PaymentError): pass

# small interface 1: payment
class Payable(ABC):
    @abstractmethod
    def pay(self, amount: int) -> str: ...

# small interface 2: refund (separate, per ISP)
class Refundable(ABC):
    @abstractmethod
    def refund(self, amount: int) -> str: ...

# bank: both pay and refund
class BankPayment(Payable, Refundable):
    def __init__(self, card_number, balance):
        self.card_number = card_number
        self.balance = balance
    def pay(self, amount):
        if amount > self.balance:
            raise InsufficientFundsError(f"balance {self.balance:,} is insufficient")
        self.balance -= amount
        return f"Card {self.card_number[-4:]}: {amount:,} paid"
    def refund(self, amount):
        self.balance += amount
        return f"Card {self.card_number[-4:]}: {amount:,} refunded"

# wallet: both pay and refund
class WalletPayment(Payable, Refundable):
    def __init__(self, user_id, balance):
        self.user_id = user_id
        self.balance = balance
    def pay(self, amount):
        if amount > self.balance:
            raise InsufficientFundsError("wallet balance is insufficient")
        self.balance -= amount
        return f"Wallet {self.user_id}: {amount:,} paid"
    def refund(self, amount):
        self.balance += amount
        return f"Wallet {self.user_id}: {amount:,} refunded"

# crypto: only pay (per ISP, not forced to refund)
class CryptoPayment(Payable):
    def __init__(self, wallet_address):
        self.wallet_address = wallet_address
    def pay(self, amount):
        return f"Crypto to {self.wallet_address[:6]}...: equivalent of {amount:,} paid"
```

نکته‌ی کلیدی: `CryptoPayment` فقط از `Payable` ارث می‌برد، نه `Refundable`. پس مجبور نیست `refund`ی که نمی‌تواند انجام دهد را با `NotImplementedError` پر کند — که هم ISP و هم LSP (فصل ۸) را نقض می‌کرد. رابط‌های کوچک، این را حل کردند.

### گام ۴: پردازشگر با Dependency Inversion

`PaymentProcessor` نباید به روش مشخصی وابسته باشد؛ به انتزاع `Payable` وابسته می‌شود (**DIP**، فصل ۸). روش واقعی از بیرون **تزریق** می‌شود:

```python
class PaymentProcessor:
    def pay_invoice(self, invoice: Invoice, method: Payable) -> str:
        amount = invoice.total()
        try:
            result = method.pay(amount)      # it doesn't know which method it is
            return f"✓ {result}"
        except PaymentError as e:            # all payment errors (Ch. 4)
            return f"✗ Payment failed: {e}"

    def refund_invoice(self, invoice: Invoice, method) -> str:
        # only if the method supports refunds
        if not isinstance(method, Refundable):
            return "✗ this payment method does not support refunds"
        return f"✓ {method.refund(invoice.total())}"
```

بررسی `isinstance(method, Refundable)` تنها جایی است که نوع را چک می‌کنیم — و آن هم برای تشخیص **قابلیت** است، نه نوع مشخص. این استفاده‌ی درست از `isinstance` است (فصل‌های ۹ و ۱۱).

### گام ۵: همه‌چیز کنار هم

```python
processor = PaymentProcessor()

inv = Invoice(discount=PercentageDiscount(10))
inv.add_item(LineItem("laptop", 30_000_000))
print(inv)
print()

# successful payment with a bank card
bank = BankPayment("6037991122334455", balance=50_000_000)
print(processor.pay_invoice(inv, bank))

# failed payment — insufficient balance
poor_wallet = WalletPayment("u1", balance=1_000_000)
print(processor.pay_invoice(inv, poor_wallet))

# crypto — works for payment
crypto = CryptoPayment("0xABCDEF123456")
print(processor.pay_invoice(inv, crypto))

# but crypto refunds are not possible
print(processor.refund_invoice(inv, crypto))
# bank refunds are possible
print(processor.refund_invoice(inv, bank))
```

### گام ۶: افزودن روش جدید — آزمون OCP

حالا لحظه‌ی حقیقت. کارفرما می‌گوید «پرداخت با کارت هدیه هم اضافه کن.» طبق OCP، **نباید هیچ کد موجودی را تغییر دهیم** — فقط یک کلاس تازه:

```python
class GiftCardPayment(Payable, Refundable):
    def __init__(self, code, balance):
        self.code = code
        self.balance = balance
    def pay(self, amount):
        if amount > self.balance:
            raise InsufficientFundsError("gift card balance is insufficient")
        self.balance -= amount
        return f"Gift card {self.code}: {amount:,} paid"
    def refund(self, amount):
        self.balance += amount
        return f"Gift card {self.code}: {amount:,} refunded"

# without any change to Invoice, PaymentProcessor, or the other methods:
gift = GiftCardPayment("GIFT-2026", balance=40_000_000)
print(processor.pay_invoice(inv, gift))       # it works!
```

`PaymentProcessor`، `Invoice` و بقیه‌ی کد **دست‌نخورده** ماندند. این ثمره‌ی مستقیم OCP و DIP است: سیستم برای توسعه باز، و برای تغییر بسته.

---

## جمع‌بندی پروژه

| بخش | مفهوم | فصل |
|-----|-------|-----|
| انواع تخفیف | الگوی Strategy، ABC | ۷، ۹ |
| `NoDiscount` | الگوی Null Object | ۷ |
| `total()` بدون if | چندریختی | ۴ |
| جدایی `Payable`/`Refundable` | Interface Segregation | ۸ |
| `CryptoPayment` بدون refund | ISP + LSP | ۸ |
| تزریق `method` | Dependency Inversion | ۷، ۸ |
| افزودن `GiftCardPayment` | Open/Closed | ۸ |
| استثناهای پرداخت | سلسله‌مراتب خطا | ۴، ۱۱ |

**درس اصلی:** خواسته‌ی کارفرما («کد به‌هم نریزد») مستقیماً به اصول SOLID ترجمه شد. با رعایت OCP، ISP و DIP، سیستمی ساختیم که افزودن قابلیت تازه در آن یعنی **نوشتن کد جدید**، نه **دستکاری کد قدیم**. این تفاوت یک سیستم حرفه‌ای با یک اسکریپت شکننده است.

**تمرین پیشنهادی:** یک سیستم «پرداخت ترکیبی» اضافه کنید که مبلغ را بین دو روش (مثلاً بخشی کیف‌پول، بخشی کارت) تقسیم کند. راهنمایی: از Composition استفاده کنید — یک `SplitPayment` که چند `Payable` را در خود نگه می‌دارد.

---

**در پروژه‌ی بعد و پایانی:** یک **سیستم رزرو آنلاین** می‌سازیم که همه‌ی مفاهیم — از مدیریت حالت و همزمانی تا معماری لایه‌ای — را کنار هم می‌آورد.
