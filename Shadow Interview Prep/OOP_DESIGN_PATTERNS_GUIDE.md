# 🏗️ Object-Oriented Programming & Design Patterns — Complete Guide

> **Part 8 of Ahmed's shadow interview prep.**
> Covers every OOP pillar in depth, the diamond problem and how each language solves it, SOLID principles, and every major design pattern — each with clear intent, the problem it solves, and implementations in **C++, Java, Python, and TypeScript**.

---

## 📑 Table of Contents

### OOP Fundamentals
1. [What is OOP? The Big Picture](#1-what-is-oop-the-big-picture)
2. [Classes & Objects](#2-classes--objects)
3. [Encapsulation](#3-encapsulation)
4. [Abstraction](#4-abstraction)
5. [Inheritance](#5-inheritance)
6. [Polymorphism](#6-polymorphism)
7. [The Diamond Problem & How Each Language Solves It](#7-the-diamond-problem--how-each-language-solves-it)
8. [Composition vs Inheritance](#8-composition-vs-inheritance)
9. [SOLID Principles](#9-solid-principles)
10. [Common OOP Pitfalls & Interview Questions](#10-common-oop-pitfalls--interview-questions)

### Design Patterns
11. [Introduction to Design Patterns](#11-introduction-to-design-patterns)

**Creational Patterns**
12. [Singleton](#12-singleton)
13. [Factory Method](#13-factory-method)
14. [Abstract Factory](#14-abstract-factory)
15. [Builder](#15-builder)
16. [Prototype](#16-prototype)

**Structural Patterns**
17. [Adapter](#17-adapter)
18. [Decorator](#18-decorator)
19. [Facade](#19-facade)
20. [Composite](#20-composite)
21. [Proxy](#21-proxy)
22. [Bridge](#22-bridge)
23. [Flyweight](#23-flyweight)

**Behavioral Patterns**
24. [Observer](#24-observer)
25. [Strategy](#25-strategy)
26. [Command](#26-command)
27. [Template Method](#27-template-method)
28. [Iterator](#28-iterator)
29. [State](#29-state)
30. [Chain of Responsibility](#30-chain-of-responsibility)
31. [Mediator](#31-mediator)
32. [Memento](#32-memento)
33. [Visitor](#33-visitor)
34. [Interpreter](#34-interpreter)

35. [Design Patterns Interview Questions](#35-design-patterns-interview-questions)

---

## 1. What is OOP? The Big Picture

Object-Oriented Programming is a paradigm that models software as a collection of **objects** — entities that bundle data (state) and behavior (methods) together.

```
The 4 pillars of OOP:
  Encapsulation  → bundle data + methods, hide internal details
  Abstraction    → expose only what's necessary, hide complexity
  Inheritance    → derive new types from existing ones (is-a relationship)
  Polymorphism   → one interface, many implementations

Supporting concepts:
  Classes        → blueprints for objects
  Objects        → instances of classes
  Interfaces     → contracts defining what an object can do
  Abstract classes → partial implementation meant to be extended
  Composition    → build complex objects from simpler ones (has-a relationship)
  SOLID          → 5 principles for maintainable OOP design
```

### Why OOP?

```
Procedural code problem: as programs grew, global state and functions
interacted in unpredictable ways. Changes in one function broke others.
No clear ownership of data.

OOP solution: model the real world. A User owns its own data.
A Booking knows its own rules. Objects are responsible for their own state.

OOP is NOT a silver bullet:
  - Functional programming solves some problems more elegantly
  - Data-oriented design can be faster for game engines
  - But OOP remains the dominant paradigm for enterprise software
```

### 📖 Resources
- [OOP in depth — MDN](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object-oriented_programming)
- [Clean Code — Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [Head First Design Patterns](https://www.oreilly.com/library/view/head-first-design/0596007124/)

---

## 2. Classes & Objects

A **class** is a blueprint. An **object** is an instance created from that blueprint.

**C++**
```cpp
#include <iostream>
#include <string>
using namespace std;

class Player {
private:                        // access modifier
    string name;
    int    goalsScored;
    double rating;

public:
    // Constructor
    Player(const string& n, double r = 5.0)
        : name(n), goalsScored(0), rating(r) {}

    // Copy constructor
    Player(const Player& other)
        : name(other.name), goalsScored(other.goalsScored), rating(other.rating) {}

    // Destructor (runs when object goes out of scope)
    ~Player() {
        cout << name << " left the field.\n";
    }

    // Member functions
    void scoreGoal() { goalsScored++; }
    void display() const {   // const: won't modify the object
        cout << name << " | Goals: " << goalsScored << " | Rating: " << rating << "\n";
    }

    // Getters
    string getName()     const { return name; }
    int    getGoals()    const { return goalsScored; }
    double getRating()   const { return rating; }

    // Operator overloading
    bool operator>(const Player& other) const {
        return rating > other.rating;
    }

    // Static member — belongs to the class, not any instance
    static int totalPlayers;
};

int Player::totalPlayers = 0;  // define static member outside class

int main() {
    Player ahmed("Ahmed", 8.5);   // stack allocation (auto-destroyed)
    Player* sara = new Player("Sara", 9.2); // heap allocation (must delete)

    ahmed.scoreGoal();
    ahmed.display();  // Ahmed | Goals: 1 | Rating: 8.5

    cout << (ahmed > *sara ? "Ahmed" : "Sara") << " rated higher\n";

    delete sara;      // must free heap memory
    return 0;
}
```

**Java**
```java
public class Player {
    // Instance fields
    private String name;
    private int    goalsScored;
    private double rating;

    // Static field — shared across all instances
    private static int totalPlayers = 0;

    // Constructor
    public Player(String name, double rating) {
        this.name        = name;
        this.rating      = rating;
        this.goalsScored = 0;
        totalPlayers++;
    }

    // Overloaded constructor — default rating
    public Player(String name) { this(name, 5.0); }

    // Methods
    public void scoreGoal() { goalsScored++; }

    public String toString() {
        return String.format("%s | Goals: %d | Rating: %.1f", name, goalsScored, rating);
    }

    // Getters & setters
    public String getName()     { return name; }
    public int    getGoals()    { return goalsScored; }
    public double getRating()   { return rating; }
    public void   setRating(double r) {
        if (r < 0 || r > 10) throw new IllegalArgumentException("Rating must be 0-10");
        this.rating = r;
    }

    // Static method
    public static int getTotalPlayers() { return totalPlayers; }

    // equals and hashCode — always override together
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Player)) return false;
        Player p = (Player) obj;
        return name.equals(p.name);
    }

    @Override
    public int hashCode() { return name.hashCode(); }
}

// Usage
Player ahmed = new Player("Ahmed", 8.5);
Player sara  = new Player("Sara");
ahmed.scoreGoal();
System.out.println(ahmed);  // Ahmed | Goals: 1 | Rating: 8.5
System.out.println(Player.getTotalPlayers()); // 2
```

**TypeScript**
```typescript
class Player {
  private name: string;
  private goalsScored: number = 0;
  private rating: number;

  // TypeScript shorthand constructor — declares and assigns simultaneously
  constructor(
    private readonly id: string,     // readonly — can't be changed after construction
    name: string,
    rating: number = 5.0
  ) {
    this.name   = name;
    this.rating = rating;
  }

  // Methods
  scoreGoal(): void { this.goalsScored++; }

  // Getters (property syntax)
  get displayName(): string { return `${this.name} (${this.rating}★)`; }
  get goals(): number { return this.goalsScored; }

  // Setter with validation
  set newRating(r: number) {
    if (r < 0 || r > 10) throw new Error("Rating must be 0-10");
    this.rating = r;
  }

  // Static
  static readonly MAX_RATING = 10;

  toString(): string {
    return `${this.name} | Goals: ${this.goalsScored} | Rating: ${this.rating}`;
  }
}

// Usage
const ahmed = new Player("p1", "Ahmed", 8.5);
ahmed.scoreGoal();
console.log(ahmed.displayName); // "Ahmed (8.5★)"
console.log(ahmed.toString());
```

**Python**
```python
class Player:
    # Class variable — shared across all instances
    total_players: int = 0

    def __init__(self, name: str, rating: float = 5.0) -> None:
        # Instance variables
        self._name         = name          # _ prefix = convention for "protected"
        self.__rating      = rating        # __ prefix = name-mangled ("private")
        self._goals_scored = 0
        Player.total_players += 1

    # Property — like a getter (use @property, not explicit getters)
    @property
    def name(self) -> str:
        return self._name

    @property
    def rating(self) -> float:
        return self.__rating

    @rating.setter
    def rating(self, value: float) -> None:
        if not 0 <= value <= 10:
            raise ValueError("Rating must be between 0 and 10")
        self.__rating = value

    def score_goal(self) -> None:
        self._goals_scored += 1

    # Dunder methods (magic methods)
    def __str__(self) -> str:
        return f"{self._name} | Goals: {self._goals_scored} | Rating: {self.__rating}"

    def __repr__(self) -> str:
        return f"Player(name={self._name!r}, rating={self.__rating})"

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Player): return NotImplemented
        return self._name == other._name

    def __hash__(self) -> int:
        return hash(self._name)

    def __gt__(self, other: "Player") -> bool:
        return self.__rating > other.__rating

    # Class method — receives the class, not instance
    @classmethod
    def create_rookie(cls, name: str) -> "Player":
        return cls(name, rating=5.0)

    # Static method — no access to class or instance
    @staticmethod
    def is_valid_rating(r: float) -> bool:
        return 0 <= r <= 10

    def __del__(self) -> None:
        Player.total_players -= 1

# Usage
ahmed = Player("Ahmed", 8.5)
sara  = Player.create_rookie("Sara")   # class method
ahmed.score_goal()
print(ahmed)              # Ahmed | Goals: 1 | Rating: 8.5
print(ahmed > sara)       # True (uses __gt__)
ahmed.rating = 9.0        # uses setter
print(Player.total_players)  # 2
```

---

## 3. Encapsulation

Encapsulation means bundling data and the methods that operate on that data together, and **hiding internal implementation details** from the outside world. The object controls its own state — outsiders must use the public interface.

> **The problem without encapsulation:** Any code anywhere can change your object's internal data directly — you can't enforce invariants, validate inputs, or change your implementation without breaking everyone who uses it.

```
Public    → accessible from anywhere
Protected → accessible within class and subclasses
Private   → accessible only within the class itself
```

**C++**
```cpp
class BankAccount {
private:
    double balance;       // outsiders cannot touch this directly
    string owner;
    vector<string> transactions;

    void logTransaction(const string& desc) {
        transactions.push_back(desc);  // private helper
    }

public:
    BankAccount(const string& owner, double initialBalance)
        : owner(owner), balance(max(0.0, initialBalance)) {}

    // Public interface — enforces business rules
    bool deposit(double amount) {
        if (amount <= 0) return false;          // validation
        balance += amount;
        logTransaction("Deposit: " + to_string(amount));
        return true;
    }

    bool withdraw(double amount) {
        if (amount <= 0 || amount > balance) return false; // invariant
        balance -= amount;
        logTransaction("Withdraw: " + to_string(amount));
        return true;
    }

    double getBalance() const { return balance; }  // read-only access
    // No setBalance() — balance can ONLY change through deposit/withdraw
};

// ❌ Cannot do:
BankAccount acc("Ahmed", 1000);
// acc.balance = -999999;  // compile error — private!
// acc.logTransaction("cheat"); // compile error

// ✅ Must use the interface:
acc.deposit(500);
acc.withdraw(200);
cout << acc.getBalance(); // 1300
```

**Java**
```java
public class BankAccount {
    private double balance;      // strictly private
    private final String owner;
    private final List<String> transactions = new ArrayList<>();

    public BankAccount(String owner, double initialBalance) {
        this.owner   = owner;
        this.balance = Math.max(0, initialBalance);
    }

    public boolean deposit(double amount) {
        if (amount <= 0) return false;
        balance += amount;
        transactions.add(String.format("+%.2f (balance: %.2f)", amount, balance));
        return true;
    }

    public boolean withdraw(double amount) {
        if (amount <= 0 || amount > balance) return false;
        balance -= amount;
        transactions.add(String.format("-%.2f (balance: %.2f)", amount, balance));
        return true;
    }

    public double getBalance()           { return balance; }
    public List<String> getHistory()     {
        return Collections.unmodifiableList(transactions); // defensive copy
    }
    // No setter for balance — encapsulation enforces this
}
```

**TypeScript**
```typescript
class BankAccount {
  #balance: number;           // ES2022 true private field (not accessible even via any cast)
  readonly #owner: string;
  #transactions: string[] = [];

  constructor(owner: string, initialBalance: number) {
    this.#owner   = owner;
    this.#balance = Math.max(0, initialBalance);
  }

  deposit(amount: number): boolean {
    if (amount <= 0) return false;
    this.#balance += amount;
    this.#log(`+${amount}`);
    return true;
  }

  withdraw(amount: number): boolean {
    if (amount <= 0 || amount > this.#balance) return false;
    this.#balance -= amount;
    this.#log(`-${amount}`);
    return true;
  }

  get balance(): number { return this.#balance; }    // read-only property
  get history(): readonly string[] { return this.#transactions; }

  #log(entry: string): void {  // private method
    this.#transactions.push(`${entry} → balance: ${this.#balance}`);
  }
}

const acc = new BankAccount("Ahmed", 1000);
// acc.#balance // SyntaxError — truly private
acc.deposit(500);
console.log(acc.balance); // 1500
```

**Python**
```python
class BankAccount:
    def __init__(self, owner: str, initial_balance: float) -> None:
        self.__owner   = owner                      # name-mangled to _BankAccount__owner
        self.__balance = max(0.0, initial_balance)
        self.__transactions: list[str] = []

    def deposit(self, amount: float) -> bool:
        if amount <= 0:
            return False
        self.__balance += amount
        self.__log(f"+{amount:.2f}")
        return True

    def withdraw(self, amount: float) -> bool:
        if amount <= 0 or amount > self.__balance:
            return False
        self.__balance -= amount
        self.__log(f"-{amount:.2f}")
        return True

    @property
    def balance(self) -> float:
        return self.__balance            # read-only — no setter defined

    @property
    def history(self) -> tuple[str, ...]:
        return tuple(self.__transactions) # immutable copy

    def __log(self, entry: str) -> None:  # "private" helper
        self.__transactions.append(f"{entry} → balance: {self.__balance:.2f}")

# Python "privacy" note:
# __name → name-mangled to _ClassName__name (harder to access accidentally)
# _name  → convention only: "please don't touch this from outside"
# Neither is truly private — Python trusts the developer
acc = BankAccount("Ahmed", 1000)
acc.deposit(500)
print(acc.balance)           # 1500
# print(acc.__balance)       # AttributeError
# print(acc._BankAccount__balance)  # 1500 — Python allows this, but it's rude
```

---

## 4. Abstraction

Abstraction means **hiding complex implementation** and showing only the essential features. You interact with a car through the steering wheel and pedals — you don't need to know how the engine works.

In OOP: abstract classes and interfaces define **what** something does, not **how**.

**C++** (Abstract class with pure virtual functions)
```cpp
#include <vector>
#include <memory>

// Abstract base class — cannot be instantiated directly
class PaymentProcessor {
public:
    // Pure virtual function (= 0) — MUST be overridden by subclasses
    virtual bool processPayment(double amount) = 0;
    virtual bool refund(const string& transactionId) = 0;
    virtual string getName() const = 0;

    // Non-virtual — shared behavior (template method pattern)
    bool charge(double amount) {
        if (amount <= 0) return false;
        log("Charging: " + to_string(amount) + " via " + getName());
        return processPayment(amount);
    }

    // Virtual destructor — REQUIRED when using polymorphism
    virtual ~PaymentProcessor() {}

private:
    void log(const string& msg) { cout << "[LOG] " << msg << "\n"; }
};

// Concrete implementations
class StripeProcessor : public PaymentProcessor {
    string apiKey;
public:
    StripeProcessor(const string& key) : apiKey(key) {}
    bool processPayment(double amount) override {
        cout << "Stripe: charging " << amount << " with key " << apiKey << "\n";
        return true;
    }
    bool refund(const string& txId) override {
        cout << "Stripe: refunding " << txId << "\n";
        return true;
    }
    string getName() const override { return "Stripe"; }
};

class FawryProcessor : public PaymentProcessor {
public:
    bool processPayment(double amount) override {
        cout << "Fawry: charging " << amount << " EGP\n";
        return true;
    }
    bool refund(const string& txId) override { return false; } // Fawry doesn't support refunds
    string getName() const override { return "Fawry"; }
};

// Client code — works with the ABSTRACTION, not the concrete type
void processBookingPayment(PaymentProcessor& processor, double amount) {
    if (!processor.charge(amount)) {
        cout << "Payment failed!\n";
    }
}

// Usage
StripeProcessor stripe("sk_live_xxx");
FawryProcessor  fawry;
processBookingPayment(stripe, 400.00); // works
processBookingPayment(fawry, 400.00);  // also works — same interface
```

**Java** (Interface + Abstract class)
```java
// Interface — pure abstraction (no state, only method signatures)
public interface PaymentProcessor {
    boolean processPayment(double amount);
    boolean refund(String transactionId);
    String  getName();

    // Default method (Java 8+) — can provide implementation in interface
    default boolean charge(double amount) {
        if (amount <= 0) return false;
        System.out.println("[LOG] Charging: " + amount + " via " + getName());
        return processPayment(amount);
    }
}

// Abstract class — partial implementation (has state)
public abstract class BasePaymentProcessor implements PaymentProcessor {
    protected final String apiKey;
    protected final List<String> transactionLog = new ArrayList<>();

    protected BasePaymentProcessor(String apiKey) {
        this.apiKey = apiKey;
    }

    @Override
    public boolean charge(double amount) {
        log("Charging " + amount + " via " + getName());
        boolean result = processPayment(amount);
        if (result) log("Success: " + amount);
        else        log("Failed:  " + amount);
        return result;
    }

    private void log(String msg) {
        transactionLog.add(msg);
        System.out.println("[" + getName() + "] " + msg);
    }

    // Abstract — subclass MUST implement
    @Override
    public abstract boolean processPayment(double amount);

    public List<String> getLog() { return Collections.unmodifiableList(transactionLog); }
}

// Concrete
public class StripeProcessor extends BasePaymentProcessor {
    public StripeProcessor(String apiKey) { super(apiKey); }

    @Override
    public boolean processPayment(double amount) {
        // Real Stripe API call would go here
        return true;
    }

    @Override
    public boolean refund(String txId) { return true; }

    @Override
    public String getName() { return "Stripe"; }
}

// Implementing multiple interfaces (Java allows this, no diamond problem here)
public class SmartPayProcessor extends BasePaymentProcessor
    implements PaymentProcessor, Refundable, Loggable {
    // ...
}
```

**TypeScript**
```typescript
// TypeScript interface — pure contract (erased at runtime)
interface PaymentProcessor {
  processPayment(amount: number): Promise<boolean>;
  refund(transactionId: string): Promise<boolean>;
  readonly name: string;
}

// Abstract class — mix of interface and implementation
abstract class BasePaymentProcessor implements PaymentProcessor {
  protected transactions: string[] = [];

  abstract processPayment(amount: number): Promise<boolean>;
  abstract refund(transactionId: string): Promise<boolean>;
  abstract get name(): string;

  async charge(amount: number): Promise<boolean> {
    if (amount <= 0) return false;
    this.log(`Charging ${amount}`);
    const result = await this.processPayment(amount);
    this.log(result ? `Success` : `Failed`);
    return result;
  }

  protected log(msg: string): void {
    const entry = `[${this.name}] ${msg}`;
    this.transactions.push(entry);
    console.log(entry);
  }
}

class StripeProcessor extends BasePaymentProcessor {
  constructor(private readonly apiKey: string) { super(); }

  async processPayment(amount: number): Promise<boolean> {
    // await stripe.charges.create({ amount, currency: 'egp' });
    return true;
  }

  async refund(transactionId: string): Promise<boolean> { return true; }
  get name(): string { return "Stripe"; }
}
```

**Python**
```python
from abc import ABC, abstractmethod

# Abstract Base Class — cannot be instantiated
class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float) -> bool: ...

    @abstractmethod
    def refund(self, transaction_id: str) -> bool: ...

    @property
    @abstractmethod
    def name(self) -> str: ...

    # Template method — uses abstract methods internally
    def charge(self, amount: float) -> bool:
        if amount <= 0: return False
        print(f"[{self.name}] Charging {amount:.2f}")
        result = self.process_payment(amount)
        print(f"[{self.name}] {'Success' if result else 'Failed'}")
        return result

class StripeProcessor(PaymentProcessor):
    def __init__(self, api_key: str):
        self._api_key = api_key

    def process_payment(self, amount: float) -> bool:
        print(f"Stripe API: charging {amount}")
        return True

    def refund(self, transaction_id: str) -> bool:
        return True

    @property
    def name(self) -> str:
        return "Stripe"

# PaymentProcessor()  # TypeError: Can't instantiate abstract class
stripe = StripeProcessor("sk_live_xxx")
stripe.charge(400.00)

# Python Protocol — structural typing (duck typing + type checking)
from typing import Protocol

class Chargeable(Protocol):
    def charge(self, amount: float) -> bool: ...

def process_booking(processor: Chargeable, amount: float) -> None:
    processor.charge(amount)
# Any object with a charge() method satisfies Chargeable — no inheritance needed
```

---

## 5. Inheritance

Inheritance lets a class **derive from another**, inheriting its attributes and behaviors. The child class (subclass) is a specialized version of the parent class (superclass).

> **is-a relationship:** A Dog is-a Animal. An AdminUser is-a User. A Futsal pitch is-a Pitch.

**C++**
```cpp
class Pitch {
protected:
    string name;
    string city;
    double pricePerHour;
    int    capacity;

public:
    Pitch(const string& n, const string& c, double p, int cap)
        : name(n), city(c), pricePerHour(p), capacity(cap) {}

    virtual double calculateBookingPrice(double hours) const {
        return pricePerHour * hours;
    }

    virtual string getType() const { return "Pitch"; }

    virtual void display() const {
        cout << getType() << ": " << name << " | " << city
             << " | " << pricePerHour << " EGP/hr | "
             << capacity << " players\n";
    }

    virtual ~Pitch() = default;
};

// Single inheritance
class ArtificialTurfPitch : public Pitch {
    bool hasFloodlights;
public:
    ArtificialTurfPitch(const string& n, const string& c, double p, int cap, bool lights)
        : Pitch(n, c, p, cap), hasFloodlights(lights) {}

    // Override virtual function
    double calculateBookingPrice(double hours) const override {
        double base = Pitch::calculateBookingPrice(hours); // call parent
        return hasFloodlights ? base * 1.1 : base;         // 10% surcharge for lights
    }

    string getType() const override { return "Artificial Turf"; }

    void display() const override {
        Pitch::display();   // call parent's display
        cout << "  Floodlights: " << (hasFloodlights ? "Yes" : "No") << "\n";
    }
};

class FutsalPitch : public Pitch {
public:
    FutsalPitch(const string& n, const string& c, double p)
        : Pitch(n, c, p, 10) {}  // futsal always 10 players

    string getType() const override { return "Futsal"; }
};

// Polymorphic usage
vector<unique_ptr<Pitch>> pitches;
pitches.push_back(make_unique<ArtificialTurfPitch>("Field A", "Cairo", 200, 14, true));
pitches.push_back(make_unique<FutsalPitch>("Mini Field", "Cairo", 150));

for (auto& p : pitches) {
    p->display();
    cout << "2hr cost: " << p->calculateBookingPrice(2) << " EGP\n\n";
}
```

**Java**
```java
public class Pitch {
    protected String name;
    protected String city;
    protected double pricePerHour;
    protected int    capacity;

    public Pitch(String name, String city, double pricePerHour, int capacity) {
        this.name         = name;
        this.city         = city;
        this.pricePerHour = pricePerHour;
        this.capacity     = capacity;
    }

    public double calculatePrice(double hours) { return pricePerHour * hours; }
    public String getType() { return "Pitch"; }

    @Override
    public String toString() {
        return String.format("%s: %s in %s | %.0f EGP/hr | %d players",
            getType(), name, city, pricePerHour, capacity);
    }
}

public class ArtificialTurfPitch extends Pitch {
    private boolean hasFloodlights;

    public ArtificialTurfPitch(String name, String city, double price,
                                int capacity, boolean lights) {
        super(name, city, price, capacity);  // call parent constructor
        this.hasFloodlights = lights;
    }

    @Override
    public double calculatePrice(double hours) {
        double base = super.calculatePrice(hours);   // call parent method
        return hasFloodlights ? base * 1.10 : base;
    }

    @Override
    public String getType() { return "Artificial Turf"; }

    // Java is single-inheritance only — can extend ONE class
    // Can implement MULTIPLE interfaces
}

// Java final — prevent further subclassing
public final class PremiumPitch extends Pitch {
    // Nobody can extend PremiumPitch
}
```

**TypeScript**
```typescript
class Pitch {
  constructor(
    protected name: string,
    protected city: string,
    protected pricePerHour: number,
    protected capacity: number
  ) {}

  calculatePrice(hours: number): number {
    return this.pricePerHour * hours;
  }

  get type(): string { return "Pitch"; }

  toString(): string {
    return `${this.type}: ${this.name} in ${this.city} | ${this.pricePerHour} EGP/hr`;
  }
}

class ArtificialTurfPitch extends Pitch {
  constructor(
    name: string, city: string, price: number,
    capacity: number,
    private hasFloodlights: boolean
  ) {
    super(name, city, price, capacity);  // must call super() first
  }

  override calculatePrice(hours: number): number {    // override keyword (TS 4.3+)
    const base = super.calculatePrice(hours);
    return this.hasFloodlights ? base * 1.10 : base;
  }

  override get type(): string { return "Artificial Turf"; }
}

// TypeScript: single class inheritance + multiple interface implementation
interface Reviewable { addReview(rating: number, comment: string): void; }
interface Bookable   { book(userId: string, hours: number): boolean; }

class SmartPitch extends Pitch implements Reviewable, Bookable {
  addReview(rating: number, comment: string): void { /* ... */ }
  book(userId: string, hours: number): boolean { /* ... */ return true; }
}
```

**Python**
```python
class Pitch:
    def __init__(self, name: str, city: str, price_per_hour: float, capacity: int) -> None:
        self.name           = name
        self.city           = city
        self.price_per_hour = price_per_hour
        self.capacity       = capacity

    def calculate_price(self, hours: float) -> float:
        return self.price_per_hour * hours

    @property
    def type(self) -> str:
        return "Pitch"

    def __str__(self) -> str:
        return f"{self.type}: {self.name} in {self.city} | {self.price_per_hour} EGP/hr"

class ArtificialTurfPitch(Pitch):
    def __init__(self, name: str, city: str, price: float, capacity: int,
                 has_floodlights: bool = False) -> None:
        super().__init__(name, city, price, capacity)  # call parent __init__
        self.has_floodlights = has_floodlights

    def calculate_price(self, hours: float) -> float:
        base = super().calculate_price(hours)   # call parent method
        return base * 1.10 if self.has_floodlights else base

    @property
    def type(self) -> str:
        return "Artificial Turf"

# Python supports MULTIPLE inheritance (unlike Java/C# single inheritance)
class Reviewable:
    def add_review(self, rating: int, comment: str) -> None:
        print(f"Review: {rating}/5 — {comment}")

class SmartPitch(ArtificialTurfPitch, Reviewable):
    """Multiple inheritance — Python uses MRO (C3 linearization) to resolve"""
    pass

smart = SmartPitch("Elite Field", "Cairo", 300, 14, True)
smart.add_review(5, "Amazing pitch!")
print(SmartPitch.__mro__)  # see the Method Resolution Order
```

---

## 6. Polymorphism

Polymorphism means **many forms**. One interface, many implementations. Code written against the interface works with any conforming object.

### Compile-time Polymorphism (Static) — Function Overloading & Templates

```cpp
// C++ — compile-time polymorphism via function overloading and templates

class Notifier {
public:
    // Function overloading — same name, different parameters
    void send(const string& message) {
        cout << "Sending text: " << message << "\n";
    }
    void send(const string& message, const string& subject) {
        cout << "Sending email: " << subject << " — " << message << "\n";
    }
    void send(const string& message, int userId) {
        cout << "Sending push to user " << userId << ": " << message << "\n";
    }
};

// Templates — generic compile-time polymorphism
template<typename T>
T maxValue(T a, T b) { return a > b ? a : b; }

maxValue(3, 5);         // int version
maxValue(3.14, 2.71);   // double version
maxValue(string("a"), string("z")); // string version
```

### Runtime Polymorphism (Dynamic) — Virtual Functions & Interfaces

```cpp
// C++ — runtime polymorphism via virtual functions
// The actual method called is determined at runtime based on the object's type

class Notification {
public:
    virtual void send(const string& recipient, const string& msg) = 0;
    virtual string getChannel() const = 0;
    virtual ~Notification() = default;
};

class EmailNotification : public Notification {
public:
    void send(const string& to, const string& msg) override {
        cout << "Email to " << to << ": " << msg << "\n";
    }
    string getChannel() const override { return "email"; }
};

class SMSNotification : public Notification {
public:
    void send(const string& to, const string& msg) override {
        cout << "SMS to " << to << ": " << msg << "\n";
    }
    string getChannel() const override { return "sms"; }
};

class PushNotification : public Notification {
public:
    void send(const string& to, const string& msg) override {
        cout << "Push to device " << to << ": " << msg << "\n";
    }
    string getChannel() const override { return "push"; }
};

// Polymorphic function — works with ANY Notification subtype
void notifyUser(Notification& notifier, const string& userId, const string& msg) {
    cout << "[" << notifier.getChannel() << "] ";
    notifier.send(userId, msg);
}

// At runtime, the correct send() is called based on the actual object type
vector<unique_ptr<Notification>> notifiers;
notifiers.push_back(make_unique<EmailNotification>());
notifiers.push_back(make_unique<SMSNotification>());
notifiers.push_back(make_unique<PushNotification>());

for (auto& n : notifiers) {
    notifyUser(*n, "ahmed@kickoff.dev", "Your booking is confirmed!");
}
// Email to ahmed@kickoff.dev: Your booking is confirmed!
// SMS to ahmed@kickoff.dev: Your booking is confirmed!
// Push to device ahmed@kickoff.dev: Your booking is confirmed!
```

**Java**
```java
// Java — polymorphism through method overriding + interface implementation

public interface Notification {
    void send(String recipient, String message);
    String getChannel();
}

public class EmailNotification implements Notification {
    @Override public void send(String to, String msg) {
        System.out.println("Email to " + to + ": " + msg);
    }
    @Override public String getChannel() { return "email"; }
}

public class NotificationService {
    private List<Notification> channels = new ArrayList<>();

    public void addChannel(Notification n) { channels.add(n); }

    // Works with ANY Notification — polymorphism
    public void notifyAll(String recipient, String message) {
        channels.forEach(n -> n.send(recipient, message));
    }
}

// Method overriding + instanceof (type checking)
Notification n = new EmailNotification();  // reference type is Notification
n.send("ahmed", "hello");                  // calls EmailNotification.send() at runtime

if (n instanceof EmailNotification email) {  // Java 16 pattern matching
    System.out.println("Got email channel");
}

// Sealed classes (Java 17) — controlled inheritance hierarchy
public sealed class Shape permits Circle, Rectangle, Triangle {
    abstract double area();
}

public final class Circle extends Shape {
    double radius;
    double area() { return Math.PI * radius * radius; }
}

// Pattern matching switch (Java 21)
double area = switch (shape) {
    case Circle    c -> Math.PI * c.radius * c.radius;
    case Rectangle r -> r.width * r.height;
    case Triangle  t -> 0.5 * t.base * t.height;
};
```

**TypeScript**
```typescript
// TypeScript — structural typing (duck typing with types)
// Objects don't need to explicitly declare they implement an interface

interface Notification {
  send(recipient: string, message: string): Promise<void>;
  readonly channel: string;
}

class EmailNotification implements Notification {
  readonly channel = "email";
  async send(to: string, msg: string): Promise<void> {
    console.log(`Email to ${to}: ${msg}`);
  }
}

class SMSNotification implements Notification {
  readonly channel = "sms";
  async send(to: string, msg: string): Promise<void> {
    console.log(`SMS to ${to}: ${msg}`);
  }
}

// Discriminated union — TypeScript's form of sealed classes
type NotificationEvent =
  | { type: "booking_confirmed"; bookingId: string }
  | { type: "booking_cancelled"; bookingId: string; reason: string }
  | { type: "payment_failed"; amount: number; error: string };

function handleNotification(event: NotificationEvent): string {
  switch (event.type) {
    case "booking_confirmed":
      return `Booking ${event.bookingId} is confirmed!`;
    case "booking_cancelled":
      return `Booking ${event.bookingId} cancelled: ${event.reason}`;
    case "payment_failed":
      return `Payment of ${event.amount} failed: ${event.error}`;
  }
  // TypeScript ensures exhaustiveness — adding a new type without a case is an error
}
```

**Python**
```python
# Python — duck typing is the foundation of polymorphism
# "If it walks like a duck and quacks like a duck, it's a duck"
# No explicit interface declaration needed

class EmailNotification:
    def send(self, recipient: str, message: str) -> None:
        print(f"Email to {recipient}: {message}")

    @property
    def channel(self) -> str: return "email"

class SMSNotification:
    def send(self, recipient: str, message: str) -> None:
        print(f"SMS to {recipient}: {message}")

    @property
    def channel(self) -> str: return "sms"

# This works without any shared base class or interface declaration!
def notify_all(notifiers: list, recipient: str, message: str) -> None:
    for n in notifiers:
        n.send(recipient, message)  # duck typing — works if it has send()

notify_all(
    [EmailNotification(), SMSNotification()],
    "ahmed@kickoff.dev",
    "Booking confirmed!"
)

# Python method overriding
class Animal:
    def speak(self) -> str: return "..."

class Dog(Animal):
    def speak(self) -> str: return "Woof!"

class Cat(Animal):
    def speak(self) -> str: return "Meow!"

animals: list[Animal] = [Dog(), Cat(), Dog()]
for animal in animals:
    print(animal.speak())  # Woof! Meow! Woof! — correct method called at runtime
```

---

## 7. The Diamond Problem & How Each Language Solves It

The diamond problem occurs when **multiple inheritance** creates ambiguity: if class D inherits from both B and C, and both inherit from A, which version of A's method does D use?

```
        A (base class with method foo())
       / \
      B   C     (both inherit from A and override foo())
       \ /
        D        (inherits from both B and C — which foo()?)

The "diamond" shape gives the problem its name.
```

### C++ — Virtual Inheritance (Explicit Solution)

```cpp
class A {
public:
    void hello() { cout << "Hello from A\n"; }
    virtual void foo() { cout << "A::foo\n"; }
};

// Without virtual inheritance — PROBLEM:
class B : public A { public: void foo() override { cout << "B::foo\n"; } };
class C : public A { public: void foo() override { cout << "C::foo\n"; } };

class D : public B, public C {};  // D has TWO copies of A!
// D d;
// d.foo();    // COMPILE ERROR: ambiguous — B::foo or C::foo?
// d.hello();  // COMPILE ERROR: ambiguous — B::A::hello or C::A::hello?

// ✅ SOLUTION: Virtual inheritance — ensures only ONE copy of A in D
class B : virtual public A { public: void foo() override { cout << "B::foo\n"; } };
class C : virtual public A { public: void foo() override { cout << "C::foo\n"; } };
class D : public B, public C {
    // Still ambiguous for foo() — D must override to resolve
    void foo() override {
        B::foo();          // explicit: call B's version
        // or C::foo();
        // or A::foo();    // call the grandparent
    }
};

D d;
d.hello();    // ✅ Now unambiguous — only one A::hello
d.foo();      // Calls D::foo which decides

// C++ approach: the developer explicitly controls everything
// Virtual inheritance has overhead (vptr, vtable changes)
```

### Java — No Multiple Class Inheritance (Design Choice)

```java
// Java simply DISALLOWS multiple inheritance of classes
// You can extend ONLY ONE class

// ❌ Illegal in Java:
// class D extends B, C {}  // compile error

// ✅ Java's solution: interfaces (which have no state, so no ambiguity in data)
interface A {
    default String greet() { return "Hello from A"; }
}

interface B extends A {
    @Override
    default String greet() { return "Hello from B"; }
}

interface C extends A {
    @Override
    default String greet() { return "Hello from C"; }
}

// When a class implements both:
class D implements B, C {
    // COMPILE ERROR unless D overrides to resolve ambiguity
    @Override
    public String greet() {
        return B.super.greet();  // explicitly choose B's version
        // or: return C.super.greet();
        // or: return "Hello from D (my own)";
    }
}

// Java's design philosophy:
// "Multiple inheritance of type (interfaces) — yes"
// "Multiple inheritance of state/implementation — no"
// Diamond in interfaces: MUST override to resolve
// Diamond in abstract classes: impossible by design (single inheritance)
```

### TypeScript — Same as Java for Classes

```typescript
// TypeScript follows Java's model:
// Single class inheritance, multiple interface implementation

interface Flyable {
  fly(): string;
  move(): string;  // default would conflict
}

interface Swimmable {
  swim(): string;
  move(): string;  // same name!
}

// Implementing both interfaces with conflicting method names
class Duck implements Flyable, Swimmable {
  fly():  string { return "Duck flies"; }
  swim(): string { return "Duck swims"; }

  // TypeScript: if both interfaces have 'move()' with COMPATIBLE signatures,
  // one implementation satisfies both
  move(): string { return "Duck moves (flying or swimming)"; }
}

// TypeScript Mixins — simulating multiple inheritance
type Constructor<T = {}> = new (...args: any[]) => T;

function Flyable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    fly(): string { return "I can fly!"; }
    altitude = 0;
  };
}

function Swimmable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    swim(): string { return "I can swim!"; }
    depth = 0;
  };
}

class Animal {
  breathe(): string { return "breathing"; }
}

// Compose behaviors via mixins — the TypeScript/JS way to do multiple inheritance
class Duck extends Flyable(Swimmable(Animal)) {}
const duck = new Duck();
duck.fly();    // "I can fly!"
duck.swim();   // "I can swim!"
duck.breathe(); // "breathing"
```

### Python — MRO (C3 Linearization) — Most Powerful Solution

```python
# Python supports true multiple inheritance and solves diamond
# with C3 Linearization (MRO — Method Resolution Order)

class A:
    def greet(self):
        print("Hello from A")

class B(A):
    def greet(self):
        print("Hello from B")
        super().greet()  # super() follows MRO — not necessarily calls A!

class C(A):
    def greet(self):
        print("Hello from C")
        super().greet()  # super() follows MRO

class D(B, C):
    def greet(self):
        print("Hello from D")
        super().greet()  # super() follows MRO: D → B → C → A

d = D()
d.greet()
# Output:
# Hello from D
# Hello from B
# Hello from C
# Hello from A
# ← All greet() methods called exactly ONCE in MRO order

# MRO — see the order Python will search
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

# C3 Linearization algorithm:
# MRO(D) = D + merge(MRO(B), MRO(C), [B, C])
# = D, B, C, A, object
# Each class appears exactly once. Preserves local precedence order.

# Real-world example: mixin pattern
class JSONMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__)

class LogMixin:
    def log(self, msg):
        print(f"[{self.__class__.__name__}] {msg}")

class TimestampMixin:
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        from datetime import datetime
        self.created_at = datetime.now()

class Booking(TimestampMixin, JSONMixin, LogMixin):
    def __init__(self, user_id: str, pitch_id: str, hours: float):
        super().__init__()   # triggers MRO chain for all __init__ mixins
        self.user_id  = user_id
        self.pitch_id = pitch_id
        self.hours    = hours

booking = Booking("u1", "p1", 2.0)
booking.log("Booking created")    # from LogMixin
print(booking.to_json())          # from JSONMixin
print(booking.created_at)         # from TimestampMixin

# Summary of approaches:
# C++:    Virtual inheritance — explicit, developer controls everything
# Java:   No multiple class inheritance — interfaces only, must resolve conflicts
# TypeScript: Same as Java + Mixins pattern for composition
# Python: C3 MRO — automatic, elegant, cooperative super() calls
```

---

## 8. Composition vs Inheritance

> **"Favor composition over inheritance"** — Gang of Four

Inheritance models **is-a** relationships. Composition models **has-a** relationships.

```
❌ When inheritance goes wrong:
   "A Manager is-a Employee" — fine
   "A Stack is-a ArrayList" — WRONG! Stack shouldn't expose ArrayList methods like get(index)
   Java's early designers did this — Stack extends Vector — terrible API

✅ Composition:
   "A Manager has-a Employee data (and also manages others)"
   "A Stack has-a internal list (implementation detail)"
```

```python
# ❌ Wrong: inheritance for code reuse
class LinkedList:
    def append(self, val): ...
    def prepend(self, val): ...
    def get(self, index): ...   # Stack shouldn't have this!

class Stack(LinkedList):       # Stack is-a LinkedList? NO!
    def push(self, val): ...
    def pop(self): ...
    # But now users can call stack.get(0), stack.prepend() — violates stack semantics!

# ✅ Correct: composition for code reuse
class Stack:
    def __init__(self):
        self._data: list = []    # has-a list (implementation detail, hidden)

    def push(self, val): self._data.append(val)
    def pop(self):        return self._data.pop()
    def peek(self):       return self._data[-1]
    def is_empty(self):   return len(self._data) == 0
    # Users can ONLY use push/pop/peek — clean API

# Composition over inheritance — real example
# ❌ Deep inheritance hierarchy (fragile)
class Vehicle:    pass
class Car(Vehicle): pass
class ElectricCar(Car): pass
class ElectricSUV(ElectricCar): pass  # getting deep...

# ✅ Composition — combine behaviors
class Engine:
    def __init__(self, type: str, hp: int): ...
    def start(self): ...

class Battery:
    def __init__(self, capacity_kwh: float): ...
    def charge(self): ...

class Car:
    def __init__(self, engine: Engine, seating: int):
        self.engine  = engine    # has-a Engine
        self.seating = seating

class ElectricCar:
    def __init__(self, battery: Battery, seating: int):
        self.battery = battery   # has-a Battery
        self.seating = seating

# Can freely mix and match components without deep hierarchies
```

```typescript
// TypeScript — composition with dependency injection
class BookingValidator {
  validate(booking: BookingInput): ValidationResult { /* ... */ }
}

class PriceCalculator {
  calculate(pitch: Pitch, hours: number): number { /* ... */ }
}

class NotificationService {
  notify(userId: string, message: string): void { /* ... */ }
}

// BookingService composes multiple collaborators
class BookingService {
  constructor(
    private validator:    BookingValidator,     // has-a validator
    private calculator:   PriceCalculator,     // has-a calculator
    private notifier:     NotificationService, // has-a notifier
    private bookingRepo:  BookingRepository,   // has-a repository
  ) {}

  async createBooking(input: BookingInput): Promise<Booking> {
    const errors = this.validator.validate(input);
    if (errors.length) throw new ValidationError(errors);

    const price   = this.calculator.calculate(input.pitch, input.hours);
    const booking = await this.bookingRepo.save({ ...input, price });

    this.notifier.notify(input.userId, `Booking confirmed! Total: ${price} EGP`);
    return booking;
  }
}
// Each collaborator can be tested independently, swapped out (different notifier in tests)
// No deep inheritance tree — just flat composition
```

---

## 9. SOLID Principles

SOLID is an acronym for 5 design principles that make OOP code **maintainable, extensible, and testable**.

### S — Single Responsibility Principle

> **A class should have only one reason to change.**

```python
# ❌ Violation — UserManager does too many things
class UserManager:
    def create_user(self, data): ...          # business logic
    def send_welcome_email(self, user): ...   # email sending
    def save_to_db(self, user): ...           # persistence
    def generate_pdf_report(self): ...         # reporting
    # If email service changes, or DB changes, or PDF lib changes → this class changes

# ✅ Single Responsibility
class UserService:
    def __init__(self, repo: UserRepository, notifier: EmailNotifier):
        self._repo     = repo
        self._notifier = notifier

    def create_user(self, data: UserCreate) -> User:
        user = User(**data)
        self._repo.save(user)
        self._notifier.send_welcome(user)
        return user

class UserRepository:
    def save(self, user: User) -> None: ...  # only knows about persistence
    def find_by_id(self, id: str) -> User: ...

class EmailNotifier:
    def send_welcome(self, user: User) -> None: ...  # only knows about email
```

### O — Open/Closed Principle

> **Open for extension, closed for modification.** Add new behavior without changing existing code.

```typescript
// ❌ Violation — must modify this every time a new discount type is added
function calculateDiscount(booking: Booking, discountType: string): number {
  if (discountType === "student")    return booking.price * 0.2;
  if (discountType === "senior")     return booking.price * 0.15;
  if (discountType === "military")   return booking.price * 0.25;
  // Adding "first-time-user" means modifying this function — violates OCP
  return 0;
}

// ✅ Open/Closed — extend without modifying
interface DiscountStrategy {
  calculate(price: number): number;
  readonly type: string;
}

class StudentDiscount implements DiscountStrategy {
  readonly type = "student";
  calculate(price: number): number { return price * 0.2; }
}

class SeniorDiscount implements DiscountStrategy {
  readonly type = "senior";
  calculate(price: number): number { return price * 0.15; }
}

class MilitaryDiscount implements DiscountStrategy {
  readonly type = "military";
  calculate(price: number): number { return price * 0.25; }
}

// Adding "first-time-user" = add a new class, no existing code changes
class FirstTimeDiscount implements DiscountStrategy {
  readonly type = "first-time";
  calculate(price: number): number { return price * 0.1; }
}

class PricingService {
  private strategies = new Map<string, DiscountStrategy>();

  register(strategy: DiscountStrategy): void {
    this.strategies.set(strategy.type, strategy);
  }

  calculateDiscount(booking: Booking, type: string): number {
    return this.strategies.get(type)?.calculate(booking.price) ?? 0;
  }
}
```

### L — Liskov Substitution Principle

> **Subtypes must be substitutable for their base types.** Anywhere you use a Pitch, you should be able to use an ArtificialTurfPitch without breaking anything.

```java
// ❌ Violation — Square extends Rectangle but breaks behavior
class Rectangle {
    protected double width, height;
    public void setWidth(double w)  { this.width = w; }
    public void setHeight(double h) { this.height = h; }
    public double area()            { return width * height; }
}

class Square extends Rectangle {
    @Override
    public void setWidth(double w)  { this.width = this.height = w; } // ← breaks invariant!
    @Override
    public void setHeight(double h) { this.width = this.height = h; }
}

// This code works for Rectangle but BREAKS for Square — violates LSP
Rectangle r = new Square();
r.setWidth(5);
r.setHeight(10);
System.out.println(r.area()); // Expected: 50. Actual: 100! Square forced both to 10.

// ✅ Fix: Don't inherit — both implement Shape
interface Shape      { double area(); }
class Rectangle implements Shape { /* width, height, area = w*h */ }
class Square    implements Shape { /* side, area = s*s */ }
// Now they're independent — no broken substitution
```

### I — Interface Segregation Principle

> **No client should depend on methods it doesn't use.** Split fat interfaces into focused ones.

```java
// ❌ Fat interface — not every pitch can do all of this
interface Pitch {
    void book(String userId, Date start, Date end);
    void addReview(String userId, int rating);
    void streamLiveMatch();          // ← not all pitches have streaming!
    void enableFloodlights();        // ← not all pitches have floodlights!
    void generateInvoice(String id); // ← not every pitch needs invoicing
}

// A basic pitch must implement streamLiveMatch() and throw UnsupportedOperationException
// That's a violation!

// ✅ Segregated interfaces — implement only what you support
interface Bookable      { void book(String userId, Date start, Date end); }
interface Reviewable    { void addReview(String userId, int rating); }
interface Streamable    { void streamLiveMatch(); }
interface Illuminatable { void enableFloodlights(); }

class BasicPitch     implements Bookable, Reviewable { /* simple */ }
class PremiumPitch   implements Bookable, Reviewable, Streamable, Illuminatable { /* full */ }
class FutsalPitch    implements Bookable, Reviewable, Illuminatable { /* no streaming */ }
```

### D — Dependency Inversion Principle

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**

```python
# ❌ High-level BookingService directly depends on low-level MySQLRepository
class MySQLBookingRepository:
    def save(self, booking): ...
    def find_by_id(self, id): ...

class BookingService:
    def __init__(self):
        self.repo = MySQLBookingRepository()  # ← tightly coupled!
    # Can't test without a real MySQL connection
    # Can't switch to PostgreSQL without modifying BookingService

# ✅ Depend on abstraction (interface/protocol)
from typing import Protocol

class BookingRepository(Protocol):
    def save(self, booking: "Booking") -> None: ...
    def find_by_id(self, id: str) -> "Booking": ...
    def find_by_user(self, user_id: str) -> list["Booking"]: ...

# High-level module — depends on the ABSTRACTION
class BookingService:
    def __init__(self, repo: BookingRepository):  # injected from outside
        self._repo = repo

    def create_booking(self, data: dict) -> "Booking":
        booking = Booking(**data)
        self._repo.save(booking)
        return booking

# Low-level modules — implement the abstraction
class PostgreSQLBookingRepository:
    def save(self, booking): ...
    def find_by_id(self, id): ...
    def find_by_user(self, user_id): ...

class InMemoryBookingRepository:   # for testing!
    def __init__(self): self._store = {}
    def save(self, booking): self._store[booking.id] = booking
    def find_by_id(self, id): return self._store.get(id)
    def find_by_user(self, uid): return [b for b in self._store.values() if b.user_id == uid]

# Production
service = BookingService(PostgreSQLBookingRepository())

# Testing — no database needed!
service = BookingService(InMemoryBookingRepository())
```

---

## 10. Common OOP Pitfalls & Interview Questions

### Common Pitfalls

```
1. Deep inheritance hierarchies (more than 2-3 levels is a smell)
   → Prefer composition and mixins

2. Inheritance for code reuse rather than is-a relationship
   → Java's Stack extends Vector is the canonical bad example

3. Violating encapsulation via getters/setters for every field
   → If you have getX/setX for every private field, you're just writing public fields with extra steps
   → Only expose what the API needs

4. God objects (one class that does everything)
   → Violates SRP — split into collaborating objects

5. Premature abstraction
   → Don't create abstract classes/interfaces before you have 2+ implementations
   → "Rule of three" — abstract after the third duplication

6. Mutable shared state
   → Objects that multiple callers modify simultaneously cause race conditions
   → Prefer immutable objects, especially for DTOs

7. Overusing singleton
   → Singletons are global state in disguise
   → They make testing hard (can't inject a test double)
   → Prefer dependency injection

8. Breaking LSP with NotImplementedException
   → If your subclass throws "not supported" for a parent method, redesign the hierarchy
```

### Interview Questions

**Q: What is the difference between abstract class and interface?**

An abstract class can have state (instance variables), constructors, and both abstract and concrete methods. It represents a partial implementation meant to be extended. An interface (traditionally) declares a contract — what methods a class must implement — with no state and no implementation. In Java 8+ and TypeScript, interfaces can have default implementations, blurring the line. The key distinction: a class can implement many interfaces but extend only one class. Use abstract class when sharing code among closely-related classes; use interface to define a contract any unrelated class can fulfill.

**Q: Explain the diamond problem and how Python solves it.**

The diamond problem occurs when class D inherits from both B and C, which both inherit from A. If B and C override the same method from A, D has ambiguity: which version does it call? Python solves this with C3 Linearization (MRO). It computes a deterministic order: D → B → C → A. When `super().method()` is called in B, it doesn't necessarily call A — it calls the NEXT class in MRO, which is C. This means all classes in the hierarchy cooperate through `super()` and each method in the chain is called exactly once. Java avoids the problem entirely by disallowing multiple class inheritance. C++ requires explicit virtual inheritance and still leaves resolution to the programmer.

**Q: What is the Liskov Substitution Principle and can you give a real violation?**

LSP says that if S is a subtype of T, then objects of type T can be replaced by objects of type S without altering correctness. A classic violation: `Square extends Rectangle`. Rectangle has `setWidth` and `setHeight` independently. Square forces both to be equal, so setting width also changes height — code that works correctly with Rectangle breaks when given a Square. The fix is to not inherit (both implement a Shape interface instead) or redesign so Square's constraints are part of its declared contract.

**Q: Composition vs Inheritance — when do you use each?**

Use inheritance when: the relationship is genuinely is-a (Dog is-a Animal), the subclass is a specialization that extends the parent's contract without breaking it, and the hierarchy is shallow (2-3 levels max). Use composition when: you want to reuse behavior without the is-a relationship (Stack has-a List), when the class needs multiple unrelated capabilities, when you need runtime flexibility (swap one component for another), or when inheritance would expose inappropriate API from the parent. The general rule: favor composition, use inheritance only when the relationship is genuinely hierarchical and LSP is satisfied.

---

## 11. Introduction to Design Patterns

Design patterns are **reusable solutions to commonly occurring problems** in software design. They're not code to copy — they're patterns of thinking.

### Origin
The Gang of Four (GoF) — Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides — catalogued 23 patterns in "Design Patterns: Elements of Reusable Object-Oriented Software" (1994). These remain the canonical reference.

### Three Categories

```
Creational patterns — How objects are created
  Singleton, Factory Method, Abstract Factory, Builder, Prototype

Structural patterns — How objects are composed
  Adapter, Decorator, Facade, Composite, Proxy, Bridge, Flyweight

Behavioral patterns — How objects communicate
  Observer, Strategy, Command, Template Method, Iterator,
  State, Chain of Responsibility, Mediator, Memento, Visitor, Interpreter
```

### When to Use Patterns

```
✅ When you recognize the problem the pattern solves
✅ When the pattern makes code more readable to other developers who know patterns
✅ When flexibility is genuinely needed (don't add Factory if you'll only ever have one implementation)

❌ Don't use patterns:
  - Just to use patterns (over-engineering)
  - When a simpler solution is clear
  - When the problem doesn't match the pattern's intent
```

---

## 12. Singleton

### Intent
Ensure a class has **only one instance** and provide a global point of access to it.

### Problem It Solves
Some things should exist only once in a system: a configuration manager, a connection pool, a logger. Without Singleton, multiple instances could hold different state, wasting resources or causing inconsistency.

### When to Use / Avoid
Use: logging, config, thread pools, connection pools.
Avoid: when it makes testing hard (can't inject a mock), when it's really just global state in disguise.

**C++**
```cpp
class DatabasePool {
private:
    static DatabasePool* instance;
    static mutex         mtx;
    vector<Connection>   connections;

    // Private constructor — prevents external instantiation
    DatabasePool() { initializeConnections(); }
    DatabasePool(const DatabasePool&) = delete;             // no copy
    DatabasePool& operator=(const DatabasePool&) = delete;  // no assignment

    void initializeConnections() {
        for (int i = 0; i < 10; i++)
            connections.emplace_back("postgresql://localhost/kickoff");
    }

public:
    // Thread-safe getInstance using double-checked locking
    static DatabasePool& getInstance() {
        if (!instance) {
            lock_guard<mutex> lock(mtx);
            if (!instance) {            // double-check after lock
                instance = new DatabasePool();
            }
        }
        return *instance;
    }

    Connection& getConnection() {
        // Return available connection from pool
        return connections[0];  // simplified
    }
};

// Modern C++ — Meyer's Singleton (thread-safe since C++11, guaranteed by standard)
class Config {
    Config() { load(); }
    void load() { /* read config file */ }
public:
    static Config& getInstance() {
        static Config instance;  // thread-safe initialization (C++11 guarantee)
        return instance;
    }
    string get(const string& key) { /* ... */ return ""; }

    Config(const Config&) = delete;
    Config& operator=(const Config&) = delete;
};

// Usage
Config::getInstance().get("database_url");
```

**Java**
```java
// Thread-safe Singleton — Bill Pugh / Holder pattern (preferred)
public class ConfigManager {
    private final Map<String, String> config = new HashMap<>();

    private ConfigManager() {
        // Load from environment variables / config file
        config.put("db_url", System.getenv("DATABASE_URL"));
        config.put("redis_url", System.getenv("REDIS_URL"));
    }

    // Holder class is loaded lazily — only when getInstance() is called
    // Class loading is thread-safe by the JVM
    private static class Holder {
        private static final ConfigManager INSTANCE = new ConfigManager();
    }

    public static ConfigManager getInstance() {
        return Holder.INSTANCE;
    }

    public String get(String key) {
        return config.getOrDefault(key, "");
    }

    public String get(String key, String defaultValue) {
        return config.getOrDefault(key, defaultValue);
    }
}

// Enum Singleton — Josh Bloch's recommendation (serialization-safe)
public enum Logger {
    INSTANCE;

    private final List<String> log = new ArrayList<>();

    public void info(String msg)  { log.add("[INFO] " + msg); System.out.println(msg); }
    public void error(String msg) { log.add("[ERROR] " + msg); System.err.println(msg); }
    public List<String> getLog()  { return Collections.unmodifiableList(log); }
}

// Usage
Logger.INSTANCE.info("Booking created");
ConfigManager.getInstance().get("db_url");
```

**TypeScript**
```typescript
// Module-level singleton (simplest — module system ensures single instance)
class DatabasePool {
  private connections: Connection[] = [];
  private static _instance: DatabasePool | null = null;

  private constructor(private readonly maxConnections = 10) {
    this.initialize();
  }

  static getInstance(): DatabasePool {
    if (!DatabasePool._instance) {
      DatabasePool._instance = new DatabasePool();
    }
    return DatabasePool._instance;
  }

  private initialize(): void {
    // Initialize connection pool
    console.log(`Pool initialized with ${this.maxConnections} connections`);
  }

  acquire(): Connection { /* ... */ return {} as Connection; }
  release(conn: Connection): void { /* ... */ }
}

// TypeScript module singleton — even simpler (recommended)
// db-pool.ts
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
export default pool;
// Importing this module always returns the same pool instance (module caching)
```

**Python**
```python
# Python singleton — module-level approach (simplest, recommended)
# config.py
import os
from typing import Optional

class _Config:
    _instance: Optional["_Config"] = None

    def __new__(cls) -> "_Config":
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._load()
        return cls._instance

    def _load(self) -> None:
        import os
        self._data = {
            "db_url":    os.getenv("DATABASE_URL", "postgresql://localhost/kickoff"),
            "redis_url": os.getenv("REDIS_URL", "redis://localhost:6379"),
            "jwt_secret":os.getenv("JWT_SECRET", ""),
        }

    def get(self, key: str, default: str = "") -> str:
        return self._data.get(key, default)

config = _Config()  # Module-level — import this everywhere

# Thread-safe singleton with metaclass
import threading

class SingletonMeta(type):
    _instances: dict = {}
    _lock = threading.Lock()

    def __call__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._instances:
                instance = super().__call__(*args, **kwargs)
                cls._instances[cls] = instance
        return cls._instances[cls]

class Logger(metaclass=SingletonMeta):
    def __init__(self):
        self._log: list[str] = []

    def info(self, msg: str) -> None:
        entry = f"[INFO] {msg}"
        self._log.append(entry)
        print(entry)
```

---

## 13. Factory Method

### Intent
Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.

### Problem It Solves
You need to create objects but don't know which exact class until runtime. The client code shouldn't know or depend on the concrete class — it just asks for "a PaymentProcessor" and gets the right one.

**C++**
```cpp
// Product interface
class Notification {
public:
    virtual void send(const string& msg) = 0;
    virtual ~Notification() = default;
};

// Concrete products
class EmailNotification : public Notification {
    string recipient;
public:
    EmailNotification(const string& r) : recipient(r) {}
    void send(const string& msg) override {
        cout << "Email to " << recipient << ": " << msg << "\n";
    }
};

class SMSNotification : public Notification {
    string phone;
public:
    SMSNotification(const string& p) : phone(p) {}
    void send(const string& msg) override {
        cout << "SMS to " << phone << ": " << msg << "\n";
    }
};

// Creator — has the factory method
class NotificationSender {
public:
    // Factory Method — subclasses override to create different products
    virtual unique_ptr<Notification> createNotification(const string& contact) = 0;

    void notify(const string& contact, const string& message) {
        auto notification = createNotification(contact);
        notification->send(message);
    }

    virtual ~NotificationSender() = default;
};

class EmailSender : public NotificationSender {
    unique_ptr<Notification> createNotification(const string& contact) override {
        return make_unique<EmailNotification>(contact);
    }
};

class SMSSender : public NotificationSender {
    unique_ptr<Notification> createNotification(const string& contact) override {
        return make_unique<SMSNotification>(contact);
    }
};

// Simple factory function (not the full GoF pattern, but very common)
unique_ptr<Notification> createNotification(const string& type, const string& contact) {
    if (type == "email") return make_unique<EmailNotification>(contact);
    if (type == "sms")   return make_unique<SMSNotification>(contact);
    throw invalid_argument("Unknown notification type: " + type);
}
```

**Java**
```java
// Product interface
public interface PaymentProcessor {
    boolean processPayment(double amount);
    boolean refund(String transactionId);
}

// Concrete products
public class StripeProcessor  implements PaymentProcessor { /* ... */ }
public class PayPalProcessor  implements PaymentProcessor { /* ... */ }
public class FawryProcessor   implements PaymentProcessor { /* ... */ }

// Factory Method pattern — the factory is an abstract class
public abstract class PaymentGateway {
    // Factory method — subclasses create the concrete processor
    protected abstract PaymentProcessor createProcessor();

    // Template method that uses the factory method
    public boolean charge(String currency, double amount) {
        PaymentProcessor processor = createProcessor();
        return processor.processPayment(amount);
    }
}

public class StripeGateway  extends PaymentGateway {
    protected PaymentProcessor createProcessor() { return new StripeProcessor(); }
}
public class FawryGateway   extends PaymentGateway {
    protected PaymentProcessor createProcessor() { return new FawryProcessor(); }
}

// Static factory method (very common in Java — simpler)
public class PaymentProcessorFactory {
    public static PaymentProcessor create(String type) {
        return switch (type) {
            case "stripe" -> new StripeProcessor();
            case "paypal" -> new PayPalProcessor();
            case "fawry"  -> new FawryProcessor();
            default -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}

// Usage
PaymentProcessor processor = PaymentProcessorFactory.create("stripe");
processor.processPayment(400.0);
```

**TypeScript**
```typescript
interface PaymentProcessor {
  processPayment(amount: number): Promise<{ success: boolean; transactionId: string }>;
  refund(transactionId: string): Promise<boolean>;
}

class StripeProcessor implements PaymentProcessor {
  constructor(private readonly apiKey: string) {}
  async processPayment(amount: number) {
    // await stripe.charges.create(...)
    return { success: true, transactionId: `stripe_${Date.now()}` };
  }
  async refund(txId: string): Promise<boolean> { return true; }
}

class FawryProcessor implements PaymentProcessor {
  async processPayment(amount: number) {
    return { success: true, transactionId: `fawry_${Date.now()}` };
  }
  async refund(txId: string): Promise<boolean> { return false; } // Fawry no refunds
}

// Factory function — TypeScript/JS often prefers plain factory functions over classes
type ProcessorType = "stripe" | "paypal" | "fawry";

function createPaymentProcessor(type: ProcessorType, config: Record<string, string>): PaymentProcessor {
  switch (type) {
    case "stripe": return new StripeProcessor(config.apiKey);
    case "fawry":  return new FawryProcessor();
    default:       throw new Error(`Unsupported payment processor: ${type}`);
  }
}

// Usage — client doesn't know which concrete class it gets
const processor = createPaymentProcessor(
  (process.env.PAYMENT_PROVIDER as ProcessorType) ?? "stripe",
  { apiKey: process.env.STRIPE_KEY ?? "" }
);
const result = await processor.processPayment(400);
```

**Python**
```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float) -> dict: ...
    @abstractmethod
    def refund(self, transaction_id: str) -> bool: ...

class StripeProcessor(PaymentProcessor):
    def __init__(self, api_key: str):
        self._key = api_key
    def process_payment(self, amount: float) -> dict:
        return {"success": True, "transaction_id": f"stripe_{id(self)}"}
    def refund(self, tx_id: str) -> bool: return True

class FawryProcessor(PaymentProcessor):
    def process_payment(self, amount: float) -> dict:
        return {"success": True, "transaction_id": f"fawry_{id(self)}"}
    def refund(self, tx_id: str) -> bool: return False

# Factory function — Pythonic approach
def create_payment_processor(
    provider: str,
    config: dict | None = None
) -> PaymentProcessor:
    config = config or {}
    processors = {
        "stripe": lambda: StripeProcessor(config.get("api_key", "")),
        "fawry":  lambda: FawryProcessor(),
    }
    factory = processors.get(provider)
    if not factory:
        raise ValueError(f"Unsupported payment provider: {provider}")
    return factory()

# Registry-based factory (extensible — Open/Closed principle)
class PaymentProcessorRegistry:
    _registry: dict[str, type] = {}

    @classmethod
    def register(cls, name: str):
        def decorator(klass: type) -> type:
            cls._registry[name] = klass
            return klass
        return decorator

    @classmethod
    def create(cls, name: str, **kwargs) -> PaymentProcessor:
        if name not in cls._registry:
            raise ValueError(f"Unknown processor: {name}")
        return cls._registry[name](**kwargs)

@PaymentProcessorRegistry.register("stripe")
class StripeProcessor(PaymentProcessor): ...

# Adding a new processor requires zero changes to existing code:
@PaymentProcessorRegistry.register("vodafone_cash")
class VodafoneCashProcessor(PaymentProcessor): ...

processor = PaymentProcessorRegistry.create("stripe", api_key="sk_live_xxx")
```

---

## 14. Abstract Factory

### Intent
Provide an interface for creating **families of related objects** without specifying their concrete classes.

### Problem It Solves
You need to create families of objects that work together. Creating a Stripe payment ecosystem needs: a StripeProcessor, a StripeWebhookHandler, and a StripeDashboard. They must all be from the same "family." Abstract Factory ensures consistency.

```typescript
// Abstract Factory — creates families of related objects

// Product interfaces (families)
interface PaymentProcessor { processPayment(amount: number): Promise<boolean>; }
interface WebhookHandler    { handleWebhook(payload: unknown): void; }
interface InvoiceGenerator  { generateInvoice(booking: Booking): string; }

// Abstract factory — creates the entire family
interface PaymentGatewayFactory {
  createProcessor():  PaymentProcessor;
  createWebhookHandler(): WebhookHandler;
  createInvoiceGenerator(): InvoiceGenerator;
}

// Concrete family 1: Stripe
class StripeProcessor     implements PaymentProcessor {
  async processPayment(amount: number): Promise<boolean> {
    console.log("Stripe: charging", amount);
    return true;
  }
}
class StripeWebhookHandler implements WebhookHandler {
  handleWebhook(payload: unknown): void { console.log("Stripe webhook:", payload); }
}
class StripeInvoiceGenerator implements InvoiceGenerator {
  generateInvoice(booking: Booking): string { return `STRIPE-INV-${booking.id}`; }
}

class StripeGatewayFactory implements PaymentGatewayFactory {
  createProcessor():       PaymentProcessor  { return new StripeProcessor(); }
  createWebhookHandler():  WebhookHandler    { return new StripeWebhookHandler(); }
  createInvoiceGenerator():InvoiceGenerator  { return new StripeInvoiceGenerator(); }
}

// Concrete family 2: Fawry (different family, but same interfaces)
class FawryProcessor      implements PaymentProcessor { /* ... */ }
class FawryWebhookHandler  implements WebhookHandler  { /* ... */ }
class FawryInvoiceGenerator implements InvoiceGenerator { /* ... */ }

class FawryGatewayFactory implements PaymentGatewayFactory {
  createProcessor():       PaymentProcessor  { return new FawryProcessor(); }
  createWebhookHandler():  WebhookHandler    { return new FawryWebhookHandler(); }
  createInvoiceGenerator():InvoiceGenerator  { return new FawryInvoiceGenerator(); }
}

// Client code — works with the ABSTRACT factory, completely decoupled from concrete classes
class PaymentSystem {
  private processor:  PaymentProcessor;
  private handler:    WebhookHandler;
  private invoicer:   InvoiceGenerator;

  constructor(factory: PaymentGatewayFactory) {
    this.processor = factory.createProcessor();
    this.handler   = factory.createWebhookHandler();
    this.invoicer  = factory.createInvoiceGenerator();
  }

  async processBooking(booking: Booking): Promise<string> {
    await this.processor.processPayment(booking.total);
    return this.invoicer.generateInvoice(booking);
  }
}

// Switch the entire payment family by swapping the factory
const paymentSystem = new PaymentSystem(
  process.env.PAYMENT_GATEWAY === "fawry"
    ? new FawryGatewayFactory()
    : new StripeGatewayFactory()
);
```

```python
from abc import ABC, abstractmethod

class UIButton(ABC):
    @abstractmethod
    def render(self) -> str: ...
    @abstractmethod
    def on_click(self, handler) -> None: ...

class UIDialog(ABC):
    @abstractmethod
    def render(self) -> str: ...

class UIFactory(ABC):
    """Abstract factory for UI components"""
    @abstractmethod
    def create_button(self) -> UIButton: ...
    @abstractmethod
    def create_dialog(self) -> UIDialog: ...

# Family 1: Light theme
class LightButton(UIButton):
    def render(self) -> str: return "<button class='btn-light'>Click</button>"
    def on_click(self, handler) -> None: handler()

class LightDialog(UIDialog):
    def render(self) -> str: return "<div class='dialog-light'>Dialog</div>"

class LightThemeFactory(UIFactory):
    def create_button(self) -> UIButton: return LightButton()
    def create_dialog(self) -> UIDialog: return LightDialog()

# Family 2: Dark theme
class DarkButton(UIButton):
    def render(self) -> str: return "<button class='btn-dark'>Click</button>"
    def on_click(self, handler) -> None: handler()

class DarkDialog(UIDialog):
    def render(self) -> str: return "<div class='dialog-dark'>Dialog</div>"

class DarkThemeFactory(UIFactory):
    def create_button(self) -> UIButton: return DarkButton()
    def create_dialog(self) -> UIDialog: return DarkDialog()

# Client — completely unaware of which theme family it's using
def render_app(factory: UIFactory) -> None:
    button = factory.create_button()
    dialog = factory.create_dialog()
    print(button.render())
    print(dialog.render())

# Switch theme by swapping the factory
theme = "dark" if user_preference == "dark" else "light"
factory = DarkThemeFactory() if theme == "dark" else LightThemeFactory()
render_app(factory)
```

---

## 15. Builder

### Intent
Separate the **construction of a complex object** from its representation so the same construction process can create different representations.

### Problem It Solves
Creating an object requires many optional parameters, and telescoping constructors become unreadable: `new Booking(userId, pitchId, startTime, endTime, null, null, true, false, "standard", null)`.

**Java**
```java
public class Booking {
    // Required fields
    private final String userId;
    private final String pitchId;
    private final LocalDateTime startTime;
    private final LocalDateTime endTime;

    // Optional fields
    private final String notes;
    private final String promoCode;
    private final boolean recurringWeekly;
    private final List<String> invitedFriends;
    private final String paymentMethod;

    // Private constructor — only Builder can create instances
    private Booking(Builder builder) {
        this.userId          = builder.userId;
        this.pitchId         = builder.pitchId;
        this.startTime       = builder.startTime;
        this.endTime         = builder.endTime;
        this.notes           = builder.notes;
        this.promoCode       = builder.promoCode;
        this.recurringWeekly = builder.recurringWeekly;
        this.invitedFriends  = Collections.unmodifiableList(builder.invitedFriends);
        this.paymentMethod   = builder.paymentMethod;
    }

    // Getters...

    // Static inner Builder class
    public static class Builder {
        // Required
        private final String userId;
        private final String pitchId;
        private final LocalDateTime startTime;
        private final LocalDateTime endTime;

        // Optional — with defaults
        private String notes        = "";
        private String promoCode    = null;
        private boolean recurringWeekly = false;
        private List<String> invitedFriends = new ArrayList<>();
        private String paymentMethod = "card";

        public Builder(String userId, String pitchId,
                       LocalDateTime start, LocalDateTime end) {
            this.userId    = Objects.requireNonNull(userId);
            this.pitchId   = Objects.requireNonNull(pitchId);
            this.startTime = Objects.requireNonNull(start);
            this.endTime   = Objects.requireNonNull(end);
        }

        public Builder withNotes(String notes) {
            this.notes = notes;
            return this;  // ← fluent — return this for chaining
        }

        public Builder withPromoCode(String code) {
            this.promoCode = code;
            return this;
        }

        public Builder recurring() {
            this.recurringWeekly = true;
            return this;
        }

        public Builder invite(String... friends) {
            this.invitedFriends.addAll(Arrays.asList(friends));
            return this;
        }

        public Builder payWith(String method) {
            this.paymentMethod = method;
            return this;
        }

        public Booking build() {
            // Validate before creating
            if (!endTime.isAfter(startTime))
                throw new IllegalStateException("End time must be after start time");
            return new Booking(this);
        }
    }
}

// Readable, no nulls, self-documenting
Booking booking = new Booking.Builder("user-1", "pitch-1", start, end)
    .withNotes("Friday night football")
    .withPromoCode("FIRST10")
    .invite("friend1@mail.com", "friend2@mail.com")
    .payWith("card")
    .build();
```

**Python**
```python
from dataclasses import dataclass, field
from typing import Optional

# Python dataclasses — often cleaner than manual Builder for Python
@dataclass
class BookingConfig:
    # Required
    user_id:   str
    pitch_id:  str
    start_time: str
    end_time:  str
    # Optional with defaults
    notes:            str           = ""
    promo_code:       Optional[str] = None
    recurring_weekly: bool          = False
    invited_friends:  list[str]     = field(default_factory=list)
    payment_method:   str           = "card"

config = BookingConfig(
    user_id="u1", pitch_id="p1",
    start_time="18:00", end_time="20:00",
    notes="Friday night",
    invited_friends=["friend1@mail.com"],
)

# Explicit Builder pattern (for validation and complex construction)
class BookingBuilder:
    def __init__(self, user_id: str, pitch_id: str) -> None:
        self._user_id  = user_id
        self._pitch_id = pitch_id
        self._notes    = ""
        self._promo    = None
        self._friends  = []
        self._payment  = "card"

    def with_notes(self, notes: str) -> "BookingBuilder":
        self._notes = notes
        return self

    def with_promo(self, code: str) -> "BookingBuilder":
        self._promo = code
        return self

    def invite(self, *friends: str) -> "BookingBuilder":
        self._friends.extend(friends)
        return self

    def pay_with(self, method: str) -> "BookingBuilder":
        self._payment = method
        return self

    def build(self) -> BookingConfig:
        return BookingConfig(
            user_id=self._user_id, pitch_id=self._pitch_id,
            start_time="", end_time="",  # simplified
            notes=self._notes, promo_code=self._promo,
            invited_friends=self._friends, payment_method=self._payment,
        )

booking = (
    BookingBuilder("u1", "p1")
    .with_notes("Friday night")
    .with_promo("FIRST10")
    .invite("friend1@mail.com", "friend2@mail.com")
    .pay_with("card")
    .build()
)
```

**TypeScript**
```typescript
// TypeScript Builder using method chaining
class BookingBuilder {
  private config: {
    userId: string;
    pitchId: string;
    startTime?: Date;
    endTime?: Date;
    notes: string;
    promoCode?: string;
    recurringWeekly: boolean;
    invitedFriends: string[];
    paymentMethod: string;
  };

  constructor(userId: string, pitchId: string) {
    this.config = {
      userId, pitchId,
      notes: "",
      recurringWeekly: false,
      invitedFriends: [],
      paymentMethod: "card",
    };
  }

  from(start: Date): this { this.config.startTime = start; return this; }
  to  (end: Date):   this { this.config.endTime   = end;   return this; }
  withNotes(notes: string):  this { this.config.notes        = notes;  return this; }
  withPromo(code: string):   this { this.config.promoCode    = code;   return this; }
  recurring():               this { this.config.recurringWeekly = true; return this; }
  payWith(method: string):   this { this.config.paymentMethod = method; return this; }
  invite(...friends: string[]): this {
    this.config.invitedFriends.push(...friends);
    return this;
  }

  build(): Booking {
    if (!this.config.startTime || !this.config.endTime) {
      throw new Error("Start and end times are required");
    }
    if (this.config.endTime <= this.config.startTime) {
      throw new Error("End time must be after start time");
    }
    return new Booking(this.config);
  }
}

// Fluent, self-documenting, validated
const booking = new BookingBuilder("user-1", "pitch-1")
  .from(new Date("2024-06-15T18:00"))
  .to  (new Date("2024-06-15T20:00"))
  .withNotes("Friday night football")
  .withPromo("FIRST10")
  .invite("ahmed@mail.com", "sara@mail.com")
  .payWith("card")
  .build();
```

---

## 16. Prototype

### Intent
Create objects by **cloning an existing object** (the prototype), rather than constructing from scratch.

### Problem It Solves
Creating an object is expensive (database lookup, complex initialization). Or you need many similar objects with slight variations.

```python
import copy

class PitchTemplate:
    """A prototype for creating similar pitches quickly"""
    def __init__(self, surface: str, capacity: int,
                 price_per_hour: float, amenities: list[str]) -> None:
        self.surface       = surface
        self.capacity      = capacity
        self.price_per_hour= price_per_hour
        self.amenities     = amenities  # mutable list — clone must deep copy

    def clone(self) -> "PitchTemplate":
        return copy.deepcopy(self)     # Python's built-in deep clone

    def __str__(self) -> str:
        return f"{self.surface} pitch | {self.capacity} players | {self.price_per_hour} EGP/hr"

# Create a prototype (expensive setup done once)
standard_template = PitchTemplate(
    surface="artificial_turf",
    capacity=14,
    price_per_hour=200,
    amenities=["showers", "parking"]
)

# Clone and customize — O(1) construction
premium_pitch = standard_template.clone()
premium_pitch.price_per_hour = 350
premium_pitch.amenities.append("floodlights")

vip_pitch = standard_template.clone()
vip_pitch.price_per_hour = 500
vip_pitch.capacity = 22

# Standard template unchanged
print(standard_template)  # artificial_turf pitch | 14 players | 200.0 EGP/hr
print(premium_pitch)      # artificial_turf pitch | 14 players | 350 EGP/hr
```

```java
// Java — Cloneable interface
public class PitchTemplate implements Cloneable {
    private String surface;
    private int    capacity;
    private double pricePerHour;
    private List<String> amenities;

    public PitchTemplate(String surface, int cap, double price, List<String> amenities) {
        this.surface      = surface;
        this.capacity     = cap;
        this.pricePerHour = price;
        this.amenities    = new ArrayList<>(amenities);
    }

    @Override
    public PitchTemplate clone() {
        try {
            PitchTemplate clone = (PitchTemplate) super.clone();
            clone.amenities = new ArrayList<>(this.amenities); // deep copy list
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}

// Registry of prototypes
class PitchPrototypeRegistry {
    private Map<String, PitchTemplate> prototypes = new HashMap<>();

    public void register(String key, PitchTemplate template) {
        prototypes.put(key, template);
    }

    public PitchTemplate get(String key) {
        PitchTemplate template = prototypes.get(key);
        return template != null ? template.clone() : null;
    }
}
```
---

## 17. Adapter

### Intent
Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

### Problem It Solves
You have a third-party class or legacy system with a different interface than what your code expects. You can't modify the original. Adapter wraps it.

```typescript
// Problem: Our system uses this interface
interface BookingRepository {
  findById(id: string): Promise<Booking | null>;
  save(booking: Booking): Promise<Booking>;
  findByUser(userId: string): Promise<Booking[]>;
}

// But we have to use this legacy storage system with a different interface
class LegacyStorageSystem {
  fetchRecord(tableName: string, recordId: string): object { /* ... */ return {}; }
  insertRecord(tableName: string, data: object): string { return ""; }
  queryRecords(tableName: string, where: object): object[] { return []; }
}

// Adapter — wraps the legacy system, exposes our expected interface
class LegacyStorageAdapter implements BookingRepository {
  private legacy: LegacyStorageSystem;
  private TABLE = "bookings";

  constructor(legacy: LegacyStorageSystem) {
    this.legacy = legacy;
  }

  async findById(id: string): Promise<Booking | null> {
    const raw = this.legacy.fetchRecord(this.TABLE, id);
    return raw ? this.toBooking(raw) : null;
  }

  async save(booking: Booking): Promise<Booking> {
    const id = this.legacy.insertRecord(this.TABLE, this.fromBooking(booking));
    return { ...booking, id };
  }

  async findByUser(userId: string): Promise<Booking[]> {
    const rows = this.legacy.queryRecords(this.TABLE, { userId });
    return rows.map(this.toBooking);
  }

  // Transformation methods
  private toBooking(raw: object): Booking {
    const r = raw as Record<string, unknown>;
    return {
      id:          r["record_id"] as string,
      userId:      r["user_identifier"] as string,
      pitchId:     r["pitch_identifier"] as string,
      totalAmount: r["total_cost"] as number,
    } as Booking;
  }

  private fromBooking(b: Booking): object {
    return {
      record_id:         b.id,
      user_identifier:   b.userId,
      pitch_identifier:  b.pitchId,
      total_cost:        b.totalAmount,
    };
  }
}

// Client code — uses the standard interface, unaware of the legacy system
class BookingService {
  constructor(private repo: BookingRepository) {} // works with adapter!
}

const service = new BookingService(new LegacyStorageAdapter(new LegacyStorageSystem()));
```

```python
# Real-world adapter: normalizing different payment gateway APIs

class StripeSDK:
    """Third-party SDK with its own interface"""
    def create_charge(self, amount_cents: int, currency: str, source: str) -> dict:
        return {"id": "ch_123", "status": "succeeded", "amount": amount_cents}

    def create_refund(self, charge_id: str) -> dict:
        return {"id": "re_123", "status": "succeeded"}

class PayPalSDK:
    """Another third-party SDK — completely different interface"""
    def execute_payment(self, amount: float, currency: str,
                        payment_method_nonce: str) -> dict:
        return {"result": "SUCCESS", "payment_id": "PAY-123"}

    def void_transaction(self, payment_id: str) -> bool:
        return True

# Target interface — what OUR system expects
class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: float, currency: str, token: str) -> str: ...
    @abstractmethod
    def refund(self, transaction_id: str) -> bool: ...

# Adapters
class StripeAdapter(PaymentGateway):
    def __init__(self) -> None:
        self._sdk = StripeSDK()

    def charge(self, amount: float, currency: str, token: str) -> str:
        result = self._sdk.create_charge(
            amount_cents=int(amount * 100),  # Stripe uses cents
            currency=currency,
            source=token,
        )
        if result["status"] != "succeeded":
            raise PaymentError("Stripe charge failed")
        return result["id"]

    def refund(self, transaction_id: str) -> bool:
        result = self._sdk.create_refund(charge_id=transaction_id)
        return result["status"] == "succeeded"

class PayPalAdapter(PaymentGateway):
    def __init__(self) -> None:
        self._sdk = PayPalSDK()

    def charge(self, amount: float, currency: str, token: str) -> str:
        result = self._sdk.execute_payment(amount, currency, token)
        if result["result"] != "SUCCESS":
            raise PaymentError("PayPal charge failed")
        return result["payment_id"]

    def refund(self, transaction_id: str) -> bool:
        return self._sdk.void_transaction(transaction_id)

# Client — only knows about PaymentGateway
def process_payment(gateway: PaymentGateway, amount: float, token: str) -> str:
    return gateway.charge(amount, "EGP", token)
```

---

## 18. Decorator

### Intent
Attach additional responsibilities to an object **dynamically** by wrapping it. Decorators provide a flexible alternative to subclassing for extending functionality.

### Problem It Solves
You want to add behavior (logging, caching, validation, retry logic) without modifying the original class, and without an explosion of subclasses for every combination.

```python
from functools import wraps
import time

# Target interface
class BookingRepository(ABC):
    @abstractmethod
    def find_by_id(self, id: str) -> Optional["Booking"]: ...
    @abstractmethod
    def save(self, booking: "Booking") -> "Booking": ...

# Concrete implementation
class PostgreSQLBookingRepository(BookingRepository):
    def find_by_id(self, id: str) -> Optional["Booking"]:
        # actual DB query
        return Booking(id=id)
    def save(self, booking: "Booking") -> "Booking":
        return booking

# Decorator 1: Add caching
class CachingBookingRepository(BookingRepository):
    def __init__(self, wrapped: BookingRepository, cache: dict) -> None:
        self._wrapped = wrapped    # wrap the original
        self._cache   = cache

    def find_by_id(self, id: str) -> Optional["Booking"]:
        if id in self._cache:
            print(f"Cache HIT: {id}")
            return self._cache[id]
        result = self._wrapped.find_by_id(id)   # delegate to wrapped
        if result:
            self._cache[id] = result
        return result

    def save(self, booking: "Booking") -> "Booking":
        result = self._wrapped.save(booking)
        self._cache[booking.id] = result        # update cache
        return result

# Decorator 2: Add logging
class LoggingBookingRepository(BookingRepository):
    def __init__(self, wrapped: BookingRepository) -> None:
        self._wrapped = wrapped

    def find_by_id(self, id: str) -> Optional["Booking"]:
        print(f"[LOG] find_by_id({id})")
        start = time.perf_counter()
        result = self._wrapped.find_by_id(id)
        elapsed = time.perf_counter() - start
        print(f"[LOG] find_by_id({id}) → {result} in {elapsed*1000:.1f}ms")
        return result

    def save(self, booking: "Booking") -> "Booking":
        print(f"[LOG] save({booking.id})")
        return self._wrapped.save(booking)

# Decorator 3: Add metrics
class MetricsBookingRepository(BookingRepository):
    def __init__(self, wrapped: BookingRepository) -> None:
        self._wrapped = wrapped
        self._hit_count = 0
        self._miss_count = 0

    def find_by_id(self, id: str) -> Optional["Booking"]:
        result = self._wrapped.find_by_id(id)
        if result: self._hit_count += 1
        else:      self._miss_count += 1
        return result

    def save(self, booking: "Booking") -> "Booking":
        return self._wrapped.save(booking)

# Stack decorators — each wraps the previous
cache  = {}
repo = PostgreSQLBookingRepository()                    # base
repo = CachingBookingRepository(repo, cache)            # + caching
repo = LoggingBookingRepository(repo)                   # + logging
repo = MetricsBookingRepository(repo)                   # + metrics
# repo.find_by_id("123") → metrics → logging → caching → PostgreSQL

# Python decorators (functions wrapping functions) — same pattern!
def retry(max_attempts: int = 3, delay: float = 1.0):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1: raise
                    time.sleep(delay * 2 ** attempt)  # exponential backoff
        return wrapper
    return decorator

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@retry(max_attempts=3, delay=0.5)
@log_calls
def send_booking_confirmation(email: str) -> bool:
    # might fail — decorator handles retries
    return True
```

```typescript
// TypeScript Decorator pattern — wrapping HTTP client
interface HttpClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, body: unknown): Promise<T>;
}

class FetchHttpClient implements HttpClient {
  async get<T>(url: string): Promise<T> {
    const res = await fetch(url);
    return res.json();
  }
  async post<T>(url: string, body: unknown): Promise<T> {
    const res = await fetch(url, { method: "POST", body: JSON.stringify(body) });
    return res.json();
  }
}

// Decorator: add auth headers
class AuthHttpClient implements HttpClient {
  constructor(private wrapped: HttpClient, private getToken: () => string) {}

  async get<T>(url: string): Promise<T> {
    return this.wrapped.get<T>(this.addAuth(url));
  }
  async post<T>(url: string, body: unknown): Promise<T> {
    return this.wrapped.post<T>(url, body);
  }
  private addAuth(url: string): string { return url; } // simplified
}

// Decorator: add retry logic
class RetryHttpClient implements HttpClient {
  constructor(private wrapped: HttpClient, private maxRetries = 3) {}

  async get<T>(url: string): Promise<T> {
    return this.withRetry(() => this.wrapped.get<T>(url));
  }
  async post<T>(url: string, body: unknown): Promise<T> {
    return this.withRetry(() => this.wrapped.post<T>(url, body));
  }

  private async withRetry<T>(fn: () => Promise<T>): Promise<T> {
    for (let i = 0; i <= this.maxRetries; i++) {
      try { return await fn(); }
      catch (err) {
        if (i === this.maxRetries) throw err;
        await new Promise(r => setTimeout(r, 300 * 2 ** i));
      }
    }
    throw new Error("Unreachable");
  }
}

// Stack decorators
const client: HttpClient = new RetryHttpClient(
  new AuthHttpClient(
    new FetchHttpClient(),
    () => localStorage.getItem("token") ?? ""
  ),
  3
);
```

---

## 19. Facade

### Intent
Provide a **simplified interface** to a complex subsystem.

### Problem It Solves
A subsystem has many classes with complex interactions. Clients need a simple way to do common tasks without knowing the details.

```java
// Complex subsystem — many classes with intricate interactions
class BookingValidator {
    public boolean validate(BookingRequest req) { /* complex rules */ return true; }
}
class AvailabilityChecker {
    public boolean isAvailable(String pitchId, LocalDateTime start, LocalDateTime end) {
        /* check DB, cache, calendars */ return true;
    }
}
class PriceCalculator {
    public double calculate(String pitchId, double hours) { return 0; }
}
class PaymentGateway {
    public String charge(String userId, double amount, String method) { return "txn_id"; }
}
class NotificationService {
    public void sendConfirmation(String userId, String bookingId) { }
    public void notifyOwner(String ownerId, String bookingId) { }
}
class BookingRepository {
    public Booking save(Booking b) { return b; }
}

// Facade — one simple interface for the entire booking flow
public class BookingFacade {
    private final BookingValidator    validator;
    private final AvailabilityChecker availability;
    private final PriceCalculator     pricing;
    private final PaymentGateway      payment;
    private final NotificationService notifier;
    private final BookingRepository   repository;

    public BookingFacade(/* inject all the above */) { /* ... */ }

    // One method — hides all the complexity
    public BookingResult book(BookingRequest request) {
        // Validate
        if (!validator.validate(request))
            return BookingResult.failure("Invalid booking request");

        // Check availability
        if (!availability.isAvailable(request.pitchId, request.start, request.end))
            return BookingResult.failure("Pitch not available for this time slot");

        // Calculate price
        double hours = ChronoUnit.HOURS.between(request.start, request.end);
        double price = pricing.calculate(request.pitchId, hours);

        // Process payment
        String txnId = payment.charge(request.userId, price, request.paymentMethod);

        // Save booking
        Booking booking = repository.save(Booking.of(request, price, txnId));

        // Notify
        notifier.sendConfirmation(request.userId, booking.getId());
        notifier.notifyOwner(request.pitchOwnerId, booking.getId());

        return BookingResult.success(booking);
    }
}

// Client — one line, simple!
BookingResult result = bookingFacade.book(new BookingRequest(
    userId, pitchId, start, end, paymentMethod
));
```

---

## 20. Composite

### Intent
Compose objects into **tree structures** to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions uniformly.

### Problem It Solves
You have a hierarchy (organization chart, file system, UI component tree) and want to treat a single item and a group of items the same way.

```typescript
// File system example — files and folders treated uniformly

interface FileSystemItem {
  name: string;
  getSize(): number;
  display(indent?: number): void;
}

// Leaf — no children
class File implements FileSystemItem {
  constructor(public name: string, private size: number) {}

  getSize(): number { return this.size; }

  display(indent = 0): void {
    console.log(`${"  ".repeat(indent)}📄 ${this.name} (${this.size}KB)`);
  }
}

// Composite — has children (can be Files or Folders)
class Folder implements FileSystemItem {
  private children: FileSystemItem[] = [];

  constructor(public name: string) {}

  add(item: FileSystemItem): void    { this.children.push(item); }
  remove(item: FileSystemItem): void {
    this.children = this.children.filter(c => c !== item);
  }

  // Composites delegate to children — the key of the pattern
  getSize(): number {
    return this.children.reduce((total, child) => total + child.getSize(), 0);
  }

  display(indent = 0): void {
    console.log(`${"  ".repeat(indent)}📁 ${this.name}/ (${this.getSize()}KB)`);
    this.children.forEach(child => child.display(indent + 1));
  }
}

// Build a tree
const root = new Folder("kickoff-project");
const src  = new Folder("src");
const tests= new Folder("tests");

src.add(new File("booking.service.ts", 12));
src.add(new File("pitch.controller.ts", 8));
src.add(new File("user.model.ts", 5));

tests.add(new File("booking.test.ts", 15));
tests.add(new File("pitch.test.ts", 10));

root.add(src);
root.add(tests);
root.add(new File("package.json", 2));

// Treat uniformly — same call whether it's a file or folder
root.display();
// 📁 kickoff-project/ (52KB)
//   📁 src/ (25KB)
//     📄 booking.service.ts (12KB)
//     📄 pitch.controller.ts (8KB)
//     📄 user.model.ts (5KB)
//   📁 tests/ (25KB)
//   📄 package.json (2KB)

console.log("Total size:", root.getSize(), "KB"); // 52KB
```

---

## 21. Proxy

### Intent
Provide a **surrogate or placeholder** for another object to control access to it.

### Problem It Solves
You need to add something (access control, lazy loading, caching, logging) before or after accessing an object, but don't want to modify the object itself.

**Types of Proxy:**
- **Virtual Proxy:** lazy initialization (create expensive object only when needed)
- **Protection Proxy:** access control
- **Remote Proxy:** communicate with a remote object as if it's local
- **Caching Proxy:** cache results

```python
from typing import Optional
import time

class ExpensiveReport:
    """Expensive to create — takes 3 seconds to generate"""
    def __init__(self, report_id: str) -> None:
        print(f"Generating report {report_id}...")
        time.sleep(0.1)  # simulated heavy computation
        self._data = f"Report data for {report_id}"

    def get_data(self) -> str:
        return self._data

    def get_summary(self) -> str:
        return f"Summary: {self._data[:20]}..."

# Virtual Proxy — lazy initialization
class LazyReportProxy:
    def __init__(self, report_id: str) -> None:
        self._report_id = report_id
        self._report: Optional[ExpensiveReport] = None  # not created yet!

    def _get_report(self) -> ExpensiveReport:
        if self._report is None:
            self._report = ExpensiveReport(self._report_id)  # create only when needed
        return self._report

    def get_data(self) -> str:
        return self._get_report().get_data()

    def get_summary(self) -> str:
        return self._get_report().get_summary()

# Protection Proxy — access control
class ReportAccessProxy:
    def __init__(self, report: ExpensiveReport, user_role: str) -> None:
        self._report    = report
        self._user_role = user_role

    def get_data(self) -> str:
        if self._user_role not in ("admin", "manager"):
            raise PermissionError("Only admins and managers can view full report data")
        return self._report.get_data()

    def get_summary(self) -> str:
        return self._report.get_summary()  # anyone can see summary

# Caching Proxy — memoize results
class CachingReportProxy:
    _cache: dict[str, "ExpensiveReport"] = {}

    def __init__(self, report_id: str) -> None:
        self._report_id = report_id

    def _get_report(self) -> ExpensiveReport:
        if self._report_id not in CachingReportProxy._cache:
            CachingReportProxy._cache[self._report_id] = ExpensiveReport(self._report_id)
        return CachingReportProxy._cache[self._report_id]

    def get_data(self) -> str:
        return self._get_report().get_data()

# Usage
lazy   = LazyReportProxy("monthly-revenue-2024")
print("Proxy created — report not generated yet")
print(lazy.get_summary())  # NOW the report is generated
print(lazy.get_data())     # same instance reused (already generated)
```

---

## 22. Bridge

### Intent
**Decouple an abstraction from its implementation** so that the two can vary independently.

### Problem It Solves
You have two dimensions of variation (e.g., notification type × delivery channel) and creating a class for every combination causes an explosion of subclasses.

```java
// Without Bridge: N shapes × M rendering APIs = N×M classes
// Circle2D, CircleOpenGL, CircleVulkan, Rectangle2D, RectangleOpenGL... (explosion!)

// With Bridge: N shapes + M rendering APIs = N+M classes

// Implementation hierarchy (the "bridge" — crosses over)
interface Renderer {
    void renderCircle(double x, double y, double radius);
    void renderRectangle(double x, double y, double width, double height);
}

class SVGRenderer implements Renderer {
    public void renderCircle(double x, double y, double r) {
        System.out.printf("<circle cx='%.0f' cy='%.0f' r='%.0f'/>\n", x, y, r);
    }
    public void renderRectangle(double x, double y, double w, double h) {
        System.out.printf("<rect x='%.0f' y='%.0f' width='%.0f' height='%.0f'/>\n", x, y, w, h);
    }
}

class CanvasRenderer implements Renderer {
    public void renderCircle(double x, double y, double r) {
        System.out.printf("ctx.arc(%.0f, %.0f, %.0f, 0, Math.PI*2)\n", x, y, r);
    }
    public void renderRectangle(double x, double y, double w, double h) {
        System.out.printf("ctx.fillRect(%.0f, %.0f, %.0f, %.0f)\n", x, y, w, h);
    }
}

// Abstraction hierarchy — uses the renderer via "bridge"
abstract class Shape {
    protected Renderer renderer;  // THE BRIDGE — held by reference, injected
    Shape(Renderer renderer) { this.renderer = renderer; }
    abstract void draw();
}

class Circle extends Shape {
    double x, y, radius;
    Circle(Renderer r, double x, double y, double radius) {
        super(r);
        this.x = x; this.y = y; this.radius = radius;
    }
    void draw() { renderer.renderCircle(x, y, radius); }
}

class Rectangle extends Shape {
    double x, y, w, h;
    Rectangle(Renderer r, double x, double y, double w, double h) {
        super(r);
        this.x = x; this.y = y; this.w = w; this.h = h;
    }
    void draw() { renderer.renderRectangle(x, y, w, h); }
}

// Combine independently — no explosion of subclasses!
Renderer svg    = new SVGRenderer();
Renderer canvas = new CanvasRenderer();

Shape circle1 = new Circle(svg, 50, 50, 30);
Shape circle2 = new Circle(canvas, 100, 100, 50);
Shape rect    = new Rectangle(svg, 10, 10, 200, 100);

circle1.draw(); // SVG circle
circle2.draw(); // Canvas circle
rect.draw();    // SVG rectangle
```

---

## 23. Flyweight

### Intent
Use sharing to support large numbers of fine-grained objects efficiently. Separates **intrinsic state** (shared, immutable) from **extrinsic state** (unique per object, passed in).

### Problem It Solves
You need to create millions of similar objects (characters in a text editor, trees in a game world, pitches on a map). Each full object wastes memory.

```python
import weakref

# Flyweight — shared, immutable data
class PitchType:
    """Flyweight — shared by all pitches of the same surface type"""
    def __init__(self, surface: str, default_capacity: int, default_price: float) -> None:
        self.surface          = surface           # intrinsic — shared
        self.default_capacity = default_capacity  # intrinsic — shared
        self.default_price    = default_price     # intrinsic — shared
        # These are loaded once (texture, model, etc.)
        self._texture         = self._load_texture(surface)

    def _load_texture(self, surface: str) -> str:
        return f"texture://{surface}.png"  # expensive — done once!

    def display(self, name: str, city: str, price_override: float) -> None:
        """Extrinsic data (name, city, price) passed in, not stored"""
        price = price_override or self.default_price
        print(f"{name} ({city}) | {self.surface} | {price} EGP/hr | {self._texture}")

# Flyweight Factory — ensures sharing
class PitchTypeFactory:
    _types: dict[str, PitchType] = {}

    @classmethod
    def get(cls, surface: str) -> PitchType:
        if surface not in cls._types:
            print(f"Creating new PitchType for surface: {surface}")
            capacity = {"natural_grass": 22, "artificial_turf": 14, "futsal": 10}.get(surface, 14)
            price    = {"natural_grass": 300, "artificial_turf": 200, "futsal": 150}.get(surface, 200)
            cls._types[surface] = PitchType(surface, capacity, price)
        return cls._types[surface]  # reuse existing!

# Pitch — only stores EXTRINSIC (unique) data
class Pitch:
    def __init__(self, name: str, city: str, surface: str, price_override: float = 0) -> None:
        self.name           = name            # extrinsic — unique
        self.city           = city            # extrinsic — unique
        self.price_override = price_override  # extrinsic — unique
        self._type          = PitchTypeFactory.get(surface)  # shared flyweight

    def display(self) -> None:
        self._type.display(self.name, self.city, self.price_override)

# Create 1000 pitches — only 3 PitchType objects exist in memory!
pitches = []
surfaces = ["artificial_turf", "natural_grass", "futsal"]
cities   = ["Cairo", "Giza", "Alexandria"]

for i in range(1000):
    surface = surfaces[i % 3]
    city    = cities[i % 3]
    pitches.append(Pitch(f"Field {i}", city, surface))

print(f"Pitches: {len(pitches)}")             # 1000
print(f"PitchType objects: {len(PitchTypeFactory._types)}")  # 3 (shared!)
pitches[0].display()
```

---

## 24. Observer

### Intent
Define a **one-to-many dependency** so that when one object changes state, all its dependents are notified and updated automatically.

### Problem It Solves
You have an object whose state changes, and many other objects need to react. Hardcoding the notifications tightly couples everything together.

```typescript
// Observer pattern — Event system for Kickoff

// Observer interface
interface EventObserver<T> {
  update(event: T): void;
}

// Subject (Observable)
class EventEmitter<T> {
  private observers: EventObserver<T>[] = [];

  subscribe(observer: EventObserver<T>): () => void {
    this.observers.push(observer);
    // Return unsubscribe function
    return () => {
      this.observers = this.observers.filter(o => o !== observer);
    };
  }

  notify(event: T): void {
    this.observers.forEach(o => o.update(event));
  }
}

// Concrete events
interface BookingEvent {
  type: "created" | "confirmed" | "cancelled";
  booking: Booking;
  userId: string;
}

// Concrete observers
class EmailNotifier implements EventObserver<BookingEvent> {
  update(event: BookingEvent): void {
    if (event.type === "confirmed") {
      console.log(`Email: Booking ${event.booking.id} confirmed!`);
    } else if (event.type === "cancelled") {
      console.log(`Email: Booking ${event.booking.id} cancelled`);
    }
  }
}

class SMSNotifier implements EventObserver<BookingEvent> {
  update(event: BookingEvent): void {
    console.log(`SMS to ${event.userId}: Booking ${event.type}`);
  }
}

class AnalyticsTracker implements EventObserver<BookingEvent> {
  private events: BookingEvent[] = [];
  update(event: BookingEvent): void {
    this.events.push(event);
    console.log(`Analytics: tracked ${event.type} event`);
  }
}

class OwnerNotifier implements EventObserver<BookingEvent> {
  update(event: BookingEvent): void {
    if (event.type === "created") {
      console.log(`Push to owner: New booking request!`);
    }
  }
}

// Usage
const bookingEvents = new EventEmitter<BookingEvent>();

const email     = new EmailNotifier();
const sms       = new SMSNotifier();
const analytics = new AnalyticsTracker();
const owner     = new OwnerNotifier();

bookingEvents.subscribe(email);
bookingEvents.subscribe(sms);
bookingEvents.subscribe(analytics);
const unsubOwner = bookingEvents.subscribe(owner);

// When booking is created — all observers notified
bookingEvents.notify({ type: "created", booking: { id: "b1" } as Booking, userId: "u1" });

// Unsubscribe owner notifications
unsubOwner();

bookingEvents.notify({ type: "confirmed", booking: { id: "b1" } as Booking, userId: "u1" });
// Owner NOT notified — unsubscribed
```

**Python**
```python
from typing import Generic, TypeVar, Callable

T = TypeVar("T")

class EventEmitter(Generic[T]):
    def __init__(self) -> None:
        self._observers: list[Callable[[T], None]] = []

    def subscribe(self, observer: Callable[[T], None]) -> Callable[[], None]:
        self._observers.append(observer)
        def unsubscribe() -> None:
            self._observers.remove(observer)
        return unsubscribe

    def emit(self, event: T) -> None:
        for observer in self._observers[:]:  # copy — safe if observers unsubscribe
            observer(event)

# Usage with functions (not classes)
booking_events: EventEmitter[dict] = EventEmitter()

def send_email(event: dict) -> None:
    print(f"Email: booking {event['id']} {event['type']}")

def track_analytics(event: dict) -> None:
    print(f"Analytics: {event['type']}")

booking_events.subscribe(send_email)
booking_events.subscribe(track_analytics)
booking_events.emit({"id": "b1", "type": "confirmed"})
```

---

## 25. Strategy

### Intent
Define a **family of algorithms**, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

### Problem It Solves
You have multiple ways to do something (sort, validate, calculate price, send notifications) and want to switch between them without changing the code that uses them.

```typescript
// Multiple pricing strategies — switched at runtime
interface PricingStrategy {
  calculatePrice(basePrice: number, hours: number, context: BookingContext): number;
  readonly name: string;
}

interface BookingContext {
  dayOfWeek: number;   // 0=Sunday, 6=Saturday
  hour: number;        // 0-23
  userId: string;
  userTier: "free" | "pro" | "vip";
}

class StandardPricing implements PricingStrategy {
  readonly name = "standard";
  calculatePrice(base: number, hours: number, ctx: BookingContext): number {
    return base * hours;
  }
}

class PeakHoursPricing implements PricingStrategy {
  readonly name = "peak-hours";
  calculatePrice(base: number, hours: number, ctx: BookingContext): number {
    const isPeak = ctx.hour >= 17 && ctx.hour <= 21;
    const isWeekend = ctx.dayOfWeek === 5 || ctx.dayOfWeek === 6;
    let multiplier = 1;
    if (isPeak)    multiplier += 0.3;  // +30% for peak hours
    if (isWeekend) multiplier += 0.2;  // +20% for weekends
    return base * hours * multiplier;
  }
}

class MembershipPricing implements PricingStrategy {
  readonly name = "membership";
  private discounts = { free: 0, pro: 0.15, vip: 0.30 };
  calculatePrice(base: number, hours: number, ctx: BookingContext): number {
    const discount = this.discounts[ctx.userTier];
    return base * hours * (1 - discount);
  }
}

class DynamicPricing implements PricingStrategy {
  readonly name = "dynamic";
  constructor(private demandFactor: number = 1) {}
  calculatePrice(base: number, hours: number, ctx: BookingContext): number {
    return base * hours * this.demandFactor;
  }
}

// Context — uses a strategy, can swap it
class PitchBookingService {
  private strategy: PricingStrategy;

  constructor(strategy: PricingStrategy = new StandardPricing()) {
    this.strategy = strategy;
  }

  setStrategy(strategy: PricingStrategy): void {
    this.strategy = strategy;
  }

  calculateBookingPrice(pitch: Pitch, hours: number, ctx: BookingContext): number {
    const price = this.strategy.calculatePrice(pitch.pricePerHour, hours, ctx);
    console.log(`[${this.strategy.name}] ${pitch.name}: ${price} EGP`);
    return price;
  }
}

const service = new PitchBookingService();
const ctx: BookingContext = { dayOfWeek: 5, hour: 19, userId: "u1", userTier: "vip" };
const pitch = { name: "Field A", pricePerHour: 200 } as Pitch;

service.setStrategy(new PeakHoursPricing());
service.calculateBookingPrice(pitch, 2, ctx);   // 200 * 2 * 1.5 = 600

service.setStrategy(new MembershipPricing());
service.calculateBookingPrice(pitch, 2, ctx);   // 200 * 2 * 0.7 = 280 (VIP 30% off)
```

---

## 26. Command

### Intent
Encapsulate a request as an object, letting you parameterize clients with queues, logs, and undo/redo operations.

### Problem It Solves
You want to: queue operations for later, log every operation, support undo/redo, or implement transactional operations.

```python
from abc import ABC, abstractmethod
from collections import deque

class Command(ABC):
    @abstractmethod
    def execute(self) -> None: ...
    @abstractmethod
    def undo(self) -> None: ...

class Booking:
    def __init__(self, id: str): self.id = id; self.status = "pending"

class ConfirmBookingCommand(Command):
    def __init__(self, booking: Booking, notifier) -> None:
        self._booking  = booking
        self._notifier = notifier
        self._previous_status = booking.status

    def execute(self) -> None:
        self._previous_status = self._booking.status
        self._booking.status  = "confirmed"
        self._notifier.send(f"Booking {self._booking.id} confirmed!")
        print(f"Execute: confirmed booking {self._booking.id}")

    def undo(self) -> None:
        self._booking.status = self._previous_status
        print(f"Undo: reverted booking {self._booking.id} to {self._previous_status}")

class CancelBookingCommand(Command):
    def __init__(self, booking: Booking, reason: str) -> None:
        self._booking  = booking
        self._reason   = reason
        self._previous = booking.status

    def execute(self) -> None:
        self._previous        = self._booking.status
        self._booking.status  = "cancelled"
        print(f"Execute: cancelled booking {self._booking.id}: {self._reason}")

    def undo(self) -> None:
        self._booking.status = self._previous
        print(f"Undo: reinstated booking {self._booking.id}")

# Invoker — executes commands and manages history
class CommandInvoker:
    def __init__(self) -> None:
        self._history: list[Command] = []
        self._undo_stack: list[Command] = []

    def execute(self, command: Command) -> None:
        command.execute()
        self._history.append(command)
        self._undo_stack.clear()  # clear redo stack on new action

    def undo(self) -> None:
        if not self._history: return
        command = self._history.pop()
        command.undo()
        self._undo_stack.append(command)

    def redo(self) -> None:
        if not self._undo_stack: return
        command = self._undo_stack.pop()
        command.execute()
        self._history.append(command)

# Command queue — async execution
class CommandQueue:
    def __init__(self) -> None:
        self._queue: deque[Command] = deque()

    def enqueue(self, command: Command) -> None:
        self._queue.append(command)

    def process_all(self) -> None:
        while self._queue:
            self._queue.popleft().execute()

# Usage
class SimpleNotifier:
    def send(self, msg: str): print(f"[Notify] {msg}")

booking  = Booking("b1")
notifier = SimpleNotifier()
invoker  = CommandInvoker()

invoker.execute(ConfirmBookingCommand(booking, notifier))
invoker.execute(CancelBookingCommand(booking, "Player cancelled"))
invoker.undo()   # undo cancel — booking back to confirmed
invoker.undo()   # undo confirm — booking back to pending
invoker.redo()   # redo confirm
```

---

## 27. Template Method

### Intent
Define the **skeleton of an algorithm** in a base class, deferring some steps to subclasses. Subclasses override specific steps without changing the algorithm's structure.

```java
// Abstract class with template method
public abstract class BookingReportGenerator {
    // Template method — defines the algorithm skeleton
    // final — subclasses cannot override the overall structure
    public final Report generate(Date from, Date to) {
        List<Booking> bookings = fetchBookings(from, to);     // step 1
        List<Booking> filtered = filterBookings(bookings);    // step 2 — override this
        Map<String, Object> stats = calculateStats(filtered); // step 3
        return formatReport(stats);                           // step 4 — override this
    }

    // Common implementation — shared by all subclasses
    private List<Booking> fetchBookings(Date from, Date to) {
        System.out.println("Fetching bookings from DB...");
        return new ArrayList<>(); // simplified
    }

    // Abstract — subclasses MUST implement (the variable steps)
    protected abstract List<Booking> filterBookings(List<Booking> all);
    protected abstract Report formatReport(Map<String, Object> stats);

    // Hook — optional override (has a default)
    protected Map<String, Object> calculateStats(List<Booking> bookings) {
        Map<String, Object> stats = new HashMap<>();
        stats.put("count", bookings.size());
        stats.put("total", bookings.stream().mapToDouble(b -> b.total).sum());
        return stats;  // subclass can override for more stats
    }
}

public class CityRevenueReport extends BookingReportGenerator {
    private final String city;
    CityRevenueReport(String city) { this.city = city; }

    @Override
    protected List<Booking> filterBookings(List<Booking> all) {
        return all.stream().filter(b -> b.city.equals(city)).toList();
    }

    @Override
    protected Report formatReport(Map<String, Object> stats) {
        return new Report("City Revenue: " + city, stats);
    }
}

public class PremiumPitchReport extends BookingReportGenerator {
    @Override
    protected List<Booking> filterBookings(List<Booking> all) {
        return all.stream().filter(b -> b.totalAmount > 400).toList();
    }

    @Override
    protected Report formatReport(Map<String, Object> stats) {
        return new Report("Premium Pitches", stats);
    }

    @Override
    protected Map<String, Object> calculateStats(List<Booking> bookings) {
        Map<String, Object> stats = super.calculateStats(bookings); // call parent
        stats.put("avg", bookings.stream().mapToDouble(b -> b.totalAmount).average().orElse(0));
        return stats; // adds average revenue
    }
}
```

---

## 28. Iterator

### Intent
Provide a way to **sequentially access elements** of a collection without exposing its underlying representation.

```typescript
// Custom iterator for paginated results
interface Iterator<T> {
  hasNext(): boolean;
  next(): T;
}

interface Iterable<T> {
  createIterator(): Iterator<T>;
}

// Paginated API iterator
class BookingPaginatedIterator implements Iterator<Booking> {
  private currentPage = 0;
  private buffer: Booking[] = [];
  private exhausted = false;

  constructor(
    private userId: string,
    private pageSize: number,
    private fetchPage: (page: number, size: number) => Promise<Booking[]>
  ) {}

  hasNext(): boolean {
    return this.buffer.length > 0 || !this.exhausted;
  }

  async nextBatch(): Promise<Booking[]> {
    if (this.exhausted) return [];
    const batch = await this.fetchPage(this.currentPage++, this.pageSize);
    if (batch.length < this.pageSize) this.exhausted = true;
    return batch;
  }

  next(): Booking {
    if (!this.hasNext()) throw new Error("No more elements");
    return this.buffer.shift()!;
  }
}

// Python has built-in iterator protocol — __iter__ and __next__
```

**Python**
```python
class BookingCollection:
    """Collection with a custom iterator"""
    def __init__(self, bookings: list) -> None:
        self._bookings = bookings

    def __iter__(self):
        return BookingIterator(self._bookings)

    def __len__(self): return len(self._bookings)

class BookingIterator:
    def __init__(self, bookings: list) -> None:
        self._bookings = bookings
        self._index    = 0

    def __iter__(self): return self

    def __next__(self):
        if self._index >= len(self._bookings):
            raise StopIteration
        booking = self._bookings[self._index]
        self._index += 1
        return booking

# Python generator — simplest form of custom iterator
def active_bookings(all_bookings):
    for b in all_bookings:
        if b.status in ("confirmed", "pending"):
            yield b                 # ← generator — lazy, memory-efficient

# Infinite generator
def booking_id_generator(prefix: str = "BK"):
    counter = 1
    while True:
        yield f"{prefix}-{counter:06d}"
        counter += 1

gen = booking_id_generator()
print(next(gen))  # BK-000001
print(next(gen))  # BK-000002
```

---

## 29. State

### Intent
Allow an object to **alter its behavior when its internal state changes**. The object will appear to change its class.

### Problem It Solves
Objects with complex state transitions using if/else chains that grow unmanageable.

```typescript
// Booking goes through states: pending → confirmed → completed / cancelled
// Each state has different allowed transitions

interface BookingState {
  confirm(booking: BookingContext): void;
  cancel(booking: BookingContext, reason: string): void;
  complete(booking: BookingContext): void;
  getStatusName(): string;
}

// State context
class BookingContext {
  private state: BookingState;
  id: string;

  constructor(id: string) {
    this.id    = id;
    this.state = new PendingState();  // initial state
  }

  // Delegate all behavior to current state
  confirm(): void { this.state.confirm(this); }
  cancel(reason: string): void { this.state.cancel(this, reason); }
  complete(): void { this.state.complete(this); }
  get status(): string { return this.state.getStatusName(); }

  // States call this to transition
  setState(state: BookingState): void {
    console.log(`Transition: ${this.state.getStatusName()} → ${state.getStatusName()}`);
    this.state = state;
  }
}

class PendingState implements BookingState {
  confirm(ctx: BookingContext): void {
    // Validate, charge payment...
    ctx.setState(new ConfirmedState());
  }
  cancel(ctx: BookingContext, reason: string): void {
    ctx.setState(new CancelledState(reason));
  }
  complete(ctx: BookingContext): void {
    throw new Error("Cannot complete a pending booking");
  }
  getStatusName(): string { return "pending"; }
}

class ConfirmedState implements BookingState {
  confirm(ctx: BookingContext): void {
    throw new Error("Booking is already confirmed");
  }
  cancel(ctx: BookingContext, reason: string): void {
    // Process refund...
    ctx.setState(new CancelledState(reason));
  }
  complete(ctx: BookingContext): void {
    ctx.setState(new CompletedState());
  }
  getStatusName(): string { return "confirmed"; }
}

class CompletedState implements BookingState {
  confirm(ctx: BookingContext): void { throw new Error("Booking already completed"); }
  cancel(ctx: BookingContext, reason: string): void { throw new Error("Cannot cancel completed booking"); }
  complete(ctx: BookingContext): void { throw new Error("Already completed"); }
  getStatusName(): string { return "completed"; }
}

class CancelledState implements BookingState {
  constructor(private reason: string) {}
  confirm(ctx: BookingContext): void { throw new Error("Cannot confirm cancelled booking"); }
  cancel(ctx: BookingContext, reason: string): void { throw new Error("Already cancelled"); }
  complete(ctx: BookingContext): void { throw new Error("Cannot complete cancelled booking"); }
  getStatusName(): string { return `cancelled (${this.reason})`; }
}

// Usage
const booking = new BookingContext("b1");
console.log(booking.status); // pending
booking.confirm();            // pending → confirmed
booking.complete();           // confirmed → completed
// booking.cancel("changed mind"); // Error: Cannot cancel completed booking
```

---

## 30. Chain of Responsibility

### Intent
Pass a request along a **chain of handlers**, each deciding to handle it or pass it further.

### Problem It Solves
Multiple objects might handle a request, but you don't know which one at design time. You want to decouple sender from receivers and give multiple handlers a chance.

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Optional

class BookingRequest:
    def __init__(self, user_id: str, promo_code: Optional[str], amount: float) -> None:
        self.user_id    = user_id
        self.promo_code = promo_code
        self.amount     = amount
        self.discount   = 0.0
        self.blocked    = False
        self.notes: list[str] = []

class RequestHandler(ABC):
    def __init__(self) -> None:
        self._next: Optional[RequestHandler] = None

    def set_next(self, handler: "RequestHandler") -> "RequestHandler":
        self._next = handler
        return handler  # return next for chaining

    def handle(self, request: BookingRequest) -> Optional[str]:
        if self._next:
            return self._next.handle(request)
        return None

    @abstractmethod
    def process(self, request: BookingRequest) -> Optional[str]: ...

class BlacklistHandler(RequestHandler):
    BLACKLISTED = {"banned_user_1", "spammer_user"}

    def process(self, request: BookingRequest) -> Optional[str]:
        if request.user_id in self.BLACKLISTED:
            request.blocked = True
            return f"User {request.user_id} is blacklisted"
        return self._next.handle(request) if self._next else None

    def handle(self, request: BookingRequest) -> Optional[str]:
        return self.process(request)

class PromoCodeHandler(RequestHandler):
    CODES = {"FIRST10": 0.10, "VIP20": 0.20, "RAMADAN15": 0.15}

    def handle(self, request: BookingRequest) -> Optional[str]:
        if request.promo_code and request.promo_code in self.CODES:
            discount = self.CODES[request.promo_code]
            request.discount  += discount
            request.amount    *= (1 - discount)
            request.notes.append(f"Promo {request.promo_code}: -{discount*100:.0f}%")
        return super().handle(request)

class LoyaltyDiscountHandler(RequestHandler):
    def handle(self, request: BookingRequest) -> Optional[str]:
        bookings_count = self._get_user_bookings(request.user_id)
        if bookings_count >= 20:
            request.discount += 0.15
            request.amount   *= 0.85
            request.notes.append("Loyalty 20+ bookings: -15%")
        elif bookings_count >= 10:
            request.discount += 0.10
            request.amount   *= 0.90
            request.notes.append("Loyalty 10+ bookings: -10%")
        return super().handle(request)

    def _get_user_bookings(self, user_id: str) -> int:
        return 12  # simplified

class PaymentValidationHandler(RequestHandler):
    def handle(self, request: BookingRequest) -> Optional[str]:
        if request.amount > 10000:
            return "Amount exceeds maximum allowed: 10,000 EGP"
        if request.amount < 50:
            return "Minimum booking amount: 50 EGP"
        return super().handle(request)

# Build the chain
blacklist  = BlacklistHandler()
promo      = PromoCodeHandler()
loyalty    = LoyaltyDiscountHandler()
validation = PaymentValidationHandler()

# Chain: blacklist → promo → loyalty → validation
blacklist.set_next(promo).set_next(loyalty).set_next(validation)

# Process a request
req = BookingRequest("user123", "FIRST10", 400.0)
error = blacklist.handle(req)
if error:
    print(f"Rejected: {error}")
else:
    print(f"Approved! Final amount: {req.amount:.2f} EGP")
    print(f"Notes: {req.notes}")
# Final amount: 306.00 EGP  (FIRST10 -10% + Loyalty -15% = -25%)
# Notes: ['Promo FIRST10: -10%', 'Loyalty 10+ bookings: -10%']
```

---

## 31. Mediator

### Intent
Define an object (mediator) that **encapsulates how objects interact**. Objects communicate through the mediator instead of directly.

### Problem It Solves
Many objects interact with many others, creating a spider web of dependencies. Mediator replaces N×N connections with N connections to one central object.

```java
// Chat room is the mediator
interface ChatMediator {
    void sendMessage(String message, User sender);
    void addUser(User user);
}

abstract class User {
    protected ChatMediator mediator;
    protected String name;

    User(ChatMediator mediator, String name) {
        this.mediator = mediator;
        this.name     = name;
    }

    abstract void send(String message);
    abstract void receive(String message, String from);
}

class ChatRoom implements ChatMediator {
    private final List<User> users = new ArrayList<>();

    public void addUser(User user) { users.add(user); }

    public void sendMessage(String message, User sender) {
        users.stream()
             .filter(u -> u != sender)  // don't echo back to sender
             .forEach(u -> u.receive(message, sender.name));
    }
}

class ChatUser extends User {
    ChatUser(ChatMediator mediator, String name) {
        super(mediator, name);
        mediator.addUser(this);   // register with mediator
    }

    public void send(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(message, this);  // delegate to mediator
    }

    public void receive(String message, String from) {
        System.out.println(name + " receives from " + from + ": " + message);
    }
}

// Usage — users don't know about each other, only the mediator
ChatRoom room = new ChatRoom();
User ahmed = new ChatUser(room, "Ahmed");
User sara  = new ChatUser(room, "Sara");
User omar  = new ChatUser(room, "Omar");

ahmed.send("Who wants to play Friday?");
// Sara receives from Ahmed: Who wants to play Friday?
// Omar receives from Ahmed: Who wants to play Friday?
```

---

## 32. Memento

### Intent
Capture and externalize an object's internal state so the object can be **restored to this state later** (undo mechanism).

```typescript
// Booking form with undo support
class BookingFormState {
  constructor(
    public readonly pitchId: string,
    public readonly date: string,
    public readonly startHour: number,
    public readonly duration: number,
    public readonly promoCode: string,
  ) {}
}

// Originator
class BookingForm {
  private pitchId  = "";
  private date     = "";
  private startHour= 18;
  private duration = 1;
  private promoCode= "";

  // Save current state as memento
  save(): BookingFormState {
    return new BookingFormState(
      this.pitchId, this.date, this.startHour, this.duration, this.promoCode
    );
  }

  // Restore from memento
  restore(state: BookingFormState): void {
    this.pitchId   = state.pitchId;
    this.date      = state.date;
    this.startHour = state.startHour;
    this.duration  = state.duration;
    this.promoCode = state.promoCode;
  }

  setField(field: keyof BookingFormState, value: string | number): void {
    (this as Record<string, unknown>)[field] = value;
  }

  getState(): string {
    return `Pitch:${this.pitchId} Date:${this.date} ${this.startHour}:00 + ${this.duration}h`;
  }
}

// Caretaker — manages the history
class FormHistory {
  private history: BookingFormState[] = [];

  push(state: BookingFormState): void { this.history.push(state); }

  pop(): BookingFormState | null {
    return this.history.pop() ?? null;
  }

  get hasHistory(): boolean { return this.history.length > 0; }
}

// Usage
const form    = new BookingForm();
const history = new FormHistory();

history.push(form.save());           // save initial state
form.setField("pitchId", "p1");
form.setField("date", "2024-06-15");

history.push(form.save());           // save after editing
form.setField("startHour", 20);
form.setField("promoCode", "FIRST10");

console.log(form.getState()); // Pitch:p1 Date:2024-06-15 20:00 + 1h

// Undo
const prev = history.pop();
if (prev) form.restore(prev);
console.log(form.getState()); // Pitch:p1 Date:2024-06-15 18:00 + 1h (restored)
```

---

## 33. Visitor

### Intent
Separate **operations from the objects** they operate on. Add new operations to an object structure without modifying the objects.

### Problem It Solves
You have a complex object hierarchy and want to add new operations to all types in the hierarchy without modifying each class (Open/Closed principle for operations).

```python
from abc import ABC, abstractmethod

# Element interface
class PitchElement(ABC):
    @abstractmethod
    def accept(self, visitor: "PitchVisitor") -> None: ...

# Concrete elements
class StandardPitch(PitchElement):
    def __init__(self, name: str, price: float) -> None:
        self.name  = name
        self.price = price

    def accept(self, visitor: "PitchVisitor") -> None:
        visitor.visit_standard(self)

class PremiumPitch(PitchElement):
    def __init__(self, name: str, price: float, has_vip_lounge: bool) -> None:
        self.name         = name
        self.price        = price
        self.has_vip_lounge = has_vip_lounge

    def accept(self, visitor: "PitchVisitor") -> None:
        visitor.visit_premium(self)

class FutsalPitch(PitchElement):
    def __init__(self, name: str, price: float) -> None:
        self.name  = name
        self.price = price

    def accept(self, visitor: "PitchVisitor") -> None:
        visitor.visit_futsal(self)

# Visitor interface
class PitchVisitor(ABC):
    @abstractmethod
    def visit_standard(self, pitch: StandardPitch) -> None: ...
    @abstractmethod
    def visit_premium(self, pitch: PremiumPitch) -> None: ...
    @abstractmethod
    def visit_futsal(self, pitch: FutsalPitch) -> None: ...

# Concrete visitors — each is a new "operation" added without modifying elements
class TaxCalculationVisitor(PitchVisitor):
    TAX_RATE = 0.14
    def __init__(self): self.total_tax = 0.0

    def visit_standard(self, p: StandardPitch) -> None:
        self.total_tax += p.price * self.TAX_RATE
        print(f"{p.name}: tax = {p.price * self.TAX_RATE:.2f}")

    def visit_premium(self, p: PremiumPitch) -> None:
        rate = self.TAX_RATE * 1.5 if p.has_vip_lounge else self.TAX_RATE
        self.total_tax += p.price * rate
        print(f"{p.name}: tax = {p.price * rate:.2f} (premium)")

    def visit_futsal(self, p: FutsalPitch) -> None:
        self.total_tax += p.price * self.TAX_RATE * 0.5  # reduced rate for futsal
        print(f"{p.name}: tax = {p.price * self.TAX_RATE * 0.5:.2f} (reduced)")

class AuditReportVisitor(PitchVisitor):
    def __init__(self): self.report = []

    def visit_standard(self, p: StandardPitch) -> None:
        self.report.append(f"STANDARD: {p.name} @ {p.price} EGP/hr")

    def visit_premium(self, p: PremiumPitch) -> None:
        features = " [VIP LOUNGE]" if p.has_vip_lounge else ""
        self.report.append(f"PREMIUM{features}: {p.name} @ {p.price} EGP/hr")

    def visit_futsal(self, p: FutsalPitch) -> None:
        self.report.append(f"FUTSAL: {p.name} @ {p.price} EGP/hr")

    def print_report(self) -> None:
        print("\n=== Audit Report ===")
        for line in self.report: print(line)

# Usage
pitches: list[PitchElement] = [
    StandardPitch("Field A", 200),
    PremiumPitch("VIP Field", 500, has_vip_lounge=True),
    FutsalPitch("Mini Court", 150),
]

# Apply tax calculation
tax_visitor = TaxCalculationVisitor()
for pitch in pitches: pitch.accept(tax_visitor)
print(f"Total tax: {tax_visitor.total_tax:.2f} EGP")

# Apply audit report — same elements, different visitor = different operation
audit = AuditReportVisitor()
for pitch in pitches: pitch.accept(audit)
audit.print_report()
# No changes to StandardPitch, PremiumPitch, FutsalPitch classes — Open/Closed!
```

---

## 34. Interpreter

### Intent
Given a language, define a representation for its grammar along with an **interpreter** that uses the representation to interpret sentences.

### Problem It Solves
You need to interpret expressions in a simple language or grammar (search queries, formula expressions, configuration DSLs, filtering rules).

```python
# Mini query language interpreter
# Query: "city:Cairo AND rating:>4 OR surface:futsal"

from abc import ABC, abstractmethod

class Expression(ABC):
    @abstractmethod
    def interpret(self, pitch: dict) -> bool: ...

class CityExpression(Expression):
    def __init__(self, city: str) -> None:
        self._city = city.lower()

    def interpret(self, pitch: dict) -> bool:
        return pitch.get("city", "").lower() == self._city

class SurfaceExpression(Expression):
    def __init__(self, surface: str) -> None:
        self._surface = surface.lower()

    def interpret(self, pitch: dict) -> bool:
        return pitch.get("surface", "").lower() == self._surface

class RatingExpression(Expression):
    def __init__(self, operator: str, value: float) -> None:
        self._op    = operator
        self._value = value

    def interpret(self, pitch: dict) -> bool:
        rating = pitch.get("rating", 0)
        if self._op == ">":  return rating > self._value
        if self._op == ">=": return rating >= self._value
        if self._op == "<":  return rating < self._value
        if self._op == "=":  return rating == self._value
        return False

class AndExpression(Expression):
    def __init__(self, left: Expression, right: Expression) -> None:
        self._left  = left
        self._right = right

    def interpret(self, pitch: dict) -> bool:
        return self._left.interpret(pitch) and self._right.interpret(pitch)

class OrExpression(Expression):
    def __init__(self, left: Expression, right: Expression) -> None:
        self._left  = left
        self._right = right

    def interpret(self, pitch: dict) -> bool:
        return self._left.interpret(pitch) or self._right.interpret(pitch)

class NotExpression(Expression):
    def __init__(self, expr: Expression) -> None:
        self._expr = expr

    def interpret(self, pitch: dict) -> bool:
        return not self._expr.interpret(pitch)

# Build and interpret: "Cairo AND rating > 4"
query = AndExpression(
    CityExpression("Cairo"),
    RatingExpression(">", 4.0)
)

pitches = [
    {"name": "Field A", "city": "Cairo",      "rating": 4.5, "surface": "artificial_turf"},
    {"name": "Field B", "city": "Giza",       "rating": 4.8, "surface": "natural_grass"},
    {"name": "Field C", "city": "Cairo",      "rating": 3.9, "surface": "futsal"},
]

results = [p for p in pitches if query.interpret(p)]
print([p["name"] for p in results])  # ['Field A']

# More complex: "(Cairo AND rating>4) OR futsal"
complex_query = OrExpression(
    AndExpression(CityExpression("Cairo"), RatingExpression(">", 4.0)),
    SurfaceExpression("futsal")
)
results2 = [p for p in pitches if complex_query.interpret(p)]
print([p["name"] for p in results2])  # ['Field A', 'Field C']
```

---

## 35. Design Patterns Interview Questions

**Q: What is the difference between Factory Method and Abstract Factory?**

Factory Method is about one product: a method in a class that creates objects, and subclasses decide which concrete type to create. Abstract Factory is about families of related products: an interface that creates multiple related objects that are meant to be used together. Use Factory Method when you have one product hierarchy. Use Abstract Factory when you have multiple product hierarchies that must remain consistent (e.g., a UI toolkit that creates Buttons and Dialogs from the same theme).

---

**Q: When would you use Observer vs Mediator?**

Both deal with communication between objects. Observer is for one-to-many: one subject, many observers that subscribe to be notified. It's for broadcasting state changes. Mediator is for many-to-many: multiple components that need to coordinate, but you don't want them knowing about each other. Use Observer for event systems, reactive programming, MVC (model notifying views). Use Mediator for chat rooms, flight control systems, form validation with interdependent fields.

---

**Q: What is the difference between Decorator and Proxy?**

Both wrap another object, but with different intent. Decorator adds behavior to an object transparently — the client knows it's using a Decorator and explicitly chooses to add behaviors (logging, caching). Proxy controls access to an object — the client might not even know it's using a proxy (lazy loading, authentication). Decorator focuses on enriching objects; Proxy focuses on controlling/protecting access.

---

**Q: When should you NOT use a Singleton?**

When it introduces hidden global state that makes testing difficult (you can't inject a test double). When it causes tight coupling — everything that uses the Singleton depends on its concrete type. When the "single instance" assumption might change (multi-tenancy, parallel tests). Prefer dependency injection: create the object once at startup and inject it everywhere. The object remains effectively a singleton in practice but can be easily replaced in tests.

---

**Q: What is the Strategy pattern and how does it differ from Template Method?**

Both define families of algorithms. Template Method uses inheritance: the algorithm skeleton is in the base class, and subclasses override steps. The algorithm structure is fixed at compile time. Strategy uses composition: the algorithm is encapsulated in a separate Strategy object, injected at runtime. Strategy is more flexible (swap algorithms at runtime) but slightly more verbose. Template Method is appropriate when the overall algorithm structure is stable. Strategy when you need runtime flexibility.

---

**Q: What problem does the Builder pattern solve that a constructor with defaults doesn't?**

Constructors with many parameters (especially optional ones) are unreadable — a call like `new Booking(userId, pitchId, start, end, null, null, true, false, "card", null)` is impossible to understand. Builder provides: a fluent API where each option is named (`.withNotes()`, `.recurring()`), prevents invalid states through validation in `build()`, makes optional fields explicit rather than null placeholders, and produces immutable objects when combined with a private constructor. In Python, keyword arguments solve some of this, but Builder adds the validation step and immutability guarantees.

---

## 📖 Master Resource List

- [Refactoring.Guru — Design Patterns](https://refactoring.guru/design-patterns) ← best visual explanations
- [Head First Design Patterns](https://www.oreilly.com/library/view/head-first-design/0596007124/) ← most approachable book
- [Design Patterns: Elements of Reusable OO Software (GoF)](https://www.oreilly.com/library/view/design-patterns-elements/0201633612/) ← the original
- [Clean Code — Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [SOLID Principles illustrated](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
- [Python MRO explained](https://www.python.org/download/releases/2.3/mro/)
- [C++ Virtual Inheritance](https://isocpp.org/wiki/faq/multiple-inheritance)
- [Java sealed classes (JEP 409)](https://openjdk.org/jeps/409)
- [SourceMaking — Design Patterns](https://sourcemaking.com/design_patterns)

---

*OOP is not about classes — it's about designing objects that collaborate. Master the patterns, and you'll recognize them everywhere. Go build great things, Ahmed. 🏗️*

> **Legend:** ✅ = correct pattern | ❌ = anti-pattern | → = leads to / transition
