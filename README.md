# lab2-oop-python
"""
Лабораторна робота №2: Проектування та реалізація об'єктно-орієнтованих систем в Python
Виконано студенткою групи КБ-25: Григор'євська
"""

from abc import ABC, abstractmethod


# =====================================================================
# ЗАВДАННЯ 1: Класова ієрархія з наслідуванням та поліморфізмом
# Предметна область: Бібліотечна система (медіа-ресурси)
# =====================================================================

class LibraryItem(ABC):
    """Базовий клас для елементів бібліотеки."""
    def __init__(self, title: str, year: int):
        self._title = title
        self._year = year

    def get_info(self) -> str:
        """Поліморфний метод, який перевизначається у підкласах."""
        return f"Назва: '{self._title}', Рік: {self._year}"

    def __str__(self) -> str:
        return self.get_info()


class Book(LibraryItem):
    """Клас Книга (підклас)."""
    def __init__(self, title: str, year: int, author: str, pages: int):
        super().__init__(title, year)
        self.author = author
        self.pages = pages

    def get_info(self) -> str:
        return f"[Книга] Автор: {self.author}, '{self._title}' ({self._year}), сторінок: {self.pages}"


class Magazine(LibraryItem):
    """Клас Журнал (підклас)."""
    def __init__(self, title: str, year: int, issue_number: int):
        super().__init__(title, year)
        self.issue_number = issue_number

    def get_info(self) -> str:
        return f"[Журнал] Видання: '{self._title}', Випуск №{self._issue_number_formatted()}, Рік: {self._year}"

    def _issue_number_formatted(self) -> str:
        return f"{self.issue_number:02d}"


class EBook(LibraryItem):
    """Клас Електронна книга (підклас)."""
    def __init__(self, title: str, year: int, file_size_mb: float, format_type: str):
        super().__init__(title, year)
        self.file_size_mb = file_size_mb
        self.format_type = format_type.upper()

    def get_info(self) -> str:
        return f"[E-Book] '{self._title}' ({self._year}) | Формат: {self.format_type}, Розмір: {self.file_size_mb} MB"


def demo_task_1():
    print("=" * 70)
    print("ЗАВДАННЯ 1: Класова ієрархія та поліморфізм")
    print("=" * 70)
    
    items = [
        Book("Алгоритми та структури даних", 2023, "І. Петренко", 450),
        Magazine("Tech Insider", 2026, 4),
        EBook("Python ООП Архітектура", 2024, 15.4, "pdf")
    ]
    
    print("Поліморфний обхід колекції елементів бібліотеки:")
    for item in items:
        print(f"  -> {item.get_info()}")
        
    # Перевірка типів через isinstance та issubclass
    print(f"\nЧи є Book підкласом LibraryItem? {issubclass(Book, LibraryItem)}")
    print(f"Чи є об'єкт Magazine екземпляром LibraryItem? {isinstance(items[1], LibraryItem)}")
    print(f"Чи є об'єкт Book екземпляром Magazine? {isinstance(items[0], Magazine)}")
    print()


# =====================================================================
# ЗАВДАННЯ 2: Інкапсуляція, property та контроль доступу
# Предметна область: Банківський рахунок
# =====================================================================

class BankAccount:
    """Клас, що моделює банківський рахунок із захищеними та приватними даними."""
    def __init__(self, owner: str, initial_balance: float):
        self.owner = owner                  # Публічний атрибут
        self._status = "Active"             # Захищений атрибут (convention)
        self.__balance = 0.0                # Приватний атрибут (name mangling)
        self.deposit(initial_balance)

    @property
    def balance(self) -> float:
        """Getter для приватного балансу."""
        return self.__balance

    @balance.setter
    def balance(self, value: float):
        """Setter із суворою валідацією вхідних значень."""
        if not isinstance(value, (int, float)):
            raise TypeError("Баланс повинен бути числовим значенням (int або float).")
        if value < 0:
            raise ValueError("Баланс не може від'ємним значенням.")
        self.__balance = float(value)

    @property
    def account_summary(self) -> str:
        """Обчислювана властивість (read-only @property)."""
        return f"Власник: {self.owner} | Статус: {self._status} | Баланс: {self.__balance:.2f} грн"

    def deposit(self, amount: float):
        if amount <= 0:
            raise ValueError("Сума поповнення має бути більше нуля.")
        self.balance = self.__balance + amount


def demo_task_2():
    print("=" * 70)
    print("ЗАВДАНня 2: Інкапсуляція, property та контроль доступу")
    print("=" * 70)
    
    acc = BankAccount("Григор'євська Вікторія", 5000.0)
    print(f"Коректне створення рахунку: {acc.account_summary}")
    
    # Використання обчислюваної властивості
    acc.deposit(1500.0)
    print(f"Після депозиту (використання read-only property): {acc.account_summary}")
    
    # Перевірка валідації через setter (спроба задати некоректний тип)
    try:
        acc.balance = "багато"
    except TypeError as e:
        print(f" Перехоплено TypeError у setter: {e}")
        
    # Перевірка валідації через setter (спроба задати від'ємне число)
    try:
        acc.balance = -300
    except ValueError as e:
        print(f" Перехоплено ValueError у setter: {e}")
        
    # Демонстрація захисту __-атрибута та механізму Name Mangling
    try:
        print(acc.__balance)
    except AttributeError:
        print(" Прямий доступ до acc.__balance викликав очікуваний AttributeError.")
        
    # Доступ через обхід name mangling _ClassName__variable
    print(f" Доступ через name mangling (_BankAccount__balance): {acc._BankAccount__balance} грн\n")


# =====================================================================
# ЗАВДАННЯ 3: Магічні методи та перевантаження операторів
# Предметна область: Клас Дріб (Fraction)
# =====================================================================

import math

class Fraction:
    """Клас, що представляє математичний дріб з перевантаженням операторів."""
    def __init__(self, numerator: int, denominator: int):
        if denominator == 0:
            raise ValueError("Знаменник не може дорівнювати нулю.")
        common = math.gcd(numerator, denominator)
        # Нормалізація знака
        sign = -1 if denominator < 0 else 1
        self.numerator = sign * (numerator // common)
        self.denominator = abs(denominator // common)

    # 1. Представлення (__str__ та __repr__)
    def __str__(self) -> str:
        return f"{self.numerator}/{self.denominator}"

    def __repr__(self) -> str:
        return f"Fraction(numerator={self.numerator}, denominator={self.denominator})"

    # 2. Арифметика (__add__, __sub__)
    def __add__(self, other: 'Fraction') -> 'Fraction':
        new_num = self.numerator * other.denominator + other.numerator * self.denominator
        new_den = self.denominator * other.denominator
        return Fraction(new_num, new_den)

    def __sub__(self, other: 'Fraction') -> 'Fraction':
        new_num = self.numerator * other.denominator - other.numerator * self.denominator
        new_den = self.denominator * other.denominator
        return Fraction(new_num, new_den)

    # 3. Порівняння (__lt__, __eq__)
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Fraction):
            return NotImplemented
        return self.numerator == other.numerator and self.denominator == other.denominator

    def __lt__(self, other: 'Fraction') -> bool:
        return self.numerator * other.denominator < other.numerator * self.denominator

    # 4. Контейнер / унікальні методи (__abs__, __float__)
    def __abs__(self) -> 'Fraction':
        return Fraction(abs(self.numerator), self.denominator)

    def __float__(self) -> float:
        return self.numerator / self.denominator


def demo_task_3():
    print("=" * 70)
    print("ЗАВДАННЯ 3: Магічні методи та перевантаження операторів")
    print("=" * 70)
    
    f1 = Fraction(1, 2)
    f2 = Fraction(3, 4)
    
    print(f"Дріб 1 (__str__): {f1}, Рядкове представлення (__repr__): {repr(f1)}")
    print(f"Дріб 2: {f2}")
    
    # Арифметика
    f_add = f1 + f2
    f_sub = f2 - f1
    print(f" Додавання (f1 + f2): {f1} + {f2} = {f_add}")
    print(f" Віднімання (f2 - f1): {f2} - {f1} = {f_sub}")
    
    # Порівняння
    print(f" Порівняння ({f1} < {f2}): {f1 < f2}")
    print(f" Перевірка рівності ({f1} == Fraction(2, 4)): {f1 == Fraction(2, 4)}")
    
    # Додаткові методи
    neg_fraction = Fraction(-5, 6)
    print(f" Абсолютне значення abs({neg_fraction}): {abs(neg_fraction)}")
    print(f" Переведення у float ({f2}): float({f2}) = {float(f2)}\n")


# =====================================================================
# ЗАВДАННЯ 4: Абстрактні класи та композиція
# Предметна область: Система електронної комерції (Замовлення та Логування)
# =====================================================================

class LoggerComponent:
    """Компонент для реалізації композиції (has-a) — ведення журналу подій."""
    def log(self, message: str):
        print(f"   [LOG SYSTEM] Запис події: {message}")


class PaymentProcessor(ABC):
    """Абстрактний базовий клас для обробки платежів."""
    def __init__(self):
        self.logger = LoggerComponent()  # Композиція: процесор 'має' логер

    @abstractmethod
    def process_payment(self, amount: float) -> bool:
        pass

    @property
    @abstractmethod
    def processor_name(self) -> str:
        pass

    def common_notification(self):
        """Конкретний метод, доступний усім підкласам."""
        print("   [NOTIFICATION] Інформаційне повідомлення клієнту надіслано.")


class CreditCardProcessor(PaymentProcessor):
    """Конкретний підклас для банківських карток."""
    def process_payment(self, amount: float) -> bool:
        self.logger.log(f"Спроба оплати карткою на суму {amount:.2f} грн.")
        print(f"   Успішно списано {amount:.2f} грн через банківську картку.")
        return True

    @property
    def processor_name(self) -> str:
        return "Credit Card Processor"


class PayPalProcessor(PaymentProcessor):
    """Конкретний підклас для платіжної системи PayPal."""
    def process_payment(self, amount: float) -> bool:
        self.logger.log(f"Спроба оплати через PayPal на суму {amount:.2f} грн.")
        print(f"   Успішно проведено транзакцію на {amount:.2f} грн через PayPal.")
        return True

    @property
    def processor_name(self) -> str:
        return "PayPal Gateway"


def demo_task_4():
    print("=" * 70)
    print("ЗАВДАННЯ 4: Абстрактні класи та композиція")
    print("=" * 70)
    
    # Демонстрація заборони інстанціювання абстрактного класу
    try:
        invalid_processor = PaymentProcessor()
    except TypeError as e:
        print(f" Перехоплено TypeError при спробі створити екземпляр ABC: {e}")
        
    processors = [CreditCardProcessor(), PayPalProcessor()]
    
    print("\nПоліморфне використання конкретних підкласів із композицією (логер):")
    for proc in processors:
        print(f"\nВикористовуємо систему: {proc.processor_name}")
        proc.common_notification()
        proc.process_payment(1250.50)
    print()


# =====================================================================
# ЗАВДАННЯ 5: Паттерн проектування (Strategy)
# Предметна область: Розрахунок знижок в інтернет-магазині
# =====================================================================

class DiscountStrategy(ABC):
    """Абстрактна стратегія розрахунку знижки."""
    @abstractmethod
    def calculate_discount(self, amount: float) -> float:
        pass


class NoDiscount(DiscountStrategy):
    """Конкретна стратегія: без знижки."""
    def calculate_discount(self, amount: float) -> float:
        return 0.0


class SeasonalDiscount(DiscountStrategy):
    """Конкретна стратегія: сезонна знижка 15%."""
    def calculate_discount(self, amount: float) -> float:
        return amount * 0.15


class VIPDiscount(DiscountStrategy):
    """Конкретна стратегія: знижка для VIP-клієнтів 30%."""
    def calculate_discount(self, amount: float) -> float:
        return amount * 0.30


class ShoppingCart:
    """Контекст, що використовує стратегію знижок (зв'язок композиції)."""
    def __init__(self, customer_name: str, strategy: DiscountStrategy):
        self.customer_name = customer_name
        self.strategy = strategy
        self.items_total = 0.0

    def add_item(self, price: float):
        self.items_total += price

    def set_discount_strategy(self, strategy: DiscountStrategy):
        """Динамічна зміна стратегії під час виконання."""
        self.strategy = strategy

    def checkout(self) -> float:
        discount = self.strategy.calculate_discount(self.items_total)
        final_price = self.items_total - discount
        print(f" Клієнт: {self.customer_name} | Сума: {self.items_total:.2f} грн | Знижка: {discount:.2f} грн | До сплати: {final_price:.2f} грн")
        return final_price


def demo_task_5():
    print("=" * 70)
    print("ЗАВДАННЯ 5: Паттерн проектування — Strategy (Поведінковий)")
    print("=" * 70)
    
    # Клієнт 1 зі звичайною стратегією
    cart1 = ShoppingCart("Олександр", NoDiscount())
    cart1.add_item(2000.0)
    cart1.checkout()
    
    # Клієнт 2 з сезонною знижкою
    cart2 = ShoppingCart("Марія", SeasonalDiscount())
    cart2.add_item(4000.0)
    cart2.checkout()
    
    # Демонстрація розширюваності: динамічна зміна стратегії на VIP без зміни коду класу ShoppingCart
    print("\n[Динамічна зміна стратегії для клієнта Олександра на VIP]:")
    cart1.set_discount_strategy(VIPDiscount())
    cart1.checkout()
    print()


if __name__ == "__main__":
    print("ПОЧАТОК ВИКОНАННЯ ЛАБОРАТОРНОЇ РОБОТИ №2 (Григор'євська, КБ-25)\n")
    demo_task_1()
    demo_task_2()
    demo_task_3()
    demo_task_4()
    demo_task_5()
    print("ЗАВЕРШЕННЯ ВИКОНАННЯ ЛАБОРАТОРНОЇ РОБОТИ №2")
