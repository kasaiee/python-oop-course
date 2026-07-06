# پاسخ تمرین‌های فصل سیزدهم: تست‌نویسی برای کد شیءگرا

---

## بخش الف: پرسش‌های مفهومی

**۱.** فلجِ ریفکتور یعنی ترس از تغییرِ کدِ کارکرده، چون هیچ راهی برای فهمیدنِ «چه چیزی شکست» جز امید و تستِ دستی نیست. تست‌ها مثل تورِ ایمنیِ بندباز عمل می‌کنند: با وجودشان، حرکتِ جسورانه (ریفکتور، افزودنِ قابلیت) ممکن می‌شود چون اگر چیزی بشکند، تور — یعنی تستِ قرمزشده — بلافاصله می‌گیردش؛ بی‌تور، عاقلانه‌ترین کار بی‌حرکتی است و کد به‌مرور فسیل می‌شود.

**۲.** **Arrange** صحنه را می‌چیند (ساختِ شیء و داده‌ی لازم)، **Act** دقیقاً یک رفتارِ موردِآزمایش را اجرا می‌کند، **Assert** ادعا را می‌سنجد (خروجی یا حالتِ جدید). این ساختار تست را خوانا می‌کند: هر تست یک داستانِ سه‌پرده‌ای کوتاه است.

**۳.** چون نتیجه‌ی هر تست باید فقط به خودش وابسته باشد؛ در غیرِ این صورت با تغییرِ ترتیبِ اجرا یا اجرای موازی، تست‌های سالم می‌شکنند و ساعت‌ها صرفِ تعقیبِ «باگِ موهوم» می‌شود. حالتِ سراسری (Singleton، متغیرِ ماژول، class variable دست‌کاری‌شده) بینِ تست‌ها باقی می‌ماند و تستِ بعدی روی صحنه‌ی به‌هم‌ریخته‌ی قبلی اجرا می‌شود — وابستگیِ پنهانی که استقلال را می‌کشد.

**۴.** **Fake** پیاده‌سازیِ ساده‌ی واقعی و دست‌ساز است (کلاس کوچکی که رفتار را در لیست ضبط می‌کند)؛ وقتی خوب است که همان بدل در چند تست به‌کار می‌رود یا رفتارِ کمی «واقعی» لازم است. **Mock** بدلِ آماده‌ی `unittest.mock` است که هر attribute را در لحظه می‌سازد و فراخوانی‌ها را با `assert_called_*` بازجویی می‌کند؛ برای بدلِ یک‌بارمصرف و شبیه‌سازیِ سریعِ خطا (`side_effect`) سریع‌تر است. اگر تعدادِ assertهای تعاملی زیاد شود، Fake معمولاً تستِ خواناتر و کم‌شکننده‌تری می‌دهد.

**۵.** یعنی نیاز به patch نشانه‌ی این است که کد وابستگی‌اش را سفت و مستقیم صدا می‌زند (مثل `datetime.now()` یا ساختِ اتصالِ دیتابیس در دلِ متد) و «نقطه‌ی تعویض» ندارد. قاعده: **اول تزریق** (سازنده وابستگی را بگیرد)، **بعد Fake** (بدل را از همان درِ تزریق بده)، **و patch فقط برای کدی که نمی‌توانید تغییرش دهید** (کتابخانه‌ی بیرونی، کدِ legacy). patch ابزارِ مشروعی است، اما اگر برای کدِ خودتان مدام لازمش دارید، مشکل از طراحی است نه از تست.

**۶.** تستِ رفتار: «بعد از `withdraw(300)`، موجودی ۷۰۰ است» — به قراردادِ عمومیِ کلاس چسبیده. تستِ پیاده‌سازی: «متدِ داخلیِ `_log` دقیقاً دو بار صدا شد» — به جزئیاتِ داخلی چسبیده. دومی شکننده است چون با هر ریفکتورِ بی‌ضرر (تغییرِ نامِ متدِ خصوصی، ادغامِ دو فراخوانی) قرمز می‌شود در حالی که رفتارِ کلاس ذره‌ای عوض نشده؛ تیم کم‌کم به قرمزیِ تست‌ها بی‌اعتنا می‌شود و تور سوراخ می‌شود.

---

## بخش ب: خواندن و تحلیل تست

**۷.**

- `test_a`: **پاس** — ۱۰۰ منهای ۴۰ می‌شود ۶۰.
- `test_b`: **شکست** — `spend(100)` با موجودیِ ۱۰۰ مجاز است (شرط `amount > self.balance` است، نه `>=`)؛ استثنایی رخ نمی‌دهد و `pytest.raises` به همین دلیل تست را شکست می‌دهد. (نکته‌ی مرزی!)
- `test_c`: **پاس** — موجودیِ پیش‌فرض صفر است و `spend(1)` استثنا می‌دهد؛ دقیقاً همان که انتظار داریم.

**۸.** دو assert آخر تست را به پیاده‌سازی چسبانده‌اند: `assert db.fetch.call_args is not None` عملاً تکرارِ `assert_called_once` و بی‌ارزش است، و `db.connect.assert_not_called()` دارد **نبودنِ** یک فراخوانیِ داخلیِ دلخواه را تحمیل می‌کند — اگر فردا `ReportService` تصمیم بگیرد قبل از fetch یک connect بزند (تغییرِ داخلیِ بی‌ضرر)، تست می‌شکند بی‌آنکه رفتارِ قابلِ‌مشاهده عوض شده باشد. تستِ سالم همان دو خطِ اول است: خروجی درست باشد و مرزِ سیستم (fetch) یک‌بار صدا شده باشد.

**۹.** حالتِ مشترکِ سطحِ ماژول (`inventory`) بینِ تست‌ها نشت می‌کند: `test_reserve_decreases_stock` دیکشنری را تغییر می‌دهد و اگر قبل از `test_stock_is_positive` اجرا شود، آن را می‌شکند. اصلاح: هر تست داده‌ی تازه‌ی خودش را بسازد — یا مستقیم داخلِ تست، یا با fixture:

```python
import pytest

@pytest.fixture
def inventory():
    return {"keyboard": 5}      # a fresh dict for every test

def test_reserve_decreases_stock(inventory):
    inventory["keyboard"] -= 2
    assert inventory["keyboard"] == 3

def test_stock_is_positive(inventory):
    assert inventory["keyboard"] == 5
```

---

## بخش ج: کدنویسی

**۱۰.**

```python
import pytest
from cart import Cart, DuplicateItemError

def test_add_item():
    cart = Cart()
    cart.add("keyboard", 500_000)
    assert cart.total() == 500_000

def test_total_sums_prices():
    cart = Cart()
    cart.add("keyboard", 500_000)
    cart.add("mouse", 300_000)
    assert cart.total() == 800_000

def test_duplicate_item_is_rejected():
    cart = Cart()
    cart.add("keyboard", 500_000)
    with pytest.raises(DuplicateItemError):
        cart.add("keyboard", 400_000)

def test_empty_cart_totals_zero():
    assert Cart().total() == 0
```

**۱۱.**

```python
import pytest

def grade(score):
    if score >= 90: return "A"
    if score >= 75: return "B"
    if score >= 50: return "C"
    return "F"

@pytest.mark.parametrize("score, expected", [
    (100, "A"),
    (90,  "A"),     # boundary
    (89,  "B"),     # just below the boundary
    (75,  "B"),     # boundary
    (50,  "C"),     # boundary
    (49,  "F"),     # just below
    (0,   "F"),
])
def test_grade(score, expected):
    assert grade(score) == expected
```

مرزها (۹۰، ۷۵، ۵۰ و همسایه‌هایشان) مهم‌ترین سطرهای این جدول‌اند — باگِ کلاسیکِ `>` به‌جای `>=` فقط با همین‌ها گیر می‌افتد.

**۱۲.**

```python
import pytest

@pytest.fixture
def loaded_cart():
    cart = Cart()
    cart.add("keyboard", 50_000)
    cart.add("mouse", 30_000)
    return cart

def test_loaded_cart_total(loaded_cart):
    assert loaded_cart.total() == 80_000

def test_adding_to_loaded_cart(loaded_cart):
    loaded_cart.add("pad", 20_000)
    assert loaded_cart.total() == 100_000
```

**۱۳.**

```python
import pytest
from unittest.mock import Mock

# --- (a) hand-made fake ---
class FakeSender:
    def __init__(self):
        self.sent = []

    def send(self, phone, text):
        self.sent.append((phone, text))

def test_notify_all_sends_to_everyone():
    sender = FakeSender()
    service = CampaignService(sender)

    count = service.notify_all(["0911", "0912", "0913"], "sale!")

    assert count == 3
    assert [p for p, _ in sender.sent] == ["0911", "0912", "0913"]

# --- (b) mock with a failure in the middle ---
def test_notify_all_skips_failed_numbers():
    sender = Mock()
    sender.send.side_effect = [None, ConnectionError("down"), None]
    service = CampaignService(sender)

    count = service.notify_all(["0911", "0912", "0913"], "sale!")

    assert count == 2                      # one failed, two succeeded
    assert sender.send.call_count == 3     # but all three were attempted
```

نکته‌ی `side_effect` با لیست: هر فراخوانی، عنصرِ بعدی را برمی‌دارد — اگر استثنا باشد، پرتاب می‌شود؛ اگر مقدار باشد، برگردانده می‌شود.

---

## بخش د: طراحی برای تست‌پذیری

**۱۴.** سه مشکل:

- **وابستگیِ سفت و درون‌ساخت:** اتصالِ دیتابیس و SMTP داخلِ متد ساخته می‌شوند (با رمز عبورِ hard-code!) — هیچ نقطه‌ی تعویضی برای بدل نیست؛ تست یعنی اتصالِ واقعی به سرورِ production.
- **زمانِ تزریق‌نشده:** `datetime.now()` مستقیم صدا می‌شود؛ تستِ «۳۰ روز گذشته» بدونِ patch ممکن نیست.
- **چند مسئولیت در یک متد:** واکشیِ داده + منطقِ تشخیصِ سررسید + ارسالِ ایمیل، همه در هم — منطقِ خالص (که ارزانْ تست می‌شود) از I/O جدا نیست.

نسخه‌ی اصلاح‌شده (اسکلت):

```python
class InvoiceService:
    def __init__(self, db, mailer, now_fn=None):
        from datetime import datetime
        self.db = db                     # injected
        self.mailer = mailer             # injected
        self.now_fn = now_fn or datetime.now

    def find_overdue(self, invoices, days=30):
        """Pure logic - cheap to test with plain data."""
        now = self.now_fn()
        return [inv for inv in invoices
                if (now - inv.due_date).days > days]

    def send_overdue_notices(self):
        invoices = self.db.fetch_invoices()
        for inv in self.find_overdue(invoices):
            self.mailer.send(inv.email, "your invoice is overdue!")
```

حالا `find_overdue` با داده‌ی ساده و `now_fn` ساختگی تست می‌شود، و `send_overdue_notices` با یک Fake برای `db` و `mailer` — بدونِ سرور، بدونِ patch، بدونِ رمزِ لورفته.

---

## بخش ه: پرسشِ تحلیلی

**۱۵.** سیاستِ «۱۰۰٪ یا هیچ» با نیتِ خوب، رفتارهای بد تولید می‌کند. اول، **تستِ نمایشی**: برای رسیدن به عدد، تست‌هایی نوشته می‌شود که فقط خط را «لمس» می‌کنند بی‌آنکه ادعایی بسنجند (فراخوانی بدونِ assert معنادار) — پوشش بالا می‌رود، اطمینان نه. دوم، **گرایش به happy path**: پوشاندنِ خطوطِ آسان به‌جای سناریوهای خطا و مرزی که واقعاً باگ‌خیزند. سوم، **مالیات بر کدِ کم‌ریسک**: وقتِ تیم صرفِ تست‌کردنِ کدِ بدیهی (getter، dataclass) می‌شود و از منطقِ حیاتی کم می‌آید؛ ضمنِ اینکه تست‌های کم‌ارزش هزینه‌ی نگهداری دارند و با هر ریفکتور باید به‌روز شوند. جایگزینِ سالم‌تر: آستانه‌ی معقول (مثلاً ۸۰-۹۰٪) به‌عنوانِ **راهنما نه دروازه**، همراه با دو قانونِ کیفی — منطقِ پرریسک (پول، قوانینِ کسب‌وکار، امنیت) باید تستِ مرزی و خطا داشته باشد، و هر باگِ production قبل از رفع، اول یک تستِ بازتولیدکننده بگیرد. هدف «اطمینانِ بالا به بخش‌های حیاتی» است؛ عددِ پوشش فقط سایه‌ی ناقصی از آن است.
