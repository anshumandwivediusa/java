# Design Patterns

The key concept of a design pattern is that it provides a **proven, reusable, standardized solution to a recurring problem in software design**. Instead of reinventing the wheel, developers can apply these structured approaches to make systems more flexible, maintainable, and scalable.

## Design Patter Categories:

| **Category** | **Core Idea** | **Key Traits** | **Purpose** | **Examples** | **Focus** | **Benefits** | **Use Cases** | **Drawbacks** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **[Creational](ca://s?q=Creational_design_patterns)** | Object creation patterns | Flexible instantiation, encapsulation, reusable | Control object creation | Singleton, Factory, Builder | **How objects are created** | Encapsulation of instantiation | Database connections, configuration | Can add complexity |
| **[Structural](ca://s?q=Structural_design_patterns)** | Object composition patterns | Simplify relationships, scalable, extensible | Organize object structure | Adapter, Decorator, Composite | **How objects are composed** | Simplifies complex structures | UI frameworks, system APIs | May introduce overhead |
| **[Behavioral](ca://s?q=Behavioral_design_patterns)** | Object interaction patterns | Workflow control, communication, responsibility sharing | Define object communication | Observer, Strategy, Command | **How objects interact** | Improves collaboration & workflows | Event-driven systems, decision logic | Can be harder to debug |

## Creational Pattern

### 1. Singleton Design Pattern

- **Definition**  
  Ensures that a class has **only one instance (per JVM per classloader)** and provides a **global point of access** to it.

- **Core Idea**  
  Restrict instantiation to a single object while allowing controlled access.

- **Key Traits**  
  - Private constructor to prevent direct instantiation  
  - Static instance variable  
  - Public static method to provide access  

- **Purpose**  
  Manage shared resources or configurations consistently across the system.

- **Examples**  
  - Database connection manager  
  - Logger service  
  - Configuration settings  
  - Thread pools  

- **Focus**  
  *How to ensure only one instance exists* and is accessible globally.

- **Benefits**  
  - Controlled access to a single instance  
  - Reduced memory footprint for shared resources  
  - Consistency across the application  

- **Use Cases**  
  - Managing database connections  
  - Centralized logging  
  - Application-wide configuration  

- **Drawbacks**  
  - Can introduce hidden dependencies  
  - Difficult to unit test (tight coupling)  
  - May lead to global state issues  

#### Java Example (Thread-Safe Singleton)

```java
public class DatabaseConnection {
    // Private static instance
    private static volatile DatabaseConnection instance;

    // Private constructor
    private DatabaseConnection() {}

    // Public method to provide access
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

#### Singleton in Spring Boot

- **Default Behavior**  
  In Spring, all beans are **singleton-scoped by default**. That means when you annotate a class with `@Component`, `@Service`, or `@Repository`, Spring creates **one instance per ApplicationContext** and reuses it.

- **Using @Component**  
  ```java
  @Component
  public class LoggerService {
      public void log(String message) {
          System.out.println("LOG: " + message);
      }
  }
  ```
  - Spring ensures only one `LoggerService` instance exists in the context.

- **Using @Bean**  
  ```java
  @Configuration
  public class AppConfig {
      @Bean
      public DatabaseConnection databaseConnection() {
          return new DatabaseConnection();
      }
  }
  ```
  - The `@Bean` method returns a singleton-managed instance by default.

- **Explicit Scope Control**  
  If you want to override, you can use `@Scope("prototype")` for new instances each time. But for Singleton, the default is sufficient.

#### ✅ Benefits in Spring Context
- No need to manually implement Singleton logic (private constructor, static instance).  
- Spring manages lifecycle, thread-safety, and dependency injection.  
- Cleaner, testable code compared to manual Singleton implementations.

### 2. Factory Design Pattern

- **Definition**  
  Provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created. It encapsulates object creation logic.

- **Core Idea**  
  Delegate instantiation to a factory class/method instead of directly using `new`. This promotes loose coupling and flexibility.

- **Key Traits**  
  - Encapsulation of object creation  
  - Decouples client code from concrete classes  
  - Promotes reusability and scalability  

- **Purpose**  
  Control and centralize object creation, especially when dealing with complex or varying object types.

- **Examples**  
  - GUI frameworks (creating buttons, text fields depending on OS)  
  - Document parsers (XML, JSON, CSV)  
  - Notification services (Email, SMS, Push)  

- **Focus**  
  *How to create families of related objects without specifying their concrete classes.*

- **Benefits**  
  - Simplifies object creation logic  
  - Improves maintainability and scalability  
  - Encourages adherence to **Open/Closed Principle**  

- **Use Cases**  
  - When the exact type of object isn’t known until runtime  
  - When you want to centralize creation logic  
  - When multiple related classes share a common interface  

- **Drawbacks**  
  - Can introduce extra complexity  
  - May lead to too many factory classes if not managed well  

#### 💻 Java Example (Factory Method)

```java
// Product interface
interface Notification {
    void notifyUser();
}

// Concrete products
class EmailNotification implements Notification {
    public void notifyUser() {
        System.out.println("Sending an Email notification");
    }
}

class SMSNotification implements Notification {
    public void notifyUser() {
        System.out.println("Sending an SMS notification");
    }
}

// Factory
class NotificationFactory {
    public static Notification createNotification(String type) {
        if (type.equalsIgnoreCase("EMAIL")) {
            return new EmailNotification();
        } else if (type.equalsIgnoreCase("SMS")) {
            return new SMSNotification();
        }
        throw new IllegalArgumentException("Unknown notification type");
    }
}

// Client
public class FactoryPatternDemo {
    public static void main(String[] args) {
        Notification notification = NotificationFactory.createNotification("EMAIL");
        notification.notifyUser();
    }
}
```

Here’s how the **Factory Pattern** can be implemented in a **Spring Boot context** using Credit Card and Debit Card as examples:

##### 1. Define the Product Interface
```java
public interface Card {
    String getCardType();
    String getLimit();
}
```

##### 2. Create Concrete Implementations
```java
import org.springframework.stereotype.Component;

@Component
public class CreditCard implements Card {
    @Override
    public String getCardType() {
        return "Credit Card";
    }

    @Override
    public String getLimit() {
        return "Credit limit: $5000";
    }
}

@Component
public class DebitCard implements Card {
    @Override
    public String getCardType() {
        return "Debit Card";
    }

    @Override
    public String getLimit() {
        return "Linked to bank account balance";
    }
}
```

##### 3. Implement the Factory
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class CardFactory {

    private final CreditCard creditCard;
    private final DebitCard debitCard;

    @Autowired
    public CardFactory(CreditCard creditCard, DebitCard debitCard) {
        this.creditCard = creditCard;
        this.debitCard = debitCard;
    }

    public Card getCard(String type) {
        if ("CREDIT".equalsIgnoreCase(type)) {
            return creditCard;
        } else if ("DEBIT".equalsIgnoreCase(type)) {
            return debitCard;
        }
        throw new IllegalArgumentException("Unknown card type: " + type);
    }
}
```

##### 4. Use in a Service or Controller
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class CardService {

    private final CardFactory cardFactory;

    @Autowired
    public CardService(CardFactory cardFactory) {
        this.cardFactory = cardFactory;
    }

    public void showCardDetails(String type) {
        Card card = cardFactory.getCard(type);
        System.out.println("Card Type: " + card.getCardType());
        System.out.println("Limit: " + card.getLimit());
    }
}
```

## 3. Abstract Factory

#### Abstract Factory Pattern

- **Definition**  
  Provides an interface for creating **families of related or dependent objects** without specifying their concrete classes.

- **Core Idea**  
  Instead of creating individual objects directly, you use a factory that produces **factories** — each factory creates a set of related objects.

- **Key Traits**  
  - Encapsulates multiple factories  
  - Ensures consistency among related objects  
  - Promotes scalability and loose coupling  

- **Purpose**  
  To create families of related objects (e.g., UI components, payment methods) that must work together, without exposing instantiation details.

#### Java Example (Credit Card vs Debit Card Families)

```java
// Product interfaces
interface Card {
    String getCardType();
}

interface PaymentProcessor {
    void processPayment(double amount);
}

// Concrete products for Credit
class CreditCard implements Card {
    public String getCardType() { return "Credit Card"; }
}
class CreditPaymentProcessor implements PaymentProcessor {
    public void processPayment(double amount) {
        System.out.println("Processing credit payment of $" + amount);
    }
}

// Concrete products for Debit
class DebitCard implements Card {
    public String getCardType() { return "Debit Card"; }
}
class DebitPaymentProcessor implements PaymentProcessor {
    public void processPayment(double amount) {
        System.out.println("Processing debit payment of $" + amount);
    }
}

// Abstract Factory
interface CardFactory {
    Card createCard();
    PaymentProcessor createProcessor();
}

// Concrete Factories
class CreditCardFactory implements CardFactory {
    public Card createCard() { return new CreditCard(); }
    public PaymentProcessor createProcessor() { return new CreditPaymentProcessor(); }
}

class DebitCardFactory implements CardFactory {
    public Card createCard() { return new DebitCard(); }
    public PaymentProcessor createProcessor() { return new DebitPaymentProcessor(); }
}

// Client
public class AbstractFactoryDemo {
    public static void main(String[] args) {
        CardFactory factory = new CreditCardFactory(); // or new DebitCardFactory()
        Card card = factory.createCard();
        PaymentProcessor processor = factory.createProcessor();

        System.out.println("Created: " + card.getCardType());
        processor.processPayment(1000.0);
    }
}
```

#### Conceptual Notes

| **Aspect** | **Abstract Factory** |
|------------|-----------------------|
| **Scope** | Creates families of related objects |
| **Example** | CreditCard + CreditPaymentProcessor, DebitCard + DebitPaymentProcessor |
| **Benefit** | Ensures consistency among related objects |
| **Use Case** | UI themes (buttons, text fields), payment systems, cross-platform toolkits |
| **Drawback** | Can add complexity with multiple factories |


## 4. Builder Design Pattern

- **Definition**  
  Separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

- **Core Idea**  
  Step-by-step object creation where the client doesn’t need to know the details of how the object is built.

- **Key Traits**  
  - Encapsulation of construction logic  
  - Flexible and reusable for different object configurations  
  - Simplifies creation of complex objects  

- **Purpose**  
  To construct complex objects (with many optional parameters) in a controlled and readable way.

#### Java Example (Bank Account Builder)

```java
// Product
class BankAccount {
    private String accountNumber;
    private String owner;
    private boolean hasCreditCard;
    private boolean hasDebitCard;

    // Private constructor
    private BankAccount(Builder builder) {
        this.accountNumber = builder.accountNumber;
        this.owner = builder.owner;
        this.hasCreditCard = builder.hasCreditCard;
        this.hasDebitCard = builder.hasDebitCard;
    }

    // Static Builder class
    public static class Builder {
        private String accountNumber;
        private String owner;
        private boolean hasCreditCard;
        private boolean hasDebitCard;

        public Builder(String accountNumber, String owner) {
            this.accountNumber = accountNumber;
            this.owner = owner;
        }

        public Builder withCreditCard() {
            this.hasCreditCard = true;
            return this;
        }

        public Builder withDebitCard() {
            this.hasDebitCard = true;
            return this;
        }

        public BankAccount build() {
            return new BankAccount(this);
        }
    }

    @Override
    public String toString() {
        return "BankAccount [accountNumber=" + accountNumber +
               ", owner=" + owner +
               ", CreditCard=" + hasCreditCard +
               ", DebitCard=" + hasDebitCard + "]";
    }
}

// Client
public class BuilderPatternDemo {
    public static void main(String[] args) {
        BankAccount account = new BankAccount.Builder("12345", "Anshuman")
                                .withCreditCard()
                                .withDebitCard()
                                .build();

        System.out.println(account);
    }
}
```

#### 🌱 Builder Pattern in Spring Boot

##### 1. Using Lombok’s `@Builder`
Spring developers frequently use **Lombok** to reduce boilerplate code.

```java
import lombok.Builder;
import lombok.Data;

@Data
@Builder
public class BankAccount {
    private String accountNumber;
    private String owner;
    private boolean hasCreditCard;
    private boolean hasDebitCard;
}
```

##### 2. Creating Objects with Builder
```java
public class BuilderDemo {
    public static void main(String[] args) {
        BankAccount account = BankAccount.builder()
                .accountNumber("12345")
                .owner("Anshuman")
                .hasCreditCard(true)
                .hasDebitCard(true)
                .build();

        System.out.println(account);
    }
}
```

##### 3. Builder in a Spring Service
```java
import org.springframework.stereotype.Service;

@Service
public class AccountService {
    public BankAccount createPremiumAccount(String accountNumber, String owner) {
        return BankAccount.builder()
                .accountNumber(accountNumber)
                .owner(owner)
                .hasCreditCard(true)
                .hasDebitCard(true)
                .build();
    }
}
```

## Prototype Design Pattern

- **Definition**  
  Specifies the kinds of objects to create using a **prototype instance**, and creates new objects by **cloning** this prototype.

- **Core Idea**  
  Instead of instantiating new objects directly, you copy (clone) an existing object. This is useful when object creation is costly or complex.

- **Key Traits**  
  - Uses cloning instead of `new`  
  - Reduces overhead of creating complex objects  
  - Supports deep and shallow copies  

- **Purpose**  
  To create new objects by duplicating existing ones, ensuring efficiency and consistency.


#### 💻 Java Example (Card Prototype)

```java
// Prototype interface
interface Card extends Cloneable {
    Card clone();
    void showDetails();
}

// Concrete prototype: CreditCard
class CreditCard implements Card {
    private String limit;

    public CreditCard(String limit) {
        this.limit = limit;
    }

    @Override
    public Card clone() {
        return new CreditCard(this.limit);
    }

    @Override
    public void showDetails() {
        System.out.println("Credit Card with limit: " + limit);
    }
}

// Concrete prototype: DebitCard
class DebitCard implements Card {
    private String linkedAccount;

    public DebitCard(String linkedAccount) {
        this.linkedAccount = linkedAccount;
    }

    @Override
    public Card clone() {
        return new DebitCard(this.linkedAccount);
    }

    @Override
    public void showDetails() {
        System.out.println("Debit Card linked to account: " + linkedAccount);
    }
}

// Client
public class PrototypeDemo {
    public static void main(String[] args) {
        CreditCard credit = new CreditCard("$5000");
        CreditCard clonedCredit = (CreditCard) credit.clone();

        DebitCard debit = new DebitCard("Account#12345");
        DebitCard clonedDebit = (DebitCard) debit.clone();

        credit.showDetails();
        clonedCredit.showDetails();

        debit.showDetails();
        clonedDebit.showDetails();
    }
}
```




Got it — let’s connect the **Adapter design pattern** to the examples we’ve been discussing (inner classes, composition, aggregation).  

---

## 🔑 Adapter Pattern Recap
- **Adapter** is a **structural design pattern**.  
- It allows incompatible interfaces to work together by acting as a **bridge**.  
- Think of it as: *“I have a plug that doesn’t fit the socket, so I use an adapter.”*  

---

## 📝 Example with Composition (strong relationship)
Suppose we have a legacy `LineItem` class, but our new system expects an `OrderItem` interface. Instead of rewriting everything, we use an **Adapter**.

```java
// Target interface expected by new system
interface OrderItem {
    String getProductName();
    int getQuantity();
}

// Legacy class (incompatible interface)
class LineItem {
    private String product;
    private int qty;

    public LineItem(String product, int qty) {
        this.product = product;
        this.qty = qty;
    }

    public String getItemName() { return product; }
    public int getItemCount() { return qty; }
}

// Adapter class using composition
class LineItemAdapter implements OrderItem {
    private LineItem lineItem; // composition

    public LineItemAdapter(LineItem lineItem) {
        this.lineItem = lineItem;
    }

    @Override
    public String getProductName() {
        return lineItem.getItemName();
    }

    @Override
    public int getQuantity() {
        return lineItem.getItemCount();
    }
}

public class AdapterDemo {
    public static void main(String[] args) {
        LineItem legacyItem = new LineItem("Laptop", 2);

        // Adapt legacy object to new interface
        OrderItem adapted = new LineItemAdapter(legacyItem);

        System.out.println(adapted.getProductName() + " x " + adapted.getQuantity());
    }
}
```


## ✅ Notes on Design
- The **Adapter** uses **composition** (`LineItemAdapter` contains a `LineItem`).  
- This makes the relationship strong: the adapter *owns* the legacy object.  
- The adapter translates method calls from the new interface (`OrderItem`) into the old one (`LineItem`).  
- This avoids rewriting legacy code and provides **polymorphism** in the new system.  

---

Here are **section notes on the Decorator Design Pattern**, explained in a simple and practical way:

---

## 🎨 Decorator Design Pattern

- **Definition**  
  Allows you to **add new behavior or responsibilities** to objects dynamically, without modifying their existing code.

- **Core Idea**  
  Wrap an object inside another object (the decorator) that adds extra functionality.  

- **Key Traits**  
  - Follows **composition over inheritance**  
  - Enhances objects at runtime  
  - Keeps original class unchanged  

- **Purpose**  
  To extend functionality of objects flexibly, without creating endless subclasses.

---

### 💻 Java Example (Payment Decorator)

```java
// Component interface
interface Payment {
    void pay(double amount);
}

// Concrete component
class CreditCardPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using Credit Card");
    }
}

// Decorator base class
abstract class PaymentDecorator implements Payment {
    protected Payment decoratedPayment;

    public PaymentDecorator(Payment decoratedPayment) {
        this.decoratedPayment = decoratedPayment;
    }

    @Override
    public void pay(double amount) {
        decoratedPayment.pay(amount);
    }
}

// Concrete decorator: adds logging
class LoggingPaymentDecorator extends PaymentDecorator {
    public LoggingPaymentDecorator(Payment decoratedPayment) {
        super(decoratedPayment);
    }

    @Override
    public void pay(double amount) {
        System.out.println("[LOG] Payment started...");
        super.pay(amount);
        System.out.println("[LOG] Payment finished.");
    }
}

// Client
public class DecoratorDemo {
    public static void main(String[] args) {
        Payment payment = new LoggingPaymentDecorator(new CreditCardPayment());
        payment.pay(1000.0);
    }
}
```

---

## 📊 Conceptual Notes

| **Aspect** | **Decorator Pattern** |
|------------|------------------------|
| **Scope** | Adds responsibilities dynamically |
| **Example** | Logging added to CreditCardPayment |
| **Benefit** | Flexible, reusable, avoids subclass explosion |
| **Use Case** | Logging, encryption, compression, validation |
| **Drawback** | Can lead to many small classes (complexity) |

---

Here are **section notes on the Facade Design Pattern**, explained in a simple and practical way:

---

## 🏢 Facade Design Pattern

- **Definition**  
  Provides a **unified, simplified interface** to a set of complex subsystems.  

- **Core Idea**  
  Instead of dealing with multiple complicated classes, the client interacts with a single **Facade class** that delegates work internally.  

- **Key Traits**  
  - Simplifies usage of complex systems  
  - Hides internal details from the client  
  - Promotes loose coupling  

- **Purpose**  
  To make a system easier to use by exposing only the necessary functionality through a single entry point.

---

### 💻 Java Example (Banking System Facade)

```java
// Subsystems
class AccountService {
    public void createAccount(String owner) {
        System.out.println("Account created for: " + owner);
    }
}

class PaymentService {
    public void makePayment(double amount) {
        System.out.println("Payment of $" + amount + " processed");
    }
}

class NotificationService {
    public void sendNotification(String message) {
        System.out.println("Notification: " + message);
    }
}

// Facade
class BankingFacade {
    private AccountService accountService;
    private PaymentService paymentService;
    private NotificationService notificationService;

    public BankingFacade() {
        this.accountService = new AccountService();
        this.paymentService = new PaymentService();
        this.notificationService = new NotificationService();
    }

    public void openAccountAndPay(String owner, double amount) {
        accountService.createAccount(owner);
        paymentService.makePayment(amount);
        notificationService.sendNotification("Account opened and payment done for " + owner);
    }
}

// Client
public class FacadeDemo {
    public static void main(String[] args) {
        BankingFacade facade = new BankingFacade();
        facade.openAccountAndPay("Anshuman", 1000.0);
    }
}
```

---

## 📊 Conceptual Notes

| **Aspect** | **Facade Pattern** |
|------------|---------------------|
| **Scope** | Simplifies interaction with complex subsystems |
| **Example** | BankingFacade wraps Account, Payment, Notification services |
| **Benefit** | Cleaner client code, hides complexity |
| **Use Case** | APIs, libraries, frameworks, microservices |
| **Drawback** | May hide too much functionality if not designed carefully |

---

Here are **section notes on the Bridge Design Pattern**, explained in a simple and practical way:

---

## 🌉 Bridge Design Pattern

- **Definition**  
  Decouples an abstraction from its implementation so that the two can vary independently.  

- **Core Idea**  
  Instead of binding an abstraction tightly to one implementation, you “bridge” them with composition. This allows you to change either side without affecting the other.  

- **Key Traits**  
  - Promotes flexibility by separating abstraction and implementation  
  - Uses composition rather than inheritance  
  - Makes code easier to extend  

- **Purpose**  
  To avoid a rigid hierarchy and allow both abstraction and implementation to evolve separately.

---

### 💻 Java Example (Payment System)

```java
// Implementor interface
interface PaymentSystem {
    void processPayment(double amount);
}

// Concrete implementors
class UPPaymentSystem implements PaymentSystem {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing payment via UPI: $" + amount);
    }
}

class CreditCardPaymentSystem implements PaymentSystem {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing payment via Credit Card: $" + amount);
    }
}

// Abstraction
abstract class Payment {
    protected PaymentSystem paymentSystem;

    public Payment(PaymentSystem paymentSystem) {
        this.paymentSystem = paymentSystem;
    }

    public abstract void makePayment(double amount);
}

// Refined Abstraction
class OnlinePayment extends Payment {
    public OnlinePayment(PaymentSystem paymentSystem) {
        super(paymentSystem);
    }

    @Override
    public void makePayment(double amount) {
        System.out.println("Initiating online payment...");
        paymentSystem.processPayment(amount);
    }
}

// Client
public class BridgeDemo {
    public static void main(String[] args) {
        Payment onlinePayment1 = new OnlinePayment(new UPPaymentSystem());
        onlinePayment1.makePayment(500.0);

        Payment onlinePayment2 = new OnlinePayment(new CreditCardPaymentSystem());
        onlinePayment2.makePayment(1000.0);
    }
}
```


## 📊 Conceptual Notes

| **Aspect** | **Bridge Pattern** |
|------------|---------------------|
| **Scope** | Separates abstraction from implementation |
| **Example** | Payment abstraction + UPI/CreditCard implementations |
| **Benefit** | Both sides can evolve independently |
| **Use Case** | Payment systems, device drivers, UI themes |
| **Drawback** | Adds extra layers, may feel complex for small systems |


Here’s a clear and practical explanation of the **Observer Design Pattern**:

---

## 👀 Observer Design Pattern

- **Definition**  
  Defines a **one-to-many dependency** between objects so that when one object (the subject) changes state, all its dependents (observers) are notified automatically.

- **Core Idea**  
  Instead of constantly checking for changes, observers “subscribe” to a subject and get updates when something changes.

- **Key Traits**  
  - Subject maintains a list of observers  
  - Observers register/unregister themselves  
  - Subject notifies observers when state changes  

- **Purpose**  
  To implement event-driven systems where multiple objects need to react to changes in another object.

---

### 💻 Java Example (Bank Account Notifications)

```java
// Observer interface
interface Observer {
    void update(String message);
}

// Concrete observers
class EmailNotifier implements Observer {
    @Override
    public void update(String message) {
        System.out.println("Email notification: " + message);
    }
}

class SMSNotifier implements Observer {
    @Override
    public void update(String message) {
        System.out.println("SMS notification: " + message);
    }
}

// Subject
class BankAccount {
    private List<Observer> observers = new ArrayList<>();

    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    public void deposit(double amount) {
        System.out.println("Deposited $" + amount);
        notifyObservers("Deposit of $" + amount + " successful.");
    }

    private void notifyObservers(String message) {
        for (Observer observer : observers) {
            observer.update(message);
        }
    }
}

// Client
public class ObserverDemo {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();

        account.addObserver(new EmailNotifier());
        account.addObserver(new SMSNotifier());

        account.deposit(1000.0);
    }
}
```

## 📊 Conceptual Notes

| **Aspect** | **Observer Pattern** |
|------------|-----------------------|
| **Scope** | One-to-many dependency |
| **Example** | BankAccount notifying Email/SMS observers |
| **Benefit** | Decouples subject from observers, flexible event handling |
| **Use Case** | Event listeners, UI frameworks, messaging systems |
| **Drawback** | Can lead to unexpected updates if too many observers |




