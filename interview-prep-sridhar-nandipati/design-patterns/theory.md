# Design Patterns Theory

This document covers common design patterns, their intents, use cases, and trade-offs, relevant for senior Java engineering interviews.

## Introduction to Design Patterns

### What are Design Patterns?
Design patterns are reusable, well-documented solutions to commonly occurring problems within a given context in software design. They are not specific algorithms or code, but rather general concepts, templates, or blueprints that describe how to structure classes and objects to solve a particular design problem. They represent best practices evolved over time by experienced software developers.

### Why use them? (Benefits)
*   **Reusability:** Patterns provide proven solutions that can be applied in many different situations, saving time and effort by avoiding reinvention of the wheel.
*   **Communication:** They establish a common vocabulary among developers. Saying "Let's use a Singleton for the configuration manager" is more concise and clearer than describing the entire structure.
*   **Proven Solutions:** Design patterns are typically time-tested and well-vetted, leading to more robust and reliable designs. They help avoid common pitfalls.
*   **Maintainability & Readability:** Code based on well-known patterns is often easier to understand, maintain, and debug because other developers familiar with the patterns can quickly grasp the design.
*   **Flexibility & Extensibility:** Many patterns are designed to make systems more flexible and easier to extend or modify in the future.

### Categories
Design patterns are typically categorized into three main groups based on their purpose:

1.  **Creational Patterns:** Deal with object creation mechanisms, trying to create objects in a manner suitable to the situation. They provide flexibility by decoupling the client from the actual creation process of the objects it needs.
    *   Examples: Singleton, Factory Method, Abstract Factory, Builder, Prototype.
2.  **Structural Patterns:** Deal with object composition or the relationships between entities. They describe ways to assemble objects and classes into larger structures while keeping these structures flexible and efficient.
    *   Examples: Adapter, Decorator, Facade, Proxy, Composite, Flyweight, Bridge.
3.  **Behavioral Patterns:** Deal with algorithms and the assignment of responsibilities between objects. They describe how objects interact and distribute responsibility.
    *   Examples: Strategy, Observer, Template Method, Chain of Responsibility, Command, State, Iterator, Mediator, Visitor, Memento.

## Creational Patterns

### Singleton
*   **Intent:** Ensure a class only has one instance, and provide a global point of access to it.
*   **Structure:**
    *   A private static variable to hold the single instance of the class.
    *   A private constructor to prevent instantiation from outside the class.
    *   A public static method (e.g., `getInstance()`) that returns the single instance (creating it if it doesn't exist).
*   **Use Cases:**
    *   **Logging:** A single logger instance for the entire application.
    *   **Configuration Manager:** A single object to hold and provide access to application configuration settings.
    *   **Database Connection Pool:** Managing a pool of database connections.
    *   **Hardware Access:** Providing synchronized access to a shared hardware resource.
    *   Runtime environments or system services like Spring Framework's ApplicationContext (beans are often singletons by default).
*   **Pros:**
    *   Guaranteed single instance.
    *   Global point of access.
    *   Lazy initialization is possible (instance created on first use).
*   **Cons:**
    *   Violates the Single Responsibility Principle (manages its own creation and lifecycle, plus its business logic).
    *   Can make unit testing difficult (global state, hard to mock).
    *   Can lead to tight coupling if overused.
    *   Concurrency issues if not implemented carefully in multi-threaded environments.
*   **Thread-Safety Considerations:**
    *   **Eager Initialization:** `private static final Singleton instance = new Singleton();` This is thread-safe as the instance is created when the class is loaded.
    *   **Synchronized `getInstance()` method:** `public static synchronized Singleton getInstance() { ... }` This is thread-safe but can have performance implications due to synchronization on every call.
    *   **Double-Checked Locking (DCL):**
        ```java
        private static volatile Singleton instance;
        public static Singleton getInstance() {
            if (instance == null) { // First check (no lock)
                synchronized (Singleton.class) {
                    if (instance == null) { // Second check (with lock)
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
        ```
        The `volatile` keyword is crucial here to prevent issues due to instruction reordering by the compiler or CPU. DCL can be tricky to get right and is often debated.
    *   **Initialization-on-demand Holder Idiom (Bill Pugh Singleton):**
        ```java
        private static class SingletonHolder {
            private static final Singleton INSTANCE = new Singleton();
        }
        public static Singleton getInstance() {
            return SingletonHolder.INSTANCE;
        }
        ```
        This is generally considered the preferred approach for thread-safe lazy initialization in Java. It leverages the JVM's class loading mechanism. The inner class `SingletonHolder` is not loaded until `getInstance()` is called.
    *   **Enum Singleton:**
        ```java
        public enum EnumSingleton {
            INSTANCE;
            // methods...
        }
        ```
        This is concise, provides built-in serialization safety, and is inherently thread-safe. Often recommended as the best way to implement a Singleton in Java if you don't need to extend any class.

> **TODO:** How has Spring's default singleton scope for beans simplified (or complicated) managing shared resources in your microservices at Intralinks or TOPS? Discuss any thread-safety concerns you had to address with Spring singletons.
**Sample Answer/Experience:**
"In my Spring Boot projects at Intralinks and TOPS, the default singleton scope for beans was generally a huge simplification for managing shared resources like service clients (e.g., `RestTemplate`, AWS SDK clients) or configuration components. Spring manages the lifecycle, ensuring there's only one instance, which is what we usually want for these. For example, an `AwsS3Client` bean configured once and injected wherever needed is efficient.
However, the main complication arises with statefulness. If a singleton bean holds mutable state that can be modified by multiple concurrent requests (as is typical in web applications), we absolutely have to ensure thread safety. For instance, if a singleton service bean had an instance variable like a `HashMap` used as a temporary cache, concurrent requests could lead to race conditions. To address this, we'd typically:
1.  Ensure such beans are stateless if possible.
2.  If state is necessary, use thread-safe collections like `ConcurrentHashMap`.
3.  Use `synchronized` blocks or methods for critical sections modifying shared state, though this can become a bottleneck and is used sparingly.
4.  Prefer request-scoped or prototype-scoped beans if the state is specific to a single request or needs to be independent.
The Bill Pugh or enum singleton patterns are great for manual singleton creation, but with Spring, the framework handles it. The key is being mindful of the implications of shared, mutable state in a concurrent environment, even with Spring's singleton management."

### Factory Method
*   **Intent:** Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.
*   **Structure:**
    *   `Product` (interface or abstract class): Defines the interface for objects the factory method creates.
    *   `ConcreteProduct` (class): Implements the `Product` interface.
    *   `Creator` (abstract class or interface): Declares the factory method (`factoryMethod()`), which returns an object of type `Product`. May also define methods that use the product.
    *   `ConcreteCreator` (class): Overrides the `factoryMethod()` to return an instance of a `ConcreteProduct`.
*   **Use Cases:**
    *   When a class cannot anticipate the class of objects it must create.
    *   When a class wants its subclasses to specify the objects it creates.
    *   To provide users of a library or framework with a way to extend its internal components. (e.g., `java.util.Calendar.getInstance()`, `java.text.NumberFormat.getInstance()`).
    *   Decoupling client code from concrete classes.
*   **Pros:**
    *   Promotes loose coupling by avoiding binding application-specific classes into the code.
    *   Subclasses can provide different implementations for the product.
    *   Follows Single Responsibility Principle (moves object creation logic into one place).
    *   Follows Open/Closed Principle (can introduce new product types without modifying client code).
*   **Cons:**
    *   Can lead to a proliferation of subclasses if many product types are needed.
    *   Clients might have to subclass the Creator class just to create a particular ConcreteProduct instance.

> **TODO:** In the TOPS project, you processed various financial data feeds via Kafka. Can you describe a scenario where a Factory Method could have been used to create different types of data parsers or processors based on message type or source?
**Sample Answer/Experience:**
"In the TOPS project, we received financial data from multiple sources via Kafka, and each source or message type (e.g., FIX protocol, custom CSV, XML) required a specific parser and then a specific processor. A Factory Method pattern would fit well here.
We could have an abstract `DataProcessorCreator` class:
```java
// Product interface
interface DataParser {
    FinancialRecord parse(String message);
}
// ConcreteProducts
class FixParser implements DataParser { /* ... */ }
class CsvParser implements DataParser { /* ... */ }

// Creator abstract class
abstract class MessageHandler {
    // The factory method
    protected abstract DataParser createParser(String messageType);

    public FinancialRecord processMessage(String rawMessage, String messageType) {
        DataParser parser = createParser(messageType); // Factory method call
        FinancialRecord record = parser.parse(rawMessage);
        // ... common processing steps ...
        return record;
    }
}

// ConcreteCreators
class FixMessageHandler extends MessageHandler {
    @Override
    protected DataParser createParser(String messageType) {
        // Could also inspect messageType if needed, though for FIX it's specific
        return new FixParser();
    }
}

class CsvMessageHandler extends MessageHandler {
    @Override
    protected DataParser createParser(String messageType) {
        return new CsvParser(); // Potentially configure CsvParser based on messageType if CSV formats vary
    }
}
```
In this setup, the `MessageHandler` class knows it needs a `DataParser` but doesn't know the concrete type. Subclasses like `FixMessageHandler` or `CsvMessageHandler` decide which specific parser to instantiate. The Kafka consumer logic could then instantiate the appropriate `MessageHandler` based on the Kafka topic or a header in the message, and then call `processMessage`. This decouples the main consumer logic from the specific parsing implementations and makes it easy to add new message types by adding new `DataParser` implementations and corresponding `MessageHandler` subclasses, adhering to the Open/Closed Principle."

### Abstract Factory
*   **Intent:** Provide an interface for creating families of related or dependent objects without specifying their concrete classes.
*   **Structure:**
    *   `AbstractFactory` (interface or abstract class): Declares methods for creating each type of abstract product (e.g., `createProductA()`, `createProductB()`).
    *   `ConcreteFactory` (class): Implements the `AbstractFactory` interface to create a specific family of concrete products. There can be multiple `ConcreteFactory` classes.
    *   `AbstractProduct` (interface or abstract class): Declares the interface for a type of product object (e.g., `ProductA`, `ProductB`).
    *   `ConcreteProduct` (class): Implements an `AbstractProduct` interface and defines a product object to be created by the corresponding `ConcreteFactory`.
    *   `Client`: Uses only the `AbstractFactory` and `AbstractProduct` interfaces.
*   **Use Cases:**
    *   When a system needs to be independent of how its products are created, composed, and represented.
    *   When a system needs to be configured with one of multiple families of products.
    *   When a family of related product objects is designed to be used together, and you need to enforce this constraint.
    *   Example: UI toolkits (creating buttons, windows, scrollbars for different operating systems like Windows, macOS, Linux). A `WindowsFactory` would create `WindowsButton`, `WindowsWindow`, while a `MacFactory` would create `MacButton`, `MacWindow`.
    *   Database abstraction layers (e.g., creating `Connection`, `Command`, `DataReader` objects for different database systems like SQL Server, Oracle, MySQL).
*   **Pros:**
    *   Isolates concrete classes: The client code works with abstract interfaces, decoupling it from specific implementations.
    *   Makes exchanging product families easy: You can switch `ConcreteFactory` instances to change the behavior of the application.
    *   Promotes consistency among products: Ensures that products from the same family are used together.
*   **Cons:**
    *   Difficult to add new kinds of products: The `AbstractFactory` interface and all its subclasses must be changed if a new product type is introduced. This violates the Open/Closed Principle.
    *   Can lead to many classes if there are many product families and product types.

> **TODO:** Imagine designing a UI component library for AEM at Herc Rentals that needs to support different themes (e.g., "Standard Herc" vs. "Compact Herc"). How would an Abstract Factory help create families of related UI elements (like buttons, text fields, date pickers) for each theme?
**Sample Answer/Experience:**
"For a themable UI component library in AEM at Herc Rentals, an Abstract Factory would be very suitable. Each theme (e.g., 'Standard Herc', 'Compact Herc') would represent a family of related UI elements.
The structure would be:
1.  **AbstractProduct Interfaces:** `Button`, `TextField`, `DatePicker`.
2.  **ConcreteProduct Classes:**
    *   For 'Standard Herc' theme: `StandardButton`, `StandardTextField`, `StandardDatePicker`.
    *   For 'Compact Herc' theme: `CompactButton`, `CompactTextField`, `CompactDatePicker`.
3.  **AbstractFactory Interface:**
    ```java
    interface UIComponentFactory {
        Button createButton(String label);
        TextField createTextField(String initialValue);
        DatePicker createDatePicker();
    }
    ```
4.  **ConcreteFactory Classes:**
    ```java
    class StandardHercThemeFactory implements UIComponentFactory {
        public Button createButton(String label) { return new StandardButton(label); }
        public TextField createTextField(String val) { return new StandardTextField(val); }
        public DatePicker createDatePicker() { return new StandardDatePicker(); }
    }

    class CompactHercThemeFactory implements UIComponentFactory {
        public Button createButton(String label) { return new CompactButton(label); }
        public TextField createTextField(String val) { return new CompactTextField(val); }
        public DatePicker createDatePicker() { return new CompactDatePicker(); }
    }
    ```
When an AEM component (like a form component) needs to render itself, it would first obtain the appropriate factory based on the currently active theme (perhaps determined from a page property or OSGi configuration).
```java
// Client code within an AEM component (e.g., a Sling Model or WCMUsePojo)
// Assume 'currentThemeFactory' is obtained based on context
UIComponentFactory factory = getCurrentThemeFactory(); // This could return StandardHercThemeFactory or CompactHercThemeFactory

Button submitButton = factory.createButton("Submit");
TextField nameField = factory.createTextField("");
// ... use these abstract product types to render ...
submitButton.render();
nameField.render();
```
This ensures that all UI elements within that component belong to the same theme, providing consistency. If we wanted to introduce a new theme, say 'Mobile Herc', we'd create a new set of concrete product classes and a new `MobileHercThemeFactory`. The client code in the AEM components wouldn't need to change, as it only programs against the `UIComponentFactory` and `Button`, `TextField` interfaces. The main challenge, as typical with Abstract Factory, is if we need to add a *new type* of UI element (e.g., `Checkbox`); that would require updating the `UIComponentFactory` interface and all its concrete factory implementations."

### Builder
*   **Intent:** Separate the construction of a complex object from its representation so that the same construction process can create different representations. Allows step-by-step construction of complex objects.
*   **Structure:**
    *   `Builder` (interface or abstract class): Specifies methods for creating parts of the `Product` object.
    *   `ConcreteBuilder` (class): Implements the `Builder` interface and provides specific implementations for the steps. It keeps track of the representation it creates and provides a method for retrieving the result.
    *   `Product` (class): The complex object being built.
    *   `Director` (class, optional): Constructs an object using the `Builder` interface. It defines the order in which to execute building steps. The client can act as a director directly.
*   **Use Cases:**
    *   When the algorithm for creating a complex object should be independent of the parts that make up the object and how they're assembled.
    *   When the construction process must allow different representations for the object that's constructed.
    *   To construct objects that have many optional parameters or require a specific sequence of initialization steps.
    *   Creating immutable objects with many fields (e.g., `StringBuilder` for `String`, `java.time.LocalDateTime.Builder`).
    *   Fluent APIs (e.g., `new User.UserBuilder("John", "Doe").age(30).phone("1234567").build();`).
*   **Pros:**
    *   Allows you to vary a product's internal representation.
    *   Encapsulates code for construction and representation.
    *   Provides better control over the construction process (step-by-step).
    *   Improves readability when creating objects with many parameters (avoids telescoping constructors).
    *   Can be used to create immutable objects easily.
*   **Cons:**
    *   Requires creating a separate `Builder` for each different type of `Product`.
    *   Can be more verbose than other creational patterns if the object is simple.
    *   The `Director` class might be unnecessary for simple constructions.

> **TODO:** You've built Spring Boot microservices. Describe how you've used the Builder pattern to construct complex DTOs (Data Transfer Objects) for REST API request/response bodies, especially when objects have many optional fields or need to be immutable.
**Sample Answer/Experience:**
"In several Spring Boot microservices at Intralinks, especially those handling complex financial operations, our DTOs for REST APIs often had numerous fields, many of which were optional. Using telescoping constructors was unwieldy and error-prone. The Builder pattern was our standard solution here.

For example, a `TradeExecutionRequest` DTO might look like:
```java
public class TradeExecutionRequest {
    private final String tradeId; // required
    private final String instrumentId; // required
    private final double quantity; // required
    private final String orderType; // e.g., "MARKET", "LIMIT", required
    private final Double limitPrice; // optional
    private final String currency; // optional, might default
    private final String clientNotes; // optional
    // ... other optional fields ...

    // Private constructor, only called by Builder
    private TradeExecutionRequest(Builder builder) {
        this.tradeId = builder.tradeId;
        this.instrumentId = builder.instrumentId;
        this.quantity = builder.quantity;
        this.orderType = builder.orderType;
        this.limitPrice = builder.limitPrice;
        this.currency = builder.currency;
        this.clientNotes = builder.clientNotes;
    }

    // Getters only, making the DTO immutable

    public static class Builder {
        private final String tradeId;
        private final String instrumentId;
        private final double quantity;
        private final String orderType;

        private Double limitPrice;
        private String currency = "USD"; // Default value
        private String clientNotes;

        public Builder(String tradeId, String instrumentId, double quantity, String orderType) {
            this.tradeId = tradeId;
            this.instrumentId = instrumentId;
            this.quantity = quantity;
            this.orderType = orderType;
        }

        public Builder limitPrice(Double limitPrice) {
            this.limitPrice = limitPrice;
            return this;
        }

        public Builder currency(String currency) {
            this.currency = currency;
            return this;
        }

        public Builder clientNotes(String clientNotes) {
            this.clientNotes = clientNotes;
            return this;
        }

        public TradeExecutionRequest build() {
            // Can add validation logic here before creating the object
            if (orderType.equals("LIMIT") && limitPrice == null) {
                throw new IllegalStateException("Limit price cannot be null for LIMIT orders.");
            }
            return new TradeExecutionRequest(this);
        }
    }
}
```
Usage in a service or controller:
```java
TradeExecutionRequest request = new TradeExecutionRequest.Builder("T123", "ISIN456", 100.0, "LIMIT")
                                    .limitPrice(150.75)
                                    .clientNotes("Client VIP special handling")
                                    .build();
```
**Benefits we saw:**
1.  **Readability:** It's very clear which field is being set, especially with many optional parameters.
2.  **Immutability:** The `TradeExecutionRequest` object is immutable once constructed, which is great for thread safety and predictable state.
3.  **Required vs. Optional:** Required fields are enforced through the Builder's constructor.
4.  **Flexibility:** Fields can be set in any order. Default values are easily handled in the builder.
5.  **Validation:** The `build()` method in the builder is a perfect place to put validation logic before the actual object is created.
This made our API contracts much cleaner and less error-prone to use, both for clients calling our services and for us internally when creating these objects."

### Prototype
*   **Intent:** Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.
*   **Structure:**
    *   `Prototype` (interface or abstract class): Declares a `clone()` method.
    *   `ConcretePrototype` (class): Implements the `clone()` method to copy itself.
    *   `Client`: Creates a new object by asking a prototype to clone itself.
*   **Use Cases:**
    *   When the classes to instantiate are specified at runtime (e.g., by dynamic loading).
    *   To avoid building a class hierarchy of factories that parallels the class hierarchy of products.
    *   When instances of a class can have one of only a few different combinations of state. It may be more convenient to install a corresponding number of prototypes and clone them rather than instantiating the class manually each time.
    *   Performance benefits if object creation is expensive and cloning is cheaper.
    *   Example: Creating game objects (e.g., enemies) with slightly different properties by cloning a base enemy prototype.
*   **Pros:**
    *   Adding and removing products at runtime: Clients can install and remove prototypes at runtime.
    *   Specifying new objects by varying values: Can create new objects by changing the values of a prototype's instance variables.
    *   Specifying new objects by varying structure: Can create new objects by changing the structure of a prototype (e.g., by composing it with other objects).
    *   Reduced subclassing compared to Factory Method.
    *   Can be more efficient than creating objects from scratch if initialization is costly.
*   **Cons:**
    *   Cloning complex objects that have circular references or refer to non-cloneable resources can be tricky.
    *   Each subclass of `Prototype` must implement the `clone()` operation, which can be difficult if the classes under consideration already exist.
    *   Requires careful implementation of `clone()` (deep vs. shallow copy).
*   **Shallow vs. Deep Copy:**
    *   **Shallow Copy:** Copies the main object, but referenced objects are shared between the original and the clone. If a referenced object is modified, the change is visible in both. `Object.clone()` provides a shallow copy by default.
    *   **Deep Copy:** Copies the main object and recursively copies all objects it references. The clone is completely independent of the original. Requires custom implementation of `clone()`.

## Structural Patterns

### Adapter (Wrapper)
*   **Intent:** Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.
*   **Structure:**
    *   `Target` (interface): Defines the domain-specific interface that `Client` uses.
    *   `Client`: Collaborates with objects conforming to the `Target` interface.
    *   `Adaptee` (class): Defines an existing interface that needs adapting.
    *   `Adapter` (class): Implements the `Target` interface and holds an instance of (or inherits from) the `Adaptee`. It translates requests from the `Client` (via `Target` interface) into requests to the `Adaptee`.
    *   Two main types:
        *   **Object Adapter:** Uses composition (holds an instance of `Adaptee`).
        *   **Class Adapter:** Uses multiple inheritance (inherits from both `Target` (often an interface) and `Adaptee` (a class)). Less common in Java due to lack of multiple class inheritance (can be achieved by implementing Target interface and extending Adaptee class).
*   **Use Cases:**
    *   When you want to use an existing class, and its interface does not match the one you need.
    *   When you want to create a reusable class that cooperates with unrelated or unforeseen classes, i.e., classes that don't necessarily have compatible interfaces.
    *   (Object adapter) when you want to use several existing subclasses, but it's impractical to adapt their interface by subclassing every one. An object adapter can adapt the interface of its parent class.
    *   Example: `java.io.InputStreamReader` adapts an `InputStream` (byte stream) to a `Reader` (character stream). `java.util.Arrays.asList()` adapts an array to a `List`.
*   **Pros:**
    *   Allows incompatible interfaces to work together.
    *   Enhances reusability of existing code.
    *   (Object Adapter) Allows a single Adapter to work with many Adaptees (the Adaptee itself and all of its subclasses if the Adaptee is a class).
*   **Cons:**
    *   (Class Adapter) Doesn't work when we want to adapt a class and all its subclasses, because it commits to one specific Adaptee class.
    *   Can introduce an extra level of indirection, which might have a small performance impact.
    *   Can make the code more complex by adding new classes.

> **TODO:** In the Intralinks/BOFA project, you likely had to integrate with various existing banking systems or third-party financial APIs. Describe a situation where the Adapter pattern was essential to bridge differences between your system's required interface and an external system's provided interface.
**Sample Answer/Experience:**
"At Intralinks/BOFA, we often had to integrate our newer microservices (built with Vert.x and Spring Boot) with existing legacy banking systems or external financial data providers. These external systems frequently had APIs that were not directly compatible with the interfaces our services expected.
For example, we had a `MarketDataService` interface in our system, expecting methods like `getQuote(String symbol)` which returned a standardized `Quote` object. One crucial external data provider exposed its data via a SOAP web service with a very different request/response structure, let's call its client `LegacyMarketDataProvider`.
To bridge this, we implemented an `LegacyMarketDataProviderAdapter`:
```java
// Our system's target interface
public interface MarketDataService {
    Optional<Quote> getQuote(String symbol);
}

// Adaptee (simplified representation of the legacy client)
public class LegacyMarketDataProvider {
    public LegacyQuoteResponse fetchQuoteForInstrument(LegacyQuoteRequest request) {
        // ... complex SOAP call logic ...
    }
}

// The Adapter
public class LegacyMarketDataProviderAdapter implements MarketDataService {
    private final LegacyMarketDataProvider legacyProvider;
    private final LegacyRequestMapper requestMapper; // Maps our symbol to LegacyQuoteRequest
    private final QuoteResponseMapper responseMapper; // Maps LegacyQuoteResponse to our Quote

    public LegacyMarketDataProviderAdapter(LegacyMarketDataProvider legacyProvider,
                                           LegacyRequestMapper requestMapper,
                                           QuoteResponseMapper responseMapper) {
        this.legacyProvider = legacyProvider;
        this.requestMapper = requestMapper;
        this.responseMapper = responseMapper;
    }

    @Override
    public Optional<Quote> getQuote(String symbol) {
        LegacyQuoteRequest legacyRequest = requestMapper.mapToLegacyRequest(symbol);
        try {
            LegacyQuoteResponse legacyResponse = legacyProvider.fetchQuoteForInstrument(legacyRequest);
            if (legacyResponse != null && legacyResponse.isSuccessful()) {
                return Optional.of(responseMapper.mapToQuote(legacyResponse));
            }
        } catch (RemoteServiceException e) {
            // Log error, handle appropriately
            logger.error("Error fetching quote from legacy provider for symbol {}: {}", symbol, e.getMessage());
        }
        return Optional.empty();
    }
}
```
This Object Adapter held an instance of the `LegacyMarketDataProvider`. It translated our `getQuote` call into the `fetchQuoteForInstrument` call, including mapping request and response objects. This isolated our core application logic from the specifics of the legacy API. If we later needed to integrate with another data provider, we could write a new adapter for it without changing our service's core code, just the configuration to inject the correct adapter."

### Decorator (Wrapper)
*   **Intent:** Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.
*   **Structure:**
    *   `Component` (interface or abstract class): Defines the interface for objects that can have responsibilities added to them dynamically.
    *   `ConcreteComponent` (class): Defines an object to which additional responsibilities can be attached.
    *   `Decorator` (abstract class): Maintains a reference to a `Component` object and defines an interface that conforms to `Component`'s interface. It forwards requests to its `Component` object.
    *   `ConcreteDecorator` (class): Adds responsibilities to the component.
*   **Use Cases:**
    *   To add responsibilities to individual objects dynamically and transparently, i.e., without affecting other objects.
    *   For responsibilities that can be withdrawn.
    *   When extension by subclassing is impractical due to a large number of independent extensions or because a class definition is hidden or otherwise unavailable for subclassing.
    *   Example: Java I/O streams (`FileInputStream` decorated by `BufferedInputStream`, then `DataInputStream`). Adding scrollbars or borders to UI components.
*   **Pros:**
    *   More flexibility than static inheritance: Responsibilities can be added and removed at runtime.
    *   Avoids feature-laden classes high up in the hierarchy.
    *   Decorators can be composed.
*   **Cons:**
    *   Can result in a system with lots of small objects that look alike to a client.
    *   Can be complex to set up and manage the chain of decorators.
    *   Decorators and the decorated component aren't identical (decorator acts as a transparent enclosure, but `instanceof` checks might behave differently).

> **TODO:** In the TOPS Kafka pipeline, you might have needed to add responsibilities to message processors dynamically, like adding auditing, encryption/decryption, or compression. How could the Decorator pattern be applied here without altering the core processor's code?
**Sample Answer/Experience:**
"In the TOPS Kafka pipeline, after parsing a message, we had a core `MessageProcessor` interface with an `process(FinancialMessage message)` method. Sometimes, we needed to add cross-cutting concerns like auditing the message, encrypting certain fields before further processing, or compressing the output. The Decorator pattern would be ideal here.
```java
// Component interface
interface MessageProcessor {
    ProcessedResult process(FinancialMessage message);
}

// ConcreteComponent
class CoreMessageProcessor implements MessageProcessor {
    public ProcessedResult process(FinancialMessage message) {
        // Core business logic for processing the message
        System.out.println("Core processing for: " + message.getId());
        return new ProcessedResult(message.getId() + "_processed");
    }
}

// Decorator abstract class
abstract class MessageProcessorDecorator implements MessageProcessor {
    protected MessageProcessor wrappedProcessor;

    public MessageProcessorDecorator(MessageProcessor processor) {
        this.wrappedProcessor = processor;
    }

    public ProcessedResult process(FinancialMessage message) {
        return wrappedProcessor.process(message); // Delegate to wrapped component
    }
}

// ConcreteDecorators
class AuditingDecorator extends MessageProcessorDecorator {
    public AuditingDecorator(MessageProcessor processor) {
        super(processor);
    }

    @Override
    public ProcessedResult process(FinancialMessage message) {
        System.out.println("AUDIT: Processing message ID: " + message.getId() + " at " + System.currentTimeMillis());
        ProcessedResult result = super.process(message); // Call wrapped processor
        System.out.println("AUDIT: Finished processing message ID: " + message.getId() + " with result: " + result.getStatus());
        return result;
    }
}

class EncryptionDecorator extends MessageProcessorDecorator {
    public EncryptionDecorator(MessageProcessor processor) {
        super(processor);
    }

    @Override
    public ProcessedResult process(FinancialMessage message) {
        System.out.println("ENCRYPT: Encrypting sensitive fields for message ID: " + message.getId());
        // message.encryptSensitiveFields(); // Hypothetical method
        ProcessedResult result = super.process(message);
        System.out.println("ENCRYPT: Post-processing after encryption for message ID: " + message.getId());
        return result;
    }
}
```
Then, when setting up a processor for a specific Kafka topic or message type:
```java
MessageProcessor coreProcessor = new CoreMessageProcessor();
MessageProcessor auditedProcessor = new AuditingDecorator(coreProcessor);
MessageProcessor encryptedAndAuditedProcessor = new EncryptionDecorator(auditedProcessor);

// Use the final decorated processor
encryptedAndAuditedProcessor.process(new FinancialMessage("MSG001"));
```
This way, we could dynamically compose the processing chain. The `CoreMessageProcessor` remains unchanged, focusing only on its business logic. We could easily add or remove decorators like `AuditingDecorator` or `EncryptionDecorator` based on configuration or message properties without altering existing classes, adhering to the Open/Closed Principle. The main downside is the potential for many small decorator objects, but the flexibility gained often outweighs this."

### Facade
*   **Intent:** Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.
*   **Structure:**
    *   `Facade` (class): Knows which subsystem classes are responsible for a request. Delegates client requests to appropriate subsystem objects.
    *   `Subsystem classes`: Implement subsystem functionality. Handle work assigned by the `Facade` object. Have no knowledge of the facade; i.e., they keep no references to it.
*   **Use Cases:**
    *   To provide a simple interface to a complex subsystem.
    *   To decouple a subsystem from clients and other subsystems, thereby promoting subsystem independence and portability.
    *   To layer your subsystems. Use a facade to define an entry point to each subsystem level. If subsystems are dependent, you can simplify the dependencies between them by making them communicate with each other solely through their facades.
    *   Example: A `CustomerServiceFacade` might provide methods like `placeOrder()`, `checkOrderStatus()`, which internally interact with `InventorySystem`, `PaymentSystem`, `ShippingSystem`.
*   **Pros:**
    *   Simplifies the client's interaction with complex subsystems.
    *   Decouples the client from the subsystem's components, allowing the subsystem to evolve independently.
    *   Reduces compilation dependencies.
*   **Cons:**
    *   The facade can become a "god object" if it encapsulates too much or if it's the only way to access the subsystem (though this is not required).
    *   It doesn't prevent clients from accessing the underlying subsystem classes directly if they need more flexibility, which might break the encapsulation offered by the facade.

### Proxy
*   **Intent:** Provide a surrogate or placeholder for another object to control access to it.
*   **Structure:**
    *   `Subject` (interface): Defines the common interface for `RealSubject` and `Proxy` so that a `Proxy` can be used anywhere a `RealSubject` is expected.
    *   `RealSubject` (class): Defines the real object that the proxy represents.
    *   `Proxy` (class): Maintains a reference that lets the proxy access the real subject. Implements the `Subject` interface so it can be substituted for the `RealSubject`. Controls access to the `RealSubject` and may be responsible for creating and deleting it.
*   **Use Cases & Types:**
    *   **Remote Proxy:** Represents an object in a different address space (e.g., RMI stub objects). Hides network communication details.
    *   **Virtual Proxy:** Creates expensive objects on demand (lazy loading). The proxy holds information about the real object and loads it only when needed. (e.g., image placeholders that load the real image on click).
    *   **Protection Proxy:** Controls access to the original object. Useful when objects should have different access rights (e.g., based on user roles).
    *   **Smart Proxy (or Smart Reference):** Performs additional actions when an object is accessed. Examples: reference counting, loading a persistent object into memory when it's first referenced, locking the RealSubject to ensure exclusive access.
    *   **Logging Proxy:** Logs method calls and parameters before delegating to the real object.
    *   Caching Proxy: Caches results of expensive operations from the RealSubject.
*   **Pros:**
    *   Can introduce a level of indirection when accessing an object.
    *   Can provide various optimizations (lazy loading, caching).
    *   Can enhance security (protection proxy).
    *   Can hide complexity (remote proxy).
*   **Cons:**
    *   Can increase complexity due to the introduction of an additional class.
    *   The response from the proxy might be delayed (e.g., if it needs to load the real object or communicate over a network).

> **TODO:** Spring AOP often uses proxies (dynamic proxies) to implement cross-cutting concerns like transaction management or security. Can you describe how this works conceptually and how it impacted your development in Spring Boot projects at Intralinks or TOPS?
**Sample Answer/Experience:**
"Yes, Spring AOP's use of dynamic proxies is fundamental to how it provides cross-cutting concerns. In our Spring Boot projects at Intralinks, this was most evident with `@Transactional` and `@PreAuthorize` (for security).
Conceptually, when a bean is eligible for AOP (e.g., it has methods annotated with `@Transactional`), Spring doesn't inject the raw bean instance directly. Instead, it creates a dynamic proxy (either a JDK dynamic proxy if the bean implements an interface, or a CGLIB proxy if it's a class-based proxy) that wraps the actual bean.
This proxy intercepts method calls:
1.  **`@Transactional`:** When a method annotated with `@Transactional` is called on the proxy, the proxy's AOP advice initiates a transaction (e.g., `beginTransaction()`) before delegating the call to the actual method on the target bean. After the target method executes, the proxy's advice handles committing or rolling back the transaction.
2.  **`@PreAuthorize`:** Similarly, before executing a method annotated with `@PreAuthorize`, the proxy's security advice would evaluate the SpEL expression in the annotation. If authorized, it calls the target method; otherwise, it throws an `AccessDeniedException`.

**Impact on Development:**
*   **Declarative Services:** The biggest impact was making services declarative. We could focus on business logic in our service beans and simply annotate methods for transactional behavior or security rules. This massively reduced boilerplate code.
*   **Non-Invasive:** Our core business logic classes didn't need to be aware of transaction management or security APIs directly.
*   **Self-Invocation Issue:** One key thing to remember (and a common pitfall) is that calls to proxied methods *within the same class* (self-invocation) bypass the proxy. For example, if a public non-transactional method in a service calls a public `@Transactional` method in the *same* service instance (`this.transactionalMethod()`), the transactional advice on the proxy won't be invoked for the second call. We had to be mindful of this and structure service calls accordingly, sometimes by refactoring the internal call into a separate Spring bean/proxy.
*   **Debugging:** Sometimes debugging can be a bit more complex as you step through proxy classes in the call stack, but IDEs are generally good at showing the underlying target method.
Overall, Spring's AOP proxy mechanism was a huge benefit, allowing us to keep our services clean and focus on business logic while declaratively applying common enterprise concerns."

### Composite
*   **Intent:** Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.
*   **Structure:**
    *   `Component` (interface or abstract class): Declares the interface for objects in the composition. Implements default behavior for the interface common to all classes, as appropriate. Declares an interface for accessing and managing its child components (e.g., `add()`, `remove()`, `getChild()`).
    *   `Leaf` (class): Represents leaf objects in the composition. A leaf has no children. Defines behavior for primitive objects in the composition.
    *   `Composite` (class): Defines behavior for components having children. Stores child components. Implements child-related operations in the `Component` interface.
    *   `Client`: Manipulates objects in the composition through the `Component` interface.
*   **Use Cases:**
    *   When you want to represent part-whole hierarchies of objects.
    *   When you want clients to be able to treat individual objects and compositions of objects uniformly.
    *   Example: UI layout elements (panels containing buttons, labels, other panels). Representing organizational hierarchies (employees, managers, departments). Graphics applications (grouping shapes).
*   **Pros:**
    *   Simplifies client code: Clients can treat composite structures and individual objects uniformly.
    *   Makes it easier to add new kinds of components. New `Leaf` or `Composite` classes can be introduced easily.
*   **Cons:**
    *   Can make the design overly general. Sometimes you might want to restrict a composite to contain only certain types of components. This is harder to enforce with the basic Composite pattern.
    *   The `Component` interface might become bloated if it needs to support many operations for managing children (some of which may not make sense for `Leaf` objects). Often, `Leaf` nodes implement child management operations as no-ops or by throwing exceptions.

### Flyweight
*   **Intent:** Use sharing to support large numbers of fine-grained objects efficiently.
*   **Structure:**
    *   `Flyweight` (interface or abstract class): Declares an interface through which flyweights can receive and act on extrinsic state.
    *   `ConcreteFlyweight` (class): Implements the `Flyweight` interface and adds storage for intrinsic state, if any. A `ConcreteFlyweight` object must be sharable.
    *   `UnsharedConcreteFlyweight` (class, optional): Not all `Flyweight` subclasses need to be shared.
    *   `FlyweightFactory` (class): Creates and manages flyweight objects. Ensures that flyweights are shared properly. When a client requests a flyweight, the factory supplies an existing instance or creates one, if it doesn't exist yet.
    *   `Client`: Maintains references to flyweights. Computes or stores the extrinsic state of flyweights.
*   **Intrinsic vs. Extrinsic State:**
    *   **Intrinsic State:** Information that is independent of the flyweight's context, making it sharable (e.g., the character code in a character object). Stored in the `ConcreteFlyweight`.
    *   **Extrinsic State:** Information that depends on the flyweight's context and therefore cannot be shared (e.g., the position of a character in a document). Passed to flyweight methods by the client.
*   **Use Cases:**
    *   When an application uses a large number of objects.
    *   When storage costs are high because of the sheer quantity of objects.
    *   When most object state can be made extrinsic.
    *   When many groups of objects may be replaced by relatively few shared objects once extrinsic state is removed.
    *   When the application doesn't depend on object identity.
    *   Example: Character rendering in a text editor (sharing character objects, position is extrinsic). Representing individual cells in a large spreadsheet. `java.lang.String` intern pool, `Integer.valueOf()` (for cached range).
*   **Pros:**
    *   Significant reduction in the number of objects, leading to memory savings.
    *   Can reduce CPU usage if intrinsic state computation is expensive.
*   **Cons:**
    *   Increased complexity due to the separation of intrinsic and extrinsic state.
    *   Runtime costs associated with transferring, finding, and/or computing extrinsic state.
    *   May introduce a slight performance overhead when accessing extrinsic state.

## Behavioral Patterns

### Strategy
*   **Intent:** Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.
*   **Structure:**
    *   `Strategy` (interface or abstract class): Declares a common interface for all supported algorithms. `Context` uses this interface to call the algorithm defined by a `ConcreteStrategy`.
    *   `ConcreteStrategy` (class): Implements the algorithm using the `Strategy` interface.
    *   `Context` (class): Is configured with a `ConcreteStrategy` object. Maintains a reference to a `Strategy` object. May define an interface that lets `Strategy` access its data.
*   **Use Cases:**
    *   When you want to use different variants of an algorithm within an object and be ableable to switch from one algorithm to another during runtime.
    *   When you have a lot of similar classes that only differ in the way they execute some behavior.
    *   To isolate the business logic of a class from the implementation details of algorithms that are not important in the context of that logic.
    *   When a class defines many behaviors, and these appear as multiple conditional statements in its operations. Instead of many conditionals, move related conditional branches into their own Strategy class.
    *   Example: Payment processing (credit card, PayPal, bank transfer strategies). Sorting algorithms (`Collections.sort()` takes a `Comparator`). Validation strategies. Compression algorithms.
*   **Pros:**
    *   Provides a way to configure a class with one of many behaviors.
    *   A class can defer the choice of algorithm to its clients.
    *   Strategies can be switched at runtime.
    *   Simplifies the `Context` object by extracting algorithmic variations.
    *   Replaces conditional statements for selecting behavior.
    *   Follows Open/Closed Principle: New strategies can be added without modifying the context.
*   **Cons:**
    *   Clients must be aware of the different Strategies to select the appropriate one.
    *   Increased number of objects in the application.
    *   Communication overhead between Strategy and Context if a Strategy needs access to a lot of data from the Context.

> **TODO:** In the TOPS project, different financial instruments or trade types might require different validation rule sets or pricing algorithms. How would you use the Strategy pattern to manage these varying algorithms within your Spring Boot services?
**Sample Answer/Experience:**
"For the TOPS project, the Strategy pattern was very useful for handling variations in business logic like validation or pricing for different financial instruments. For instance, equity trades, bond trades, and FX trades might have distinct validation rules.
We could define a `ValidationStrategy` interface:
```java
// Strategy interface
interface ValidationStrategy {
    List<ValidationError> validate(TradeData tradeData);
}

// ConcreteStrategies
class EquityValidationStrategy implements ValidationStrategy {
    @Override
    public List<ValidationError> validate(TradeData tradeData) {
        // Equity-specific validation logic
        List<ValidationError> errors = new ArrayList<>();
        if (tradeData.getExchange() == null) errors.add(new ValidationError("Exchange is mandatory for equities."));
        // ... more rules ...
        return errors;
    }
}

class BondValidationStrategy implements ValidationStrategy {
    @Override
    public List<ValidationError> validate(TradeData tradeData) {
        // Bond-specific validation logic
        List<ValidationError> errors = new ArrayList<>();
        if (tradeData.getMaturityDate() == null) errors.add(new ValidationError("Maturity date is mandatory for bonds."));
        // ... more rules ...
        return errors;
    }
}

// Context
@Service
class TradeValidatorService {
    private Map<InstrumentType, ValidationStrategy> strategies = new EnumMap<>(InstrumentType.class);

    // Strategies could be injected by Spring, e.g., by finding all beans implementing ValidationStrategy
    @Autowired
    public TradeValidatorService(List<ValidationStrategy> strategyList) {
        // A bit more logic needed here to map them correctly, e.g. based on an annotation on the strategy
        // For simplicity:
        strategies.put(InstrumentType.EQUITY, new EquityValidationStrategy()); // In reality, Spring would inject these
        strategies.put(InstrumentType.BOND, new BondValidationStrategy());
    }

    public List<ValidationError> validateTrade(TradeData tradeData) {
        ValidationStrategy strategy = strategies.get(tradeData.getInstrumentType());
        if (strategy == null) {
            return Arrays.asList(new ValidationError("No validation strategy found for instrument type: " + tradeData.getInstrumentType()));
        }
        return strategy.validate(tradeData);
    }
}
```
In our Spring Boot application, the `TradeValidatorService` (Context) would be injected with a map of all available `ValidationStrategy` implementations, perhaps keyed by an `InstrumentType` enum. When a trade needs validation, the service would look up the appropriate strategy for the trade's instrument type and delegate the validation call to it.
This allowed us to:
1.  Add new validation strategies for new instrument types without modifying `TradeValidatorService` (Open/Closed Principle).
2.  Keep validation logic for each instrument type cohesive and separate.
3.  Easily test each validation strategy in isolation.
A similar approach could be used for pricing algorithms, fee calculations, etc., making the system flexible and maintainable."

### Observer
*   **Intent:** Define a one-to-many dependency between objects so that when one object (the subject) changes state, all its dependents (observers) are notified and updated automatically.
*   **Structure:**
    *   `Subject` (interface or abstract class): Knows its observers. Provides an interface for attaching (`registerObserver`) and detaching (`removeObserver`) `Observer` objects.
    *   `ConcreteSubject` (class): Stores state of interest to `ConcreteObserver` objects. Sends a notification to its observers when its state changes (by calling `notifyObservers()`).
    *   `Observer` (interface or abstract class): Defines an updating interface (`update()`) for objects that should be notified of changes in a subject.
    *   `ConcreteObserver` (class): Maintains a reference to a `ConcreteSubject` object (optional, for pull model). Stores state that should stay consistent with the subject's. Implements the `Observer` updating interface to keep its state consistent with the subject's.
*   **Push vs. Pull Model:**
    *   **Push Model:** The subject sends observers detailed information about the change, whether they want it or not.
    *   **Pull Model:** The subject sends only a minimal notification, and observers query the subject for details. More flexible but less efficient if observers always need specific data.
*   **Use Cases:**
    *   When an abstraction has two aspects, one dependent on the other. Encapsulating these aspects in separate objects lets you vary and reuse them independently.
    *   When a change to one object requires changing others, and you don't know how many objects need to be changed.
    *   When an object should be able to notify other objects without making assumptions about who these objects are. In other words, you don't want these objects tightly coupled.
    *   Example: Event handling systems (e.g., Swing/AWT listeners). Model-View-Controller (MVC) architectural pattern (View observes Model). Implementing distributed event-handling systems. RSS feeds.
    *   `java.util.Observer` and `java.util.Observable` (deprecated in Java 9 due to issues like not being serializable and Observable being a class, not an interface). Modern implementations often use `PropertyChangeListener` or custom listener interfaces, or event bus systems like Vert.x event bus or Spring Application Events.
*   **Pros:**
    *   Abstract coupling between Subject and Observer. Subject knows only that it has a list of Observers, not their concrete classes.
    *   Support for broadcast communication.
    *   Observers can be added/removed at runtime.
*   **Cons:**
    *   Unexpected updates: Observers are notified in an unpredictable order.
    *   Can lead to performance issues if there are many observers or if the update logic is complex (potential for cascading updates).
    *   Memory leaks if observers are not properly removed (the "lapsed listener" problem).

> **TODO:** Vert.x, used in the Intralinks project, heavily uses an event-driven model. How does this relate to the Observer pattern? Discuss how you handled events and ensured loose coupling between Vert.x verticles.
**Sample Answer/Experience:**
"Vert.x's event bus is a prime example of a system that embodies the Observer pattern, though it's more of a distributed, message-passing variation.
In Vert.x:
*   **Subject (or Publisher):** A verticle that publishes a message to a specific address on the event bus is acting like a subject. It doesn't know who is listening.
*   **Observer (or Subscriber/Consumer):** Verticles that register a `handler` for a specific event bus address are acting as observers. When a message is published to that address, their handler's `handle(Message<T> event)` method is invoked.
*   **Event/Notification:** The `Message<T>` object itself is the notification, carrying the data.

We used this extensively in Intralinks for inter-verticle communication. For example, an HTTP API verticle, upon receiving a file upload request, might publish a message to an address like `file.upload.pending` with details about the file. A separate `FileProcessingVerticle` would have registered a consumer for this address.
```java
// In HttpApiVerticle
eventBus.publish("file.upload.pending", new JsonObject().put("filePath", "/tmp/uploaded.dat"));

// In FileProcessingVerticle
eventBus.consumer("file.upload.pending", message -> {
    JsonObject fileInfo = (JsonObject) message.body();
    String filePath = fileInfo.getString("filePath");
    System.out.println("OBSERVER: Received file for processing: " + filePath);
    // ... process the file ...
    message.reply(new JsonObject().put("status", "processed")); // Optional reply
});
```
**Loose Coupling:** This provided excellent loose coupling. The `HttpApiVerticle` didn't need to know which verticle(s) would handle file processing, or even if there were any. It just published an event. We could add more `FileProcessingVerticle` instances (observers) for scalability, or even add new types of verticles (e.g., an `AuditVerticle`) listening to the same event, without changing the publisher. This is a key benefit of the Observer pattern.
The Vert.x event bus also supports point-to-point messaging (`send` vs `publish`) which is also observer-like but ensures only one consumer handles the message.
Compared to the classic Java Observer pattern (like `java.util.Observer`), the Vert.x event bus is more powerful for microservice architectures as it works across different verticles (which can be thought of as lightweight components, potentially in different threads or even distributed). It manages thread safety and message delivery transparently for the most part."

### Template Method
*   **Intent:** Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.
*   **Structure:**
    *   `AbstractClass` (abstract class): Defines abstract primitive operations that concrete subclasses define to implement steps of an algorithm. Implements a template method defining the skeleton of an algorithm. The template method calls primitive operations as well as operations defined in `AbstractClass` or those of other objects.
    *   `ConcreteClass` (class): Implements the primitive operations to carry out subclass-specific steps of the algorithm.
*   **Hooks:** Primitive operations that provide default behavior are called "hooks." Subclasses can override them if needed, but don't have to.
*   **Use Cases:**
    *   To implement the invariant parts of an algorithm once and leave it up to subclasses to implement the behavior that can vary.
    *   When common behavior among subclasses should be factored and localized in a common class to avoid code duplication. This is an example of "inversion of control" (Hollywood Principle: "Don't call us, we'll call you").
    *   To control subclasses' extensions. The template method might call hook operations at specific points, thereby permitting extensions only at those points.
    *   Example: Framework development. Common algorithm for data processing where steps like data reading, data parsing, data validation can be customized by subclasses. `HttpServlet`'s `doGet`, `doPost` methods call internal methods like `service` which can be seen as a template.
*   **Pros:**
    *   Code reuse for the invariant part of the algorithm.
    *   Enforces an overall structure for an algorithm.
    *   Allows subclasses to customize specific parts of the algorithm.
    *   "Inversion of Control" - the parent class calls operations of the subclass, not the other way around.
*   **Cons:**
    *   Can be difficult to change the fundamental steps of the algorithm (as they are in the abstract class).
    *   Can lead to a proliferation of subclasses if there are many variations.
    *   Subclasses are tightly coupled to the `AbstractClass` regarding the algorithm's structure.

### Chain of Responsibility
*   **Intent:** Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.
*   **Structure:**
    *   `Handler` (interface or abstract class): Defines an interface for handling requests. Optionally implements the successor link.
    *   `ConcreteHandler` (class): Handles requests it is responsible for. Can access its successor. If the `ConcreteHandler` can handle the request, it does so; otherwise, it forwards the request to its successor.
    *   `Client`: Initiates the request to a `ConcreteHandler` object on the chain.
*   **Use Cases:**
    *   When more than one object may handle a request, and the handler isn't known a priori. The handler should be ascertained automatically.
    *   When you want to issue a request to one of several objects without specifying the receiver explicitly.
    *   When the set of objects that can handle a request should be specified dynamically.
    *   Example: Servlet filters in Java EE (each filter processes the request and can pass it to the next filter in the chain). UI event propagation (e.g., an event bubbling up a component hierarchy). Logging frameworks (different log handlers for different levels or targets). Approval workflows.
*   **Pros:**
    *   Reduced coupling: Decouples the sender from the receiver. The sender doesn't need to know which object in the chain will handle the request.
    *   Flexibility in assigning responsibilities to objects: You can add or remove handlers from the chain dynamically.
    *   Flexibility in how requests are handled: Different handlers can implement different logic.
*   **Cons:**
    *   Receipt isn't guaranteed: A request can reach the end of the chain with no handler processing it (unless the chain is configured to always have a default handler).
    *   Can be difficult to debug if the chain is long or complex.
    *   May have performance implications due to traversal of the chain.

> **TODO:** Servlet Filters in Java web applications (like those built with Spring MVC in your projects) are a classic example of Chain of Responsibility. Describe how you might have used a custom filter chain for handling concerns like authentication, request logging, and header manipulation in a Spring Boot application at Intralinks.
**Sample Answer/Experience:**
"In our Spring Boot applications at Intralinks, we definitely used Servlet Filters, which embody the Chain of Responsibility pattern, to handle various cross-cutting concerns before a request even reached the DispatcherServlet and our controllers.
For instance, a typical chain might look like:
1.  **RequestLoggingFilter:** The first filter in the chain. Its job is to log incoming request details (URI, method, headers, even payload if configured carefully for non-sensitive data). After logging, it calls `chain.doFilter(request, response)`.
2.  **AuthenticationFilter:** Next, a custom JWT-based authentication filter. It would inspect the `Authorization` header, validate the token, and if valid, populate the Spring SecurityContext. If invalid, it might short-circuit the chain and return a 401/403 response. If valid, it calls `chain.doFilter()`.
3.  **HeaderManipulationFilter:** This filter might add standard security headers to the response (e.g., X-Content-Type-Options, X-XSS-Protection) or add correlation IDs. It would typically call `chain.doFilter()` first, and then modify the response headers *after* the rest of the chain (including the controller) has processed the request.
4.  **Standard Spring Filters:** Then other filters like Spring Security's own filters, Spring MVC's character encoding filter, etc., would execute.

Each filter is a `ConcreteHandler`. It decides if it can handle/process the request for its specific concern and then either passes the request to the next filter in the chain using `FilterChain.doFilter()` or, in some cases (like auth failure), terminates the chain.
To implement this, we'd create classes implementing `javax.servlet.Filter` and register them as `@Component` beans or using a `FilterRegistrationBean` for more control over ordering (`@Order` annotation or `FilterRegistrationBean.setOrder()`).
```java
@Component
@Order(1) // Example order
public class RequestLoggingFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        // Log request details
        System.out.println("LOGGING FILTER: Incoming request: " + httpRequest.getRequestURI());
        chain.doFilter(request, response); // Pass to next in chain
        // Can log response details here too
    }
}

@Component
@Order(2)
public class CustomAuthenticationFilter implements Filter {
    // ...
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        // Attempt authentication
        if (isAuthenticated(request)) {
            chain.doFilter(request, response);
        } else {
            ((HttpServletResponse) response).sendError(HttpServletResponse.SC_UNAUTHORIZED, "Authentication Failed");
            // Don't call chain.doFilter()
        }
    }
    private boolean isAuthenticated(ServletRequest request) { /* ... */ return true;}
}
```
This pattern allowed us to keep each concern modular and independent. Adding a new cross-cutting concern just meant adding a new filter to the chain, without modifying existing filters or controller logic."

### Command
*   **Intent:** Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.
*   **Structure:**
    *   `Command` (interface or abstract class): Declares an interface for executing an operation (`execute()`).
    *   `ConcreteCommand` (class): Implements the `Command` interface. Defines a binding between a `Receiver` object and an action. Implements `execute()` by invoking the corresponding operation(s) on `Receiver`.
    *   `Client` (or `Invoker` setup): Creates a `ConcreteCommand` object and sets its `Receiver`.
    *   `Invoker` (class): Asks the `Command` to carry out the request (by calling `command.execute()`).
    *   `Receiver` (class): Knows how to perform the operations associated with carrying out a request. Any class may serve as a `Receiver`.
*   **Use Cases:**
    *   Parameterizing objects by an action to perform (e.g., UI buttons that execute different commands).
    *   To specify, queue, and execute requests at different times. A command object can have a lifetime independent of the original request.
    *   To support undo/redo. The `Command` can store state for reversing its execution.
    *   To support logging changes (transactions). The `execute` method can store the command for later logging.
    *   To structure a system around high-level operations built on primitive operations.
    *   Example: Menu items, toolbar buttons in a GUI. Task queuing (e.g., thread pools executing `Runnable` commands). Implementing "macro" recording.
*   **Pros:**
    *   Decouples the object that invokes the operation from the one that knows how to perform it.
    *   Commands are first-class objects. They can be manipulated and extended like any other object.
    *   You can assemble commands into a composite command.
    *   Easy to add new commands because you don't have to change existing classes.
    *   Supports undo/redo and transaction-like semantics.
*   **Cons:**
    *   Can result in a proliferation of command classes if many actions are needed.
    *   Each command needs to store all parameters required for its execution, which might be complex.

### State
*   **Intent:** Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.
*   **Structure:**
    *   `Context` (class): Defines the interface of interest to clients. Maintains an instance of a `ConcreteState` subclass that defines the current state. Delegates state-specific behavior to the current `ConcreteState` object.
    *   `State` (interface or abstract class): Defines an interface for encapsulating the behavior associated with a particular state of the `Context`.
    *   `ConcreteState` (class): Implements a behavior associated with a state of the `Context`. May also handle transitions to other states.
*   **Use Cases:**
    *   An object's behavior depends on its state, and it must change its behavior at run-time depending on that state.
    *   Operations have large, multipart conditional statements that depend on the object's state. This state is usually represented by one or more enumerated constants. Often, several operations will contain this same conditional structure. The State pattern puts each branch of the conditional in a separate class. This lets you treat the object's state as an object in its own right that can vary independently from other objects.
    *   Example: TCP connection states (Listening, Established, Closed). Vending machine states. Document lifecycle states (Draft, Review, Approved, Published).
*   **Pros:**
    *   Localizes state-specific behavior and partitions behavior for different states.
    *   Makes state transitions explicit.
    *   State objects can be shared if they have no instance variables (Flyweight).
    *   Avoids large conditional statements based on state.
*   **Cons:**
    *   Can result in a large number of `State` classes if an object has many states.
    *   `Context` object can become coupled to its `ConcreteState` objects if it's responsible for state transitions.
    *   Can be overkill for simple state changes.

> **TODO:** Consider a document processing workflow in an AEM application at Herc Rentals (e.g., 'Draft' -> 'In Review' -> 'Approved' -> 'Published'). How would the State pattern help manage the behavior and transitions of a document object through these states?
**Sample Answer/Experience:**
"For a document workflow in AEM at Herc Rentals, the State pattern would be very effective in managing how a document behaves and what actions are permissible in each state.
The `DocumentContext` would be our AEM content resource or a Sling Model adapting it. It would hold a reference to the current `DocumentState`.
```java
// State interface
interface DocumentState {
    void submitForReview(DocumentContext document);
    void approve(DocumentContext document);
    void publish(DocumentContext document);
    void reject(DocumentContext document);
    // Method to get available actions for UI
    List<String> getAllowedActions();
}

// ConcreteState classes
class DraftState implements DocumentState {
    public void submitForReview(DocumentContext doc) {
        System.out.println("Submitting document " + doc.getId() + " for review.");
        // Logic for submission, e.g., assign to reviewer group
        doc.setCurrentState(new InReviewState()); // Transition state
    }
    public void approve(DocumentContext doc) { throw new IllegalStateException("Cannot approve a document in Draft state."); }
    public void publish(DocumentContext doc) { throw new IllegalStateException("Cannot publish a document in Draft state."); }
    public void reject(DocumentContext doc) { /* Can perhaps delete or archive */ }
    public List<String> getAllowedActions() { return Arrays.asList("submitForReview", "delete"); }
}

class InReviewState implements DocumentState {
    public void submitForReview(DocumentContext doc) { throw new IllegalStateException("Document already in review."); }
    public void approve(DocumentContext doc) {
        System.out.println("Approving document " + doc.getId());
        doc.setCurrentState(new ApprovedState());
    }
    public void publish(DocumentContext doc) { throw new IllegalStateException("Must be approved before publishing."); }
    public void reject(DocumentContext doc) {
        System.out.println("Rejecting document " + doc.getId());
        doc.setCurrentState(new DraftState()); // Back to draft
    }
    public List<String> getAllowedActions() { return Arrays.asList("approve", "reject"); }
}

class ApprovedState implements DocumentState {
    // ... similar implementations for publish, etc. ...
    public void submitForReview(DocumentContext doc) { /* Maybe allow resubmission */ }
    public void approve(DocumentContext doc) {  throw new IllegalStateException("Document already approved."); }
    public void publish(DocumentContext doc) {
        System.out.println("Publishing document " + doc.getId());
        // Actual publish logic (e.g., replicate to publish instances)
        doc.setCurrentState(new PublishedState());
    }
    public void reject(DocumentContext doc) { /* Not typical to reject an approved doc, maybe 'unapprove' */ }
    public List<String> getAllowedActions() { return Arrays.asList("publish", "unapprove"); }
}

class PublishedState implements DocumentState { /* Mostly terminal or allows unpublishing */
    public void submitForReview(DocumentContext doc) { throw new IllegalStateException("Cannot submit a published document for review."); }
    public void approve(DocumentContext doc) { throw new IllegalStateException("Document already published."); }
    public void publish(DocumentContext doc) { throw new IllegalStateException("Document already published."); }
    public void reject(DocumentContext doc) { throw new IllegalStateException("Cannot reject a published document."); }
    public List<String> getAllowedActions() { return Arrays.asList("unpublish", "archive"); }
}

// Context class (could be part of a Sling Model)
class DocumentContext {
    private String id;
    private DocumentState currentState; // Current state object

    public DocumentContext(String id) {
        this.id = id;
        this.currentState = new DraftState(); // Initial state
        // Persist initial state to JCR property
    }

    public void setCurrentState(DocumentState state) {
        this.currentState = state;
        // Persist new state string (e.g., state.getClass().getSimpleName()) to JCR property
        System.out.println("Document " + id + " transitioned to " + state.getClass().getSimpleName());
    }
    public String getId() { return id; }

    // Delegate actions to the current state object
    public void submit() { currentState.submitForReview(this); }
    public void approve() { currentState.approve(this); }
    public void publish() { currentState.publish(this); }
    public void reject() { currentState.reject(this); }
    public List<String> getAvailableActions() { return currentState.getAllowedActions(); }
}
```
When a user interacts with the document in AEM (e.g., clicks a workflow button), the action would be delegated to the `DocumentContext` object, which in turn calls the corresponding method on its `currentState` object. The state object handles the logic and any transitions. This approach cleanly separates the logic for each state, makes it easy to add new states or modify transitions, and avoids complex conditional logic in the `DocumentContext` itself. The `getAllowedActions()` method is useful for dynamically rendering UI buttons based on the current state."

### Iterator
*   **Intent:** Provide a way to access the elements of an aggregate object (collection) sequentially without exposing its underlying representation.
*   **Structure:**
    *   `Iterator` (interface): Defines an interface for accessing and traversing elements (e.g., `hasNext()`, `next()`, `remove()`).
    *   `ConcreteIterator` (class): Implements the `Iterator` interface. Keeps track of the current position in the traversal of the aggregate.
    *   `Aggregate` (interface): Defines an interface for creating an `Iterator` object (e.g., `createIterator()`).
    *   `ConcreteAggregate` (class): Implements the `Aggregate` interface and returns an instance of the `ConcreteIterator`.
*   **Use Cases:**
    *   To access an aggregate object's contents without exposing its internal structure.
    *   To support multiple traversals of aggregate objects.
    *   To provide a uniform interface for traversing different aggregate structures (e.g., lists, trees, sets).
    *   Example: Java Collections Framework (`java.util.Iterator`, `java.util.Iterable`). `Scanner` can iterate over input.
*   **Pros:**
    *   Supports variations in the traversal of an aggregate.
    *   Iterators simplify the `Aggregate` interface.
    *   More than one traversal can be pending on an aggregate.
    *   Decouples algorithms from the specific collection implementation.
*   **Cons:**
    *   If the aggregate is modified while an iterator is active (and the iterator is not designed for concurrent modification, like Java's fail-fast iterators), it can lead to unpredictable behavior or exceptions (`ConcurrentModificationException`).
    *   For simple collections, creating an external iterator might be overkill.

### Mediator
*   **Intent:** Define an object that encapsulates how a set of objects interact. Mediator promotes loose coupling by keeping objects from referring to each other explicitly, and it lets you vary their interaction independently.
*   **Structure:**
    *   `Mediator` (interface or abstract class): Defines an interface for communicating with `Colleague` objects.
    *   `ConcreteMediator` (class): Implements cooperative behavior by coordinating `Colleague` objects. Knows and maintains its colleagues.
    *   `Colleague` (interface or abstract class): Defines an interface for objects that communicate through a `Mediator`.
    *   `ConcreteColleague` (class): Communicates with its `Mediator` whenever it would have otherwise communicated with another colleague.
*   **Use Cases:**
    *   When a set of objects communicate in well-defined but complex ways. The resulting interdependencies are unstructured and difficult to understand.
    *   Reusing an object is difficult because it refers to and communicates with many other objects.
    *   A behavior that's distributed between several classes should be customizable without a lot of subclassing.
    *   Example: GUI components in a dialog box (e.g., a button click might affect a list box and a text field; the dialog itself can be the mediator). Air traffic control system. Chat room application.
*   **Pros:**
    *   Limits subclassing. A mediator localizes behavior that would otherwise be distributed among several objects.
    *   Decouples colleagues. Colleagues only know the mediator, not each other.
    *   Simplifies object protocols. Replaces many-to-many interactions with one-to-many interactions between the mediator and its colleagues.
    *   Centralizes control. The mediator encapsulates the interaction logic.
*   **Cons:**
    *   The `Mediator` can become a "god object" or a monolith, complex and hard to maintain, if it takes on too many responsibilities.
    *   Can make it harder to understand how objects interact if the mediator logic is very complex.
    *   Can reduce performance slightly due to the indirection.
