---
created: 2026-08-30
purpose: OOP, SOLID principles, and Design Patterns for Infosys interview
---
# OOP, SOLID & Design Patterns - Infosys Interview

## 1. Four Pillars of OOP

### Encapsulation
Bundling data and methods that operate on that data within a single unit (class), hiding internal details.

```python
class BankAccount:
    def __init__(self, balance=0):
        self.__balance = balance  # Private attribute
    
    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount
    
    def get_balance(self):
        return self.__balance
```

**Interview Answer**: "Encapsulation is like a medicine capsule - the bitter medicine (data) is wrapped inside a sweet coating (methods). Users interact through the public interface without knowing internal implementation."

### Abstraction
Hiding complex implementation details and showing only the essential features.

```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def pay(self, amount):
        pass

class StripeProcessor(PaymentProcessor):
    def pay(self, amount):
        # Complex Stripe API integration hidden
        return f"Paid ${amount} via Stripe"
```

**Interview Answer**: "Abstraction is like driving a car - you use steering and pedals without knowing how the engine works internally."

### Inheritance
Creating new classes from existing classes, promoting code reuse.

```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        raise NotImplementedError

class Dog(Animal):
    def speak(self):
        return f"{self.name} says Woof!"

class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"
```

**Interview Answer**: "Inheritance establishes an IS-A relationship. A Dog IS an Animal, so it inherits Animal properties and can override specific behaviors."

### Polymorphism
Same interface, different implementations. Objects can be treated as instances of their parent class.

```python
def animal_sounds(animals):
    for animal in animals:
        print(animal.speak())  # Same method, different behavior

dog = Dog("Rex")
cat = Cat("Whiskers")
animal_sounds([dog, cat])
```

**Interview Answer**: "Polymorphism means 'many forms'. The same method call `speak()` produces different outputs depending on whether it's a Dog or Cat."

---

## 2. SOLID Principles

### S - Single Responsibility Principle
**"A class should have only one reason to change."**

```python
# BAD - Multiple responsibilities
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
    
    def save_to_database(self):
        pass
    
    def send_welcome_email(self):
        pass
    
    def generate_report(self):
        pass

# GOOD - Single responsibility
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserRepository:
    def save(self, user):
        pass

class EmailService:
    def send_welcome(self, user):
        pass

class ReportGenerator:
    def generate(self, user):
        pass
```

**Your Context**: "In LawPrix, I applied SRP by creating separate Django apps - one for auth, one for case management, one for notifications. Each handles one concern."

---

### O - Open/Closed Principle
**"Open for extension, closed for modification."**

```python
# BAD - Must modify existing code to add new types
class Discount:
    def get_discount(self, customer_type):
        if customer_type == "regular":
            return 0.1
        elif customer_type == "premium":
            return 0.2

# GOOD - Open for extension without modification
from abc import ABC, abstractmethod

class Discount(ABC):
    @abstractmethod
    def get_discount(self):
        pass

class RegularDiscount(Discount):
    def get_discount(self):
        return 0.1

class PremiumDiscount(Discount):
    def get_discount(self):
        return 0.2
```

---

### L - Liskov Substitution Principle
**"Subtypes must be substitutable for their base types."**

```python
# BAD - Violates LSP
class Bird:
    def fly(self):
        return "Flying"

class Penguin(Bird):
    def fly(self):
        raise Exception("Penguins can't fly!")  # Breaks expectation

# GOOD - Respects LSP
class Bird:
    def move(self):
        return "Moving"

class FlyingBird(Bird):
    def fly(self):
        return "Flying"

class Penguin(Bird):
    def move(self):
        return "Swimming"
```

**Interview Answer**: "If you replace a parent class object with a child class object, the program should still work correctly. A Penguin shouldn't break code expecting any Bird to fly."

---

### I - Interface Segregation Principle
**"Clients shouldn't be forced to depend on interfaces they don't use."**

```python
# BAD - Fat interface
class Worker(ABC):
    @abstractmethod
    def work(self):
        pass
    
    @abstractmethod
    def eat(self):
        pass
    
    @abstractmethod
    def sleep(self):
        pass

# GOOD - Segregated interfaces
class Workable(ABC):
    @abstractmethod
    def work(self):
        pass

class Eatable(ABC):
    @abstractmethod
    def eat(self):
        pass

class Robot(Workable):  # Robots work but don't eat
    def work(self):
        return "Working"

class Human(Workable, Eatable):
    def work(self):
        return "Working"
    def eat(self):
        return "Eating"
```

---

### D - Dependency Inversion Principle
**"Depend on abstractions, not concretions."**

```python
# BAD - Depends on concrete class
class MySQLDatabase:
    def query(self, sql):
        pass

class UserService:
    def __init__(self):
        self.db = MySQLDatabase()  # Tight coupling

# GOOD - Depends on abstraction
class Database(ABC):
    @abstractmethod
    def query(self, sql):
        pass

class MySQLDatabase(Database):
    def query(self, sql):
        return "MySQL result"

class PostgreSQLDatabase(Database):
    def query(self, sql):
        return "PostgreSQL result"

class UserService:
    def __init__(self, db: Database):  # Loose coupling
        self.db = db
```

**Your Context**: "In LawPrix, I use Supabase as the database layer. If I ever need to migrate to plain PostgreSQL, the DIP principle means I only need to change the implementation, not the entire application."

---

## 3. Design Patterns

### Singleton Pattern
**Ensure only one instance of a class exists throughout the application.**

```python
# Python Singleton - Simple
class DatabaseConnection:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.connection = None
        return cls._instance
    
    def connect(self):
        if self.connection is None:
            self.connection = "Connected to DB"
        return self.connection

# Usage
db1 = DatabaseConnection()
db2 = DatabaseConnection()
print(db1 is db2)  # True - same instance
```

**When to Use**: Database connections, logging, configuration managers, cache.

**Interview Answer**: "Singleton is useful when you need exactly one instance - like a database connection pool or a configuration manager. In PGPulse, the Redis cache connection could be a Singleton."

---

### Factory Pattern
**Create objects without specifying the exact class. Delegate instantiation to subclasses or factory methods.**

```python
from abc import ABC, abstractmethod

class Notification(ABC):
    @abstractmethod
    def send(self, message):
        pass

class EmailNotification(Notification):
    def send(self, message):
        return f"Email sent: {message}"

class SMSNotification(Notification):
    def send(self, message):
        return f"SMS sent: {message}"

class PushNotification(Notification):
    def send(self, message):
        return f"Push sent: {message}"

class NotificationFactory:
    @staticmethod
    def create(notification_type: str) -> Notification:
        if notification_type == "email":
            return EmailNotification()
        elif notification_type == "sms":
            return SMSNotification()
        elif notification_type == "push":
            return PushNotification()
        else:
            raise ValueError(f"Unknown type: {notification_type}")

# Usage
notification = NotificationFactory.create("email")
notification.send("Hello!")  # Email sent: Hello!
```

**When to Use**: When object creation is complex, when you need to decide at runtime which class to instantiate.

**Your Context**: "In LawPrix, notifications could use Factory pattern - different channels (email, in-app, push) created based on user preferences."

---

### Observer Pattern
**Define a one-to-many dependency between objects so that when one object changes state, all dependents are notified.**

```python
class Subject:
    def __init__(self):
        self._observers = []
    
    def attach(self, observer):
        self._observers.append(observer)
    
    def detach(self, observer):
        self._observers.remove(observer)
    
    def notify(self, message):
        for observer in self._observers:
            observer.update(message)

class Observer:
    def update(self, message):
        pass

class EmailObserver(Observer):
    def update(self, message):
        print(f"Email: {message}")

class SMSObserver(Observer):
    def update(self, message):
        print(f"SMS: {message}")

# Usage
subject = Subject()
subject.attach(EmailObserver())
subject.attach(SMSObserver())
subject.notify("New case assigned!")  # Both notified
```

---

### Strategy Pattern
**Define a family of algorithms, encapsulate each one, and make them interchangeable.**

```python
class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data):
        pass

class BubbleSort(SortStrategy):
    def sort(self, data):
        return sorted(data)  # Simplified

class QuickSort(SortStrategy):
    def sort(self, data):
        return sorted(data)  # Simplified

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self._strategy = strategy
    
    def sort(self, data):
        return self._strategy.sort(data)

# Usage
sorter = Sorter(BubbleSort())
sorter.sort([3, 1, 4, 1, 5])
```

---

## 4. OOP Interview Quick Answers

**Q: What is the difference between Abstraction and Encapsulation?**
A: Abstraction hides complexity by showing only essentials (what). Encapsulation hides internal state by restricting direct access (how). Abstraction is design-level; encapsulation is implementation-level.

**Q: When would you use Composition over Inheritance?**
A: When you need flexible "HAS-A" relationship instead of rigid "IS-A". A Car HAS-A Engine (composition) rather than Car IS-A Engine (inheritance). Composition allows runtime changes.

**Q: What is the difference between Abstract Class and Interface?**
A: Abstract class can have implementation (partial), Interface has only method signatures. Python doesn't have interfaces formally but uses ABC. Use abstract class for shared code, interface for contract.

**Q: Explain SOLID in one line each.**
A: S: One class, one job. O: Add features without changing existing code. L: Child classes must work where parent works. I: Small interfaces > large interfaces. D: Depend on abstractions.

---

> **For Infosys**: They love asking SOLID principles and Singleton/Factory patterns. Be ready to explain each with a real example from your projects.
