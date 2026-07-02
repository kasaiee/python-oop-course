# فصل شانزدهم — پروژه اول: سیستم مدیریت کتابخانه

> این فصل از نوعِ **آموزش با مثال** است. برخلافِ فصل‌های مفهومی، اینجا یک پروژه‌ی واقعی را از صفر می‌سازیم: از شنیدنِ نیازِ مبهمِ کارفرما تا پیاده‌سازیِ مرحله‌به‌مرحله.

---

## مقدمه

پانزده فصل مفهوم یاد گرفتیم. حالا وقتِ ساختن است.

در این پروژه یک **سیستم مدیریت کتابخانه** می‌سازیم. اما مهم‌تر از خودِ کد، **فرایند** است: چطور نیازِ گنگِ کارفرما را بشنویم، سوالِ درست بپرسیم، نیازمندی‌ها را جمع‌بندی کنیم، و بعد قدم‌به‌قدم — از ساده‌ترین کد تا سیستمِ کامل — پیش برویم. این همان کاری است که در روزِ اولِ یک شغلِ واقعی از شما می‌خواهند.

---

## نیاز کارفرما

کارفرما — مدیرِ یک کتابخانه‌ی محلی — به شما مراجعه می‌کند و می‌گوید:

> «ما یک کتابخانه داریم و الان همه‌چیز را روی کاغذ ثبت می‌کنیم. می‌خواهم یک سیستمِ کامپیوتری داشته باشم که کتاب‌ها را مدیریت کند. اعضا بتوانند کتاب امانت بگیرند و پس بدهند. باید بدانم هر کتاب دستِ کیست. آها، و بعضی اعضا کتاب‌ها را دیر برمی‌گردانند؛ می‌خواهم برایشان جریمه در نظر بگیرید. همین دیگر، فکر کنم ساده باشد.»

این توصیف، مثلِ همه‌ی توصیف‌های کارفرما، **مبهم** است. «همین دیگر، ساده است» جمله‌ای است که هر برنامه‌نویسی باید نسبت به آن محتاط باشد. کارِ ما این است که این ابهام را با سوال روشن کنیم.

---

## تحلیل نیازمندی‌ها

خودتان را جای برنامه‌نویسی بگذارید که می‌خواهد این پروژه را انجام دهد. پیش از نوشتنِ کد، باید نقاطِ مبهم را با سوال روشن کنید. این سوال‌ها را از کارفرما می‌پرسیم:

**درباره‌ی کتاب‌ها:**
- «آیا از هر کتاب فقط یک نسخه دارید یا چند نسخه؟» → کارفرما: «از کتاب‌های پرطرفدار چند نسخه داریم.»
  - نتیجه‌ی مهم: باید بینِ «کتاب» (عنوان) و «نسخه» (جلدِ فیزیکی) تمایز بگذاریم — همان تحلیلی که در فصل دوم کردیم.

**درباره‌ی اعضا:**
- «آیا هر عضو محدودیتی در تعدادِ کتابِ امانتی دارد؟» → «بله، هر عضو حداکثر ۳ کتاب.»
- «آیا انواعِ مختلفِ عضویت دارید؟» → «فعلاً نه، همه یک‌جور.»

**درباره‌ی امانت و جریمه:**
- «مدتِ مجازِ امانت چند روز است؟» → «۱۴ روز.»
- «جریمه چطور حساب می‌شود؟» → «برای هر روزِ تأخیر، ۵ هزار تومان.»
- «اگر کتابی که عضو می‌خواهد در دسترس نباشد چه؟» → «باید به او بگویید در دسترس نیست.»

**درباره‌ی ذخیره‌سازی:**
- «آیا داده‌ها باید ذخیره شوند و بعد از بستنِ برنامه بمانند؟» → «بله، حتماً باید ذخیره شوند.»

این گفت‌وگو، مهارتِ کلیدیِ بازارِ کار است: **تبدیلِ خواسته‌ی مبهم به مشخصاتِ دقیق** از طریقِ سوالِ درست.

---

## جمع‌بندی نیازمندی‌ها

حالا نیازمندی‌ها را به‌شکلِ ساختارمند جمع می‌کنیم تا نقشه‌ی راهِ پیاده‌سازی باشد:

**موجودیت‌ها (اسم‌ها):**

```
Library (کتابخانه — هماهنگ‌کننده)
├── Book (کتاب: عنوان، نویسنده، شابک)
│     └── Copy (نسخه: شناسه، وضعیت)
├── Member (عضو: نام، شناسه، فهرستِ امانت‌ها)
└── Loan (امانت: نسخه، عضو، تاریخِ امانت، تاریخِ بازگشت)
```

**قوانینِ کسب‌وکار:**

| قانون | مقدار |
|-------|-------|
| سقفِ امانتِ همزمانِ هر عضو | ۳ کتاب |
| مدتِ مجازِ امانت | ۱۴ روز |
| جریمه‌ی هر روزِ تأخیر | ۵٬۰۰۰ تومان |
| امانتِ نسخه‌ی در دسترس‌نبودن | ممنوع (خطا) |

**نیازهای فنی:**
- ذخیره‌سازیِ دائمی (به JSON)
- مدیریتِ خطاهای معنادار (نسخه‌ی ناموجود، سقفِ امانت)

---

## پیش‌نیازهای دانشی

برای این پروژه، این فصل‌ها را باید بلد باشید:

- **[فصل ۲: تفکر شیءگرا](02-oo-thinking.md)** — استخراجِ کلاس‌ها از نیازمندی، و همین مثالِ کتابخانه در سطحِ پایه.
- **[فصل ۳: مفاهیم بنیادین](03-fundamentals-class-object-self-constructor.md)** — کلاس، شیء، `self`، سازنده، class/instance variable.
- **[فصل ۴: اصول چهارگانه](04-four-pillars.md)** — کپسول‌سازی، ارث‌بری (برای استثناها)، چندریختی.
- **[فصل ۷: Composition و معماری](07-composition-architecture.md)** — لایه‌بندی و رابطه‌ی بینِ کلاس‌ها.
- **[فصل ۱۱: مدل‌های داده مدرن](11-modern-data-models.md)** — `dataclass` و `Enum` برای مدل‌ها.
- **[فصل ۱۲: استثناها و سریال‌سازی](12-exceptions-serialization-introspection.md)** — استثناهای سفارشی و ذخیره در JSON.

---

## پیاده‌سازی

روشِ ما: از ساده‌ترین کدِ ممکن شروع می‌کنیم و هر بار یک قابلیت اضافه می‌کنیم. هیچ‌وقت همه‌چیز را یک‌جا نمی‌نویسیم.

### گام ۱: ساده‌ترین شروع — کتاب و نسخه

اول فقط مدلِ کتاب و نسخه. از `Enum` (فصل ۱۱) برای وضعیتِ نسخه استفاده می‌کنیم:

```python
from enum import Enum

class CopyStatus(Enum):
    AVAILABLE = "available"
    BORROWED = "borrowed"

class Book:
    def __init__(self, title, author, isbn):
        self.title = title
        self.author = author
        self.isbn = isbn
        self.copies = []          # فهرستِ نسخه‌های این کتاب

class Copy:
    def __init__(self, copy_id, book):
        self.copy_id = copy_id
        self.book = book
        self.status = CopyStatus.AVAILABLE
```

در این گام، فقط ساختارِ داده را داریم. `Book` عنوان و نویسنده را نگه می‌دارد و فهرستی از `Copy`ها. هر `Copy` وضعیتش را با `CopyStatus` نگه می‌دارد — به‌جای رشته‌ی خام، که در فصل ۱۱ دیدیم چرا بهتر است.

### گام ۲: افزودن عضو و استثناهای سفارشی

حالا عضو را اضافه می‌کنیم و سلسله‌مراتبِ استثنا (فصل ۱۲) را می‌سازیم:

```python
# --- استثناهای سفارشی (فصل ۱۲) ---
class LibraryError(Exception):
    """پایه‌ی همه‌ی خطاهای کتابخانه"""

class CopyNotAvailableError(LibraryError):
    """نسخه در دسترس نیست"""

class BorrowLimitError(LibraryError):
    """سقفِ امانتِ عضو پر است"""

# --- عضو ---
class Member:
    MAX_BOOKS = 3                  # class variable — قانونِ مشترک (فصل ۳)

    def __init__(self, member_id, name):
        self.member_id = member_id
        self.name = name
        self.borrowed_copies = []  # instance variable — مختصِ هر عضو

    def can_borrow(self):
        return len(self.borrowed_copies) < self.MAX_BOOKS
```

استثناها را سلسله‌مراتبی ساختیم تا بعداً بتوانیم با `except LibraryError` همه را یک‌جا بگیریم. `MAX_BOOKS` را class variable گذاشتیم چون قانونی مشترک بینِ همه‌ی اعضاست، و `borrowed_copies` را instance variable چون مختصِ هر عضو است — دقیقاً تمایزِ فصل ۳.

### گام ۳: امانت — قلبِ منطق

حالا `Loan` و منطقِ امانت را می‌سازیم. از `dataclass` (فصل ۱۱) برای `Loan` استفاده می‌کنیم چون عمدتاً حاملِ داده است:

```python
from dataclasses import dataclass, field
from datetime import date, timedelta

@dataclass
class Loan:
    copy: Copy
    member: Member
    borrow_date: date = field(default_factory=date.today)
    loan_days: int = 14
    return_date: date = None

    @property
    def due_date(self):                          # فقط‌خواندنی، محاسبه‌شونده (فصل ۵)
        return self.borrow_date + timedelta(days=self.loan_days)

    def days_overdue(self, on_date=None):
        on_date = on_date or date.today()
        end = self.return_date or on_date
        overdue = (end - self.due_date).days
        return max(0, overdue)                    # منفی نشود

    def fine(self, daily_rate=5000):
        return self.days_overdue() * daily_rate
```

از `default_factory=date.today` استفاده کردیم (نه `default`) — یادِ دامِ پیش‌فرضِ تغییرپذیر از فصل ۱۰. `due_date` یک `@property`ِ فقط‌خواندنی است که همیشه از `borrow_date` محاسبه می‌شود. `days_overdue` و `fine` منطقِ جریمه را نزدیکِ داده‌ی خودشان نگه می‌دارند — همان اصلِ «کلاسِ Rich» فصل ۷.

### گام ۴: کتابخانه — هماهنگ‌کننده

حالا `Library` که همه را کنار هم می‌آورد. توجه کنید که `Library` فقط **هماهنگ می‌کند** و قوانین را از خودِ موجودیت‌ها می‌پرسد (فصل ۲):

```python
class Library:
    def __init__(self, name):
        self.name = name
        self.books = {}            # isbn → Book
        self.members = {}          # member_id → Member
        self.loans = []            # فهرستِ امانت‌های فعال

    def add_book(self, book, num_copies=1):
        self.books[book.isbn] = book
        for i in range(num_copies):
            copy_id = f"{book.isbn}-{i+1}"
            book.copies.append(Copy(copy_id, book))

    def add_member(self, member):
        self.members[member.member_id] = member

    def _find_available_copy(self, isbn):
        book = self.books.get(isbn)
        if not book:
            raise LibraryError(f"کتابی با شابک {isbn} یافت نشد")
        for copy in book.copies:
            if copy.status == CopyStatus.AVAILABLE:
                return copy
        return None

    def borrow(self, member_id, isbn):
        member = self.members[member_id]
        # قانونِ سقف را از خودِ عضو بپرس (Tell, Don't Ask — فصل ۷)
        if not member.can_borrow():
            raise BorrowLimitError(f"{member.name} به سقفِ {Member.MAX_BOOKS} کتاب رسیده")
        copy = self._find_available_copy(isbn)
        if copy is None:
            raise CopyNotAvailableError(f"نسخه‌ی در دسترسی از «{self.books[isbn].title}» نیست")
        # ثبتِ امانت
        copy.status = CopyStatus.BORROWED
        member.borrowed_copies.append(copy)
        loan = Loan(copy=copy, member=member)
        self.loans.append(loan)
        return loan

    def return_book(self, member_id, copy_id):
        member = self.members[member_id]
        loan = next((l for l in self.loans
                     if l.copy.copy_id == copy_id and l.member is member), None)
        if loan is None:
            raise LibraryError("چنین امانتی یافت نشد")
        loan.return_date = date.today()
        loan.copy.status = CopyStatus.AVAILABLE
        member.borrowed_copies.remove(loan.copy)
        self.loans.remove(loan)
        return loan.fine()          # مبلغِ جریمه (اگر تأخیر بوده)
```

بیایید این را امتحان کنیم:

```python
library = Library("کتابخانه‌ی مرکزی")
book = Book("پایتون روان", "لوسیانو رامالیو", "978-1")
library.add_book(book, num_copies=2)          # ۲ نسخه
member = Member("m1", "علی")
library.add_member(member)

loan = library.borrow("m1", "978-1")
print(f"امانت داده شد. مهلت: {loan.due_date}")
print(f"وضعیتِ نسخه: {loan.copy.status.value}")   # borrowed

fine = library.return_book("m1", loan.copy.copy_id)
print(f"بازگشت داده شد. جریمه: {fine} تومان")     # 0 (به‌موقع)
```

### گام ۵: تستِ سناریوهای خطا

بررسی می‌کنیم که استثناها درست کار می‌کنند:

```python
# سناریو: سقفِ امانت
member2 = Member("m2", "سارا")
library.add_member(member2)
big_book = Book("کتابِ تک‌نسخه", "نویسنده", "978-2")
library.add_book(big_book, num_copies=1)

library.borrow("m2", "978-2")
try:
    library.borrow("m2", "978-2")     # نسخه‌ی دوم وجود ندارد
except CopyNotAvailableError as e:
    print(f"خطا (مطابقِ انتظار): {e}")

# گرفتنِ همه‌ی خطاهای کتابخانه با کلاسِ پایه (فصل ۴ و ۱۲)
try:
    library.borrow("m2", "978-999")   # شابکِ ناموجود
except LibraryError as e:             # هر خطای کتابخانه را می‌گیرد
    print(f"خطای کتابخانه: {e}")
```

استفاده از `except LibraryError` نشان می‌دهد چرا سلسله‌مراتبِ استثنا ارزشمند است: یک `except` همه‌ی خطاهای پروژه را می‌گیرد.

### گام ۶: ذخیره‌سازی (سریال‌سازی به JSON)

آخرین نیاز: ماندگاریِ داده. از الگوی `to_dict`/`from_dict` (فصل ۱۲) استفاده می‌کنیم:

```python
import json

class Library:
    # ... متدهای قبلی ...

    def save(self, path):
        data = {
            "name": self.name,
            "books": [
                {"title": b.title, "author": b.author, "isbn": b.isbn,
                 "copies": [{"copy_id": c.copy_id, "status": c.status.value}
                            for c in b.copies]}
                for b in self.books.values()
            ],
            "members": [
                {"member_id": m.member_id, "name": m.name,
                 "borrowed": [c.copy_id for c in m.borrowed_copies]}
                for m in self.members.values()
            ],
        }
        with open(path, "w", encoding="utf-8") as f:
            json.dump(data, f, ensure_ascii=False, indent=2)

    @classmethod
    def load(cls, path):               # Factory برای بازسازی (فصل ۵)
        with open(path, encoding="utf-8") as f:
            data = json.load(f)
        library = cls(data["name"])
        for b in data["books"]:
            book = Book(b["title"], b["author"], b["isbn"])
            library.books[book.isbn] = book
            for c in b["copies"]:
                copy = Copy(c["copy_id"], book)
                copy.status = CopyStatus(c["status"])
                book.copies.append(copy)
        for m in data["members"]:
            member = Member(m["member_id"], m["name"])
            library.members[member.member_id] = member
        return library

# استفاده:
library.save("library.json")
restored = Library.load("library.json")
print(restored.name)               # کتابخانه‌ی مرکزی
```

`load` را `@classmethod` ساختیم چون یک **Factory** است (فصل ۵): یک راهِ جایگزین برای ساختِ `Library` — این‌بار از فایل.

---

## جمع‌بندی پروژه

چه چیزی ساختیم و کدام مفاهیم به کار رفت:

| بخش | مفهوم | فصل |
|-----|-------|-----|
| تمایزِ Book/Copy | استخراجِ کلاس، مسئولیت | ۲ |
| `MAX_BOOKS`، `borrowed_copies` | class/instance variable | ۳ |
| `CopyStatus`, وضعیت‌ها | `Enum` | ۱۱ |
| `Loan` | `dataclass`, `default_factory` | ۱۰، ۱۱ |
| `due_date`, `fine` | `@property`, کلاسِ Rich | ۵، ۷ |
| `LibraryError` و فرزندان | استثنای سفارشی، سلسله‌مراتب | ۴، ۱۲ |
| `Library` هماهنگ‌کننده | Tell Don't Ask، لایه‌بندی | ۷ |
| `save`/`load` | سریال‌سازی JSON، Factory | ۵، ۱۲ |

**درسِ اصلی:** ما هرگز همه‌چیز را یک‌جا ننوشتیم. از یک مدلِ ساده شروع کردیم و شش گام، هر بار یک قابلیت افزودیم. این، روشِ واقعیِ ساختنِ نرم‌افزار است — نه نوشتنِ یک‌باره‌ی هزار خط، بلکه رشدِ تدریجی و آزموده.

**تمرینِ پیشنهادی برای شما:** سیستم را گسترش دهید تا انواعِ عضویت (عادی و ویژه، با سقف‌های متفاوت) را پشتیبانی کند. راهنمایی: از ارث‌بری (فصل ۴) یا بهتر، Composition و Strategy (فصل ۷) برای «سیاستِ امانت» استفاده کنید و Tradeoffشان را بسنجید.

---

**در پروژه‌ی بعد:** یک **سیستم پرداخت و صورت‌حساب آنلاین** می‌سازیم که تمرکزش بر چندریختی، الگوی Strategy، و اصولِ SOLID خواهد بود.
