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

## ✅ Benefits in Spring Context
- No need to manually implement Singleton logic (private constructor, static instance).  
- Spring manages lifecycle, thread-safety, and dependency injection.  
- Cleaner, testable code compared to manual Singleton implementations.

