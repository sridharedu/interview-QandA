# Java 8, 9, 10 & 11 Key Features Theory

This document covers key features introduced in Java 8 and significant updates from Java 9, 10, and 11, relevant for senior Java engineering interviews.

## Java 8 Features

Java 8, released in March 2014, was a massive release bringing functional programming capabilities to Java.

### Lambda Expressions
*   **Syntax:** Lambda expressions provide a concise way to represent anonymous functions. The basic syntax is `(parameters) -> expression` or `(parameters) -> { statements; }`.
    *   `() -> System.out.println("Hello")` (no parameters)
    *   `x -> x * x` (single parameter, type inferred)
    *   `(int x, int y) -> x + y` (multiple parameters with types)
    *   `(String s) -> { System.out.println(s); return s.length(); }` (block lambda)
*   **Functional Interfaces:** Lambdas are used to implement functional interfaces (interfaces with a single abstract method - SAM). The lambda's parameter types and return type must match the abstract method's signature.
*   **Target Typing:** The specific type of a lambda expression is inferred by the compiler from the context in which it is used (e.g., assignment to a variable of a functional interface type, or a method parameter of a functional interface type).
*   **`this` Keyword Behavior:**
    *   Inside a lambda expression, `this` refers to the `this` of the enclosing class (lexical scoping). This is different from anonymous inner classes, where `this` refers to the anonymous class instance itself.
    *   Lambdas do not introduce a new scope for `this`.
*   **Variable Capture:** Lambdas can capture (close over) variables from their enclosing scope. They can capture `final` or "effectively final" local variables. Effectively final means the variable's value is never changed after initialization. They can also capture instance and static variables.

> **TODO:** Describe how you utilized lambda expressions in your Vert.x applications at Intralinks for handling asynchronous events or in Spring Boot for defining concise bean configurations or task definitions.
**Sample Answer/Experience:**
"In my Vert.x projects at Intralinks, lambda expressions were fundamental for handling asynchronous operations and events concisely. For instance, when making an HTTP client request or setting up an event bus consumer, Vert.x uses handlers that are functional interfaces. Lambdas made this very clean:
```java
// Vert.x HTTP Client example
webClient.get("/api/data")
  .as(BodyCodec.jsonObject())
  .send(ar -> { // ar is AsyncResult<HttpResponse<JsonObject>>
    if (ar.succeeded()) {
      HttpResponse<JsonObject> response = ar.result();
      JsonObject body = response.body();
      System.out.println("Received response: " + body.encodePrettily());
      // process response
    } else {
      System.err.println("Something went wrong: " + ar.cause().getMessage());
    }
  });

// Vert.x Event Bus consumer
eventBus.consumer("some.address", message -> { // message is Message<JsonObject>
  System.out.println("Received message: " + message.body().getString("key"));
  // process message
});
```
This is far more readable than using anonymous inner classes. Similarly, in Spring Boot, for simple `Runnable` tasks submitted to an `ExecutorService`, or for defining `Comparator`s, or even in configurations like Spring Security with `HttpSecurity` lambdas, they significantly reduced verbosity."

### Functional Interfaces
*   **Definition:** An interface with exactly one abstract method (SAM). It can have multiple default or static methods.
*   **`@FunctionalInterface` Annotation:** This annotation is optional but recommended. It causes the compiler to generate an error if the annotated interface does not satisfy the requirements of a functional interface.
*   **Common Examples (in `java.util.function` package):**
    *   **`Predicate<T>`:** Represents a predicate (boolean-valued function) of one argument. Method: `boolean test(T t)`.
    *   **`Function<T, R>`:** Represents a function that accepts one argument and produces a result. Method: `R apply(T t)`.
    *   **`Consumer<T>`:** Represents an operation that accepts a single input argument and returns no result. Method: `void accept(T t)`.
    *   **`Supplier<T>`:** Represents a supplier of results. Method: `T get()`.
    *   **`Runnable`:** (from `java.lang`) Already a functional interface. Method: `void run()`.
    *   **`BiFunction<T, U, R>`:** Represents a function that accepts two arguments and produces a result.
    *   **`UnaryOperator<T>`:** A specialization of `Function` for the case where the operand and result are of the same type.
    *   **`BinaryOperator<T>`:** A specialization of `BiFunction` for the case where the operands and result are all of the same type.

### Stream API
*   **Overview:** A sequence of elements supporting sequential and parallel aggregate operations. Streams are not data structures; they carry values from a source (like collections, arrays, I/O channels) through a pipeline of computational steps.
*   **Stream Sources:**
    *   `Collection.stream()` or `Collection.parallelStream()`
    *   `Arrays.stream(array)`
    *   `Stream.of(val1, val2, ...)`
    *   `Stream.iterate(seed, unaryOperator)`
    *   `Stream.generate(supplier)`
    *   `Files.lines(path)`
*   **Stream Characteristics:**
    *   **Pipelining:** Many stream operations return a stream themselves, allowing operations to be chained.
    *   **Internal Iteration:** Iteration is handled internally by the stream, unlike external iteration with `for` loops.
    *   **Lazy Evaluation:** Intermediate operations are not executed until a terminal operation is invoked. This allows for optimizations.
    *   **Possibly Unbounded:** While most streams from collections are finite, streams can be infinite (e.g., `Stream.iterate`).
    *   **Consumable:** Streams can typically be traversed only once.
*   **Intermediate Operations (return a new Stream):**
    *   **`filter(Predicate<T> predicate)`:** Selects elements matching the predicate.
    *   **`map(Function<T, R> mapper)`:** Transforms each element using the mapper function.
    *   **`flatMap(Function<T, Stream<R>> mapper)`:** Transforms each element into a stream of other objects, then flattens these streams into a single stream. Useful when each element can map to zero, one, or multiple elements in the resulting stream.
    *   **`distinct()`:** Returns a stream with unique elements (based on `equals()`).
    *   **`sorted()` / `sorted(Comparator<T> comparator)`:** Sorts elements.
    *   **`peek(Consumer<T> action)`:** Performs an action on each element as it passes through the stream (primarily for debugging).
    *   **`limit(long maxSize)`:** Truncates the stream to be no longer than `maxSize`.
    *   **`skip(long n)`:** Discards the first `n` elements.
*   **Terminal Operations (produce a result or side-effect):**
    *   **`forEach(Consumer<T> action)`:** Performs an action for each element.
    *   **`collect(Collector<T, A, R> collector)`:** Performs a mutable reduction operation. Very versatile.
        *   Common collectors: `Collectors.toList()`, `Collectors.toSet()`, `Collectors.toMap()`, `Collectors.joining()`, `Collectors.groupingBy()`, `Collectors.counting()`.
    *   **`reduce(T identity, BinaryOperator<T> accumulator)` / `reduce(BinaryOperator<T> accumulator)`:** Combines stream elements into a single summary result.
    *   **`count()`:** Returns the number of elements in the stream.
    *   **`anyMatch(Predicate<T> predicate)`:** Returns `true` if any element matches the predicate.
    *   **`allMatch(Predicate<T> predicate)`:** Returns `true` if all elements match the predicate.
    *   **`noneMatch(Predicate<T> predicate)`:** Returns `true` if no elements match the predicate.
    *   **`findFirst()`:** Returns an `Optional` describing the first element, or an empty `Optional` if the stream is empty.
    *   **`findAny()`:** Returns an `Optional` describing some element of the stream, or an empty `Optional`. Particularly useful for parallel streams.
    *   `min(Comparator<T> comparator)` / `max(Comparator<T> comparator)`: Returns min/max element.
*   **Parallel Streams:**
    *   Created using `collection.parallelStream()` or `stream.parallel()`.
    *   Automatically partitions the stream and processes elements in parallel using the common ForkJoinPool.
    *   Not always faster; overhead of parallelism can outweigh benefits for small datasets or certain operations.
    *   Requires careful consideration of thread-safety if operations have side effects or use shared mutable state. Stateless or safely concurrent operations are ideal.
*   **Primitive Streams:** Specialized streams for `int`, `long`, `double` to avoid boxing/unboxing overhead.
    *   `IntStream`, `LongStream`, `DoubleStream`.
    *   Provide specialized operations like `sum()`, `average()`, `summaryStatistics()`.
    *   Mapping between object streams and primitive streams: `mapToInt()`, `mapToLong()`, `mapToDouble()`, and `boxed()` (from primitive stream to object stream).

> **TODO:** Provide an example from the TOPS project where you used the Stream API for processing Kafka messages or transforming collections of financial data. Discuss any performance considerations you encountered, especially with parallel streams.
**Sample Answer/Experience:**
"In the TOPS project, we extensively used the Stream API for processing collections of financial transactions read from Kafka messages. For instance, after deserializing a batch of trades, we often needed to filter them based on certain criteria (e.g., specific instrument types, transaction volume), transform them into a different DTO format, and then group them by client ID.
```java
// Assume 'rawTrades' is a List<RawTradeData> from Kafka
List<ProcessedTrade> processedTrades = rawTrades.stream()
    .filter(trade -> "EQUITY".equals(trade.getInstrumentType()))
    .filter(trade -> trade.getVolume() > 1000)
    .map(trade -> new ProcessedTrade(trade.getTradeId(), trade.getClientId(), trade.getNetValue()))
    .collect(Collectors.toList());

Map<String, List<ProcessedTrade>> tradesByClient = processedTrades.stream()
    .collect(Collectors.groupingBy(ProcessedTrade::getClientId));
```
This declarative style made the code much more readable than traditional loops and conditional statements.
**Performance Considerations:**
For very large collections, we experimented with `parallelStream()`. In one instance, we had a complex calculation that needed to be applied to each trade in a large batch.
```java
// Potentially performance-intensive calculation
results = trades.parallelStream()
                .map(this::performComplexCalculation) // Assumed to be CPU-bound and thread-safe
                .collect(Collectors.toList());
```
While `parallelStream()` can offer speedups for CPU-bound tasks on multi-core processors, we found it crucial to:
1.  **Ensure thread safety:** The `performComplexCalculation` method and any state it accessed had to be thread-safe.
2.  **Benchmark:** We always benchmarked `parallelStream()` against the sequential version. For smaller datasets or operations with significant I/O, the overhead of the ForkJoinPool and stream splitting sometimes made it slower.
3.  **Avoid shared mutable state:** Modifying shared collections or objects from within a parallel stream is dangerous and can lead to incorrect results or race conditions.
4.  **Consider the source:** If the stream source doesn't split well (e.g., some custom iterators), parallelism might not be effective. `ArrayList` splits very well.
In the TOPS project, for most of our per-message Kafka processing, the data size was small enough that sequential streams were sufficient and avoided any parallelism overhead. We reserved parallel streams for specific, identified batch processing bottlenecks after careful profiling."

### Default and Static methods in Interfaces
*   **Purpose:** Allow interfaces to evolve without breaking existing implementing classes.
*   **Default Methods:**
    *   Provide a default implementation for a method directly in the interface.
    *   Marked with the `default` keyword.
    *   If an implementing class does not override the default method, it inherits the default implementation.
    *   Use Cases: Adding new functionality to existing interfaces (e.g., `forEach` in `Iterable`).
*   **Static Methods:**
    *   Methods that are part of the interface but not tied to instances of implementing classes.
    *   Called using `InterfaceName.staticMethod()`.
    *   Use Cases: Utility methods related to the interface (e.g., factory methods for comparators in `Comparator` interface).
*   **Rules for Inheritance:**
    *   A class implementing an interface inherits default methods.
    *   If a class implements multiple interfaces that have default methods with the same signature, the class must explicitly override the method to resolve the conflict (or the superclass implementation takes precedence).
    *   A class implementation always takes precedence over an interface default implementation.
    *   Static methods are not inherited by implementing classes or sub-interfaces.

> **TODO:** Reflect on how default methods in interfaces could have helped you evolve service contracts in your Spring Boot microservices (Intralinks/TOPS) without forcing immediate changes on all implementing classes.
**Sample Answer/Experience:**
"In our microservice architecture at Intralinks, we often had shared library modules defining service interfaces that multiple microservices implemented. For example, we might have an `AuditService` interface:
```java
public interface AuditService {
    void recordEvent(String eventType, Map<String, Object> details);
    // Other methods...
}
```
If we later needed to add a new, non-critical method, say `recordEventWithPriority(String eventType, Map<String, Object> details, Priority priority)`, adding it directly would break all existing microservices implementing `AuditService`.
Default methods provide a great solution here:
```java
public interface AuditService {
    void recordEvent(String eventType, Map<String, Object> details);

    default void recordEventWithPriority(String eventType, Map<String, Object> details, Priority priority) {
        // Default implementation might just call the old method, ignoring priority,
        // or log a warning that priority is not fully handled by this implementation.
        System.out.println("WARN: Priority ignored by this AuditService implementation for event: " + eventType);
        recordEvent(eventType, details);
    }
    // Other methods...
}
```
This way, existing services would continue to compile and run, inheriting the default behavior. New services, or existing ones when they are updated, could then choose to provide a specific implementation for `recordEventWithPriority`. This allowed us to evolve shared interfaces in a backward-compatible manner, which is crucial in a distributed system where services are updated independently."

### `Optional` Class
*   **Purpose:** A container object that may or may not contain a non-null value. It is used to represent optional values instead of returning `null`, thereby reducing the likelihood of `NullPointerExceptions` and making APIs clearer about optional return values.
*   **Creating Optionals:**
    *   `Optional.of(value)`: Creates an `Optional` with a non-null value. Throws `NullPointerException` if `value` is null.
    *   `Optional.ofNullable(value)`: Creates an `Optional` that may hold a null value. If `value` is null, it returns an empty `Optional`.
    *   `Optional.empty()`: Creates an empty `Optional`.
*   **Methods:**
    *   `isPresent()`: Returns `true` if a value is present, `false` otherwise.
    *   `isEmpty()` (Java 11+): Returns `true` if no value is present (opposite of `isPresent()`).
    *   `get()`: Returns the value if present, otherwise throws `NoSuchElementException`. (Use with caution, typically after `isPresent()` or in situations where presence is guaranteed).
    *   `orElse(T other)`: Returns the value if present, otherwise returns `other`.
    *   `orElseGet(Supplier<? extends T> otherSupplier)`: Returns the value if present, otherwise invokes `otherSupplier` and returns the result of that invocation.
    *   `orElseThrow(Supplier<? extends X> exceptionSupplier)`: Returns the value if present, otherwise throws an exception created by the `exceptionSupplier`.
    *   `ifPresent(Consumer<? super T> consumer)`: If a value is present, invokes the specified consumer with the value, otherwise does nothing.
    *   `map(Function<? super T, ? extends R> mapper)`: If a value is present, applies the provided mapping function to it, and if the result is non-null, returns an `Optional` describing the result. Otherwise returns an empty `Optional`.
    *   `flatMap(Function<? super T, Optional<R>> mapper)`: If a value is present, applies the provided `Optional`-bearing mapping function to it, returns that result, otherwise returns an empty `Optional`. Useful for chaining operations that themselves return `Optional`.
    *   `filter(Predicate<? super T> predicate)`: If a value is present, and the value matches the given predicate, returns an `Optional` describing the value, otherwise returns an empty `Optional`.

> **TODO:** How did using `Optional` change the way you designed methods and handled potentially missing values in your Spring Boot REST controllers or services, for example, when fetching data from a database in the Herc Rentals AEM project?
**Sample Answer/Experience:**
"Using `Optional` significantly improved how we handled potentially missing values in our Spring Boot services, particularly with database interactions via Spring Data JPA, and in the data passed to AEM components at Herc Rentals.
For example, a Spring Data repository method like `findById(String id)` returns `Optional<User>`. In our service layer, instead of:
```java
// Old way
User user = userRepository.findById(id); // Could be null
if (user != null) {
    // do something
} else {
    // handle not found
}
```
We would write:
```java
// Using Optional
Optional<User> userOpt = userRepository.findById(id);

// Example 1: Throw custom exception if not found
User user = userOpt.orElseThrow(() -> new ResourceNotFoundException("User with id " + id + " not found"));
// process user

// Example 2: Return a DTO, or a default/empty DTO if not found
UserDTO dto = userOpt.map(this::convertToUserDTO) // convertToUserDTO is a method reference
                     .orElse(UserDTO.EMPTY_USER_DTO);

// Example 3: Perform action only if present
userOpt.ifPresent(u -> System.out.println("User found: " + u.getName()));
```
In REST controllers, if a service layer method returned an `Optional<ResourceDTO>`, we could elegantly handle it:
```java
@GetMapping("/{id}")
public ResponseEntity<ResourceDTO> getResource(@PathVariable String id) {
    return resourceService.findById(id) // service returns Optional<ResourceDTO>
        .map(ResponseEntity::ok) // If present, 200 OK with body
        .orElse(ResponseEntity.notFound().build()); // If empty, 404 Not Found
}
```
This made the code more expressive about the possibility of absent values and forced us to actively consider the 'not found' case, reducing `NullPointerExceptions`. The `map` and `flatMap` methods on `Optional` also allowed for cleaner chaining of operations on potentially absent values without nested null checks. It made the intent of the code much clearer."

### New Date and Time API (`java.time`)
*   **Overview:** A comprehensive, immutable, and thread-safe API for handling dates, times, durations, and periods. Replaces the old, problematic `java.util.Date` and `java.util.Calendar` classes. Based on ISO-8601 standard.
*   **Key Classes:**
    *   **`LocalDate`:** Represents a date (year, month, day) without time-of-day and time-zone. (e.g., 2023-12-25).
    *   **`LocalTime`:** Represents a time (hour, minute, second, nanosecond) without date and time-zone. (e.g., 10:15:30).
    *   **`LocalDateTime`:** Represents a date-time without a time-zone. (e.g., 2023-12-25T10:15:30).
    *   **`ZonedDateTime`:** Represents a date-time with a specific time-zone. (e.g., 2023-12-25T10:15:30+01:00[Europe/Paris]). Handles daylight saving time changes.
    *   **`Instant`:** Represents an instantaneous point on the timeline (nanosecond precision), typically from the epoch (1970-01-01T00:00:00Z). Machine-friendly.
    *   **`Duration`:** Represents a time-based amount of time (seconds, nanoseconds). (e.g., "5 hours", "30.5 seconds").
    *   **`Period`:** Represents a date-based amount of time (years, months, days). (e.g., "2 years, 3 months, 5 days").
    *   **`DateTimeFormatter`:** For parsing and formatting date-time objects. Provides predefined formatters (e.g., `ISO_DATE_TIME`) and allows defining custom patterns.
*   **Benefits over old API:**
    *   **Immutability:** All core classes are immutable, making them thread-safe.
    *   **Clarity & Usability:** Well-defined classes for specific concepts (date, time, date-time, zoned). Methods are intuitive (e.g., `plusDays`, `withMonth`).
    *   **Better Time-Zone Handling:** Clear distinction between local and zoned date-times. `ZoneId` and `ZoneOffset` for time zones.
    *   **Fluent API:** Methods often return new instances, allowing for chained calls.
    *   **Performance:** Generally good performance.

### `CompletableFuture`
*   **Introduction to Asynchronous Programming:** `CompletableFuture` (in `java.util.concurrent`) is an implementation of the `Future` and `CompletionStage` interfaces. It provides a powerful way to write asynchronous, non-blocking code. It allows you to specify actions to be performed when an asynchronous computation completes, without manually managing threads.
*   **Basic Usage:**
    *   **`supplyAsync(Supplier<U> supplier)` / `supplyAsync(Supplier<U> supplier, Executor executor)`:** Starts an asynchronous computation. Returns a `CompletableFuture` that will be completed with the value obtained from the supplier.
    *   **`runAsync(Runnable runnable)` / `runAsync(Runnable runnable, Executor executor)`:** Starts an asynchronous computation that doesn't return a value.
    *   **`thenApply(Function<? super T,? extends U> fn)`:** Transforms the result of a completed `CompletableFuture`. Takes a function that is applied to the result when it becomes available. Returns a new `CompletableFuture`.
    *   **`thenAccept(Consumer<? super T> action)`:** Performs an action with the result of a completed `CompletableFuture`.
    *   **`thenRun(Runnable action)`:** Executes a `Runnable` after the completion of the `CompletableFuture`.
    *   **`exceptionally(Function<Throwable,? extends T> fn)`:** Handles exceptions that occur during the computation. If the original future completes exceptionally, this function is applied to the throwable.
    *   **`handle(BiFunction<? super T, Throwable, ? extends U> fn)`:** Handles both successful completion and exceptional completion.
*   **Combining Futures:**
    *   **`thenCompose(Function<? super T, ? extends CompletionStage<U>> fn)`:** Chains two `CompletableFuture`s sequentially. The function `fn` takes the result of the first future and returns another `CompletionStage`. Useful when the next async operation depends on the result of the first. (Similar to `flatMap` for `Optional`/`Stream`).
    *   **`thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)`:** Combines the results of two independent `CompletableFuture`s when both are complete.
    *   **`allOf(CompletableFuture<?>... cfs)`:** Returns a `CompletableFuture<Void>` that completes when all of the given `CompletableFuture`s complete.
    *   **`anyOf(CompletableFuture<?>... cfs)`:** Returns a `CompletableFuture<Object>` that completes when any of the given `CompletableFuture`s complete, with the same result.

> **TODO:** Describe a scenario in your Intralinks/BOFA project where you used `CompletableFuture` to orchestrate multiple asynchronous calls to downstream services or AWS services (e.g., fetching data from S3, then calling another microservice).
**Sample Answer/Experience:**
"At Intralinks, we often had workflows that required orchestrating multiple asynchronous operations. For example, a request might come in to process a user document. This could involve:
1.  Fetching user metadata from a User Service (async call).
2.  Fetching the document from AWS S3 (async call).
3.  Calling a Document Processing Service with the document and user metadata (async call).
4.  Notifying another service upon completion (async call).

`CompletableFuture` was invaluable here to avoid callback hell and manage the flow.
```java
// Assume userService, s3Service, docProcessingService, notificationService all return CompletableFuture

public CompletableFuture<ProcessingStatus> processUserDocument(String userId, String documentKey) {
    CompletableFuture<UserMetadata> userFuture = userService.getMetadata(userId);
    CompletableFuture<S3Object> documentFuture = s3Service.getDocument(documentKey);

    return userFuture.thenCombine(documentFuture, (metadata, s3Doc) -> {
        // Both user metadata and document are fetched
        // Prepare input for document processing service
        ProcessingInput input = new ProcessingInput(metadata, s3Doc.contentAsBytes());
        return input;
    }).thenCompose(processingInput -> {
        // Call the document processing service
        return docProcessingService.process(processingInput);
    }).thenCompose(processingResult -> {
        // After processing, notify another service
        return notificationService.sendNotification(userId, documentKey, processingResult.getStatus());
    }).thenApply(notificationSent -> {
        // Final status based on notification success (or processingResult directly)
        return notificationSent ? ProcessingStatus.SUCCESSFULLY_NOTIFIED : ProcessingStatus.PROCESSING_DONE_NOTIFY_FAILED;
    }).exceptionally(ex -> {
        logger.error("Error in document processing flow for user {} doc {}: {}", userId, documentKey, ex.getMessage());
        return ProcessingStatus.FAILED;
    });
}
```
In this example:
*   `userService.getMetadata()` and `s3Service.getDocument()` are called concurrently.
*   `thenCombine` waits for both to complete and combines their results.
*   `thenCompose` is used to chain subsequent asynchronous calls (`docProcessingService.process` and `notificationService.sendNotification`) because these calls themselves return `CompletableFuture`.
*   `thenApply` is used for a synchronous transformation at the end.
*   `exceptionally` provides a centralized way to handle any failure in the preceding chain.

This approach made the complex asynchronous workflow much more manageable, readable, and easier to reason about compared to nested callbacks. We also often provided custom `Executor`s to `supplyAsync` or `thenApplyAsync` if the tasks were particularly blocking or long-running to avoid starving the common ForkJoinPool."

### Nashorn JavaScript Engine
*   Introduced in Java 8 as a replacement for the older Rhino JavaScript engine.
*   Allowed embedding JavaScript code within Java applications and running it on the JVM.
*   Provided better performance and compatibility with modern JavaScript features compared to Rhino.
*   **Deprecated in Java 11 and removed in Java 15.** GraalVM's JavaScript engine is now the recommended alternative for running JavaScript on the JVM.

### PermGen removal and Metaspace
*   **PermGen (Permanent Generation):** In older JVM versions, a part of the heap used to store class metadata, interned strings, and static variables. It had a fixed maximum size, often leading to `java.lang.OutOfMemoryError: PermGen space` if many classes were loaded.
*   **Metaspace (Java 8+):** PermGen was replaced by Metaspace.
    *   Metaspace is allocated from native memory, not the Java heap.
    *   By default, Metaspace size is auto-tuned and can grow as needed, limited by available native memory. This reduces the occurrence of OOMEs related to class metadata.
    *   Maximum Metaspace size can be configured using `-XX:MaxMetaspaceSize`.
    *   Interned strings were moved from PermGen to the main Java heap in Java 7. Static variables are also on the heap.

## Java 9-11 Features (Selected)

### Java Platform Module System (JPMS - Java 9)
*   **Core Concepts:**
    *   **Module:** A uniquely named, reusable group of related packages, resources, and a module descriptor.
    *   **Module Descriptor (`module-info.java`):** A file at the root of the module's source code.
        *   `module com.example.mymodule { ... }`
        *   **`requires <moduleName>`:** Declares a dependency on another module.
        *   **`exports <packageName>`:** Makes types in `packageName` accessible to other modules.
        *   **`opens <packageName>`:** Makes types in `packageName` accessible at runtime via reflection.
        *   `uses <serviceInterface>` / `provides <serviceInterface> with <implementationClass>`: For service discovery.
*   **Benefits:**
    *   **Strong Encapsulation:** Hides internal implementation details of a module by default.
    *   **Reliable Configuration:** Explicit dependencies, avoiding "classpath hell."
    *   **Scalability:** Enables creation of smaller custom JREs (using `jlink`).
    *   Improved Security and Maintainability.
*   (This is a very brief overview; JPMS is a large and complex topic).

> **TODO:** While JPMS is a large topic, did you encounter any challenges or benefits related to modularity when working with Java 9+ features in your microservices, particularly concerning dependencies or classpath issues when deploying to AWS?
**Sample Answer/Experience:**
"While many of our core microservices at Intralinks and TOPS were still predominantly Java 8 based during my tenure for broader compatibility, we started exploring Java 11 for newer, smaller services. The most immediate benefit we saw with Java 9+ was the potential for creating smaller deployment artifacts using `jlink` by packaging a custom JRE with only the necessary JDK modules. This was particularly attractive for AWS Lambda or containerized deployments on ECS/EKS, as smaller images mean faster startup times and reduced disk footprint.
However, fully adopting JPMS with `module-info.java` for application-level modules presented challenges:
1.  **Library Support:** Not all third-party libraries we used were fully modularized initially (i.e., provided `module-info.java` with proper exports). This meant dealing with 'automatic modules' which have less strong encapsulation.
2.  **Reflection & `opens`:** Spring Boot and other frameworks like Hibernate heavily use reflection. We had to be careful to `open` the necessary packages in our `module-info.java` files to allow these frameworks to access our beans and entities, which felt a bit like working against strong encapsulation at times.
3.  **Build Tool Complexity:** Integrating JPMS with Maven or Gradle required updating build configurations, and there was a learning curve for the team.
The primary benefit we aimed for was more reliable configuration and avoiding classpath hell, especially as the number of microservices and shared internal libraries grew. While we didn't fully convert all existing applications to be modular, for new self-contained services, using `jlink` with explicit JDK module dependencies (e.g., `requires java.sql, java.net.http;`) was a clear win for deployment size. The strong encapsulation, once properly configured, also gave us more confidence in preventing unintended internal API usage between our own modules/libraries."

### Local-Variable Type Inference (`var` keyword - Java 10)
*   **Usage:** Allows the compiler to infer the type of a local variable from the type of its initializer.
    *   `var list = new ArrayList<String>();` (inferred as `ArrayList<String>`)
    *   `var stream = list.stream();` (inferred as `Stream<String>`)
*   **Scope:** Can be used for local variables in methods, constructors, initializer blocks, `for` loops, and `try-with-resources`. Cannot be used for fields, method parameters, or method return types.
*   **Limitations:**
    *   Initializer must be present.
    *   Cannot be initialized to `null` (type unknown).
    *   Cannot be used with lambda expressions without an explicit target type on the left.
*   **Benefits for Readability:**
    *   Reduces boilerplate when variable types are obvious or very long (e.g., generic types).
    *   Can improve readability by aligning variable names.
    *   However, should be used judiciously; explicit types can sometimes be clearer, especially if the initializer's type isn't immediately obvious.

> **TODO:** How did the `var` keyword (Java 10) influence your coding style? Provide an example from your data processing work (TOPS) or microservice development where `var` improved code readability, and any situations where you chose not to use it.
**Sample Answer/Experience:**
"When we started using Java 11 in some of our newer services, the `var` keyword for local variable type inference was a welcome addition for reducing verbosity, especially with complex generic types.
For example, in the TOPS project, when processing nested data structures or dealing with results from complex Stream API collectors, type names could get quite long:
```java
// Before var
Map<String, List<Map<TransactionType, BigDecimal>>> clientTransactionAggregates = processComplexData();

// With var
var clientTransactionAggregates = processComplexData();
```
In this case, `var` definitely improved readability by making the line shorter and focusing on the variable name and its purpose, assuming `processComplexData()` has a clear return type.
Another good use case was in `try-with-resources` or loops:
```java
// Before var
try (InputStream fis = new FileInputStream("file.txt");
     BufferedInputStream bis = new BufferedInputStream(fis);
     DataInputStream dis = new DataInputStream(bis)) {
    // ...
}

// With var
try (var fis = new FileInputStream("file.txt");
     var bis = new BufferedInputStream(fis);
     var dis = new DataInputStream(bis)) {
    // ...
}
```
**Situations where I chose NOT to use `var`:**
1.  **Readability Suffers:** If the initializer's type wasn't immediately obvious and the explicit type provided better context for the reader. For example, if a factory method's name didn't clearly indicate the returned type: `var data = someFactory.createInstance();` (What is `data`? `SpecificData` or `BaseData`?). Here, `SpecificData data = someFactory.createInstance();` might be better.
2.  **Polymorphism:** When I specifically wanted to program to an interface and make that clear, e.g., `List<String> names = new ArrayList<>();` rather than `var names = new ArrayList<String>();` (which infers `ArrayList<String>`). Using the interface type is often better for maintainability.
3.  **Primitive Types or Simple Types:** For `int`, `String`, `boolean`, etc., `var` doesn't add much value and can sometimes look a bit odd: `var count = 10;` vs `int count = 10;`.
We adopted a style guideline to use `var` when it genuinely improves readability by removing redundant type information, but to stick to explicit types when clarity or programming to interfaces was more important."

### New `String` methods (Java 11)
*   **`isBlank()`:** Returns `true` if the string is empty or contains only white space characters.
*   **`lines()`:** Returns a `Stream<String>` of lines extracted from the string, separated by line terminators.
*   **`strip()`:** Returns a string whose value is this string, with all leading and trailing white space removed. (Unicode-aware, unlike `trim()`).
*   **`stripLeading()`:** Removes leading white space.
*   **`stripTrailing()`:** Removes trailing white space.
*   **`repeat(int count)`:** Returns a string whose value is the concatenation of this string repeated `count` times.

### New `Collection` factory methods (Java 9)
*   Convenience factory methods for creating immutable collections (lists, sets, maps).
*   **`List.of(e1, e2, ...)`**
*   **`Set.of(e1, e2, ...)`** (throws `IllegalArgumentException` for duplicates)
*   **`Map.of(k1, v1, k2, v2, ...)`** (up to 10 key-value pairs)
*   **`Map.ofEntries(Map.Entry<K,V>... entries)`** (for more than 10 entries, using `Map.entry(k, v)`)
*   **Characteristics:**
    *   **Immutable:** Attempting to modify them (add, remove, set) results in `UnsupportedOperationException`.
    *   Null elements/keys/values are generally not allowed (throw `NullPointerException`).
    *   Order of elements is preserved for `List.of` and `Set.of` (iteration order). `Map.of` iteration order is not strictly guaranteed but is often based on insertion.

> **TODO:** How did the static factory methods for Collections (e.g., `List.of()`, `Set.of()`) introduced in Java 9 simplify your code, perhaps when creating test data or defining small, fixed collections in your Spring Boot or AEM projects?
**Sample Answer/Experience:**
"The static factory methods for collections like `List.of()`, `Set.of()`, and `Map.of()` were a small but very welcome improvement for code conciseness, especially for creating small, immutable collections.
Previously, to create a small, unmodifiable list, we might do:
```java
List<String> names = Collections.unmodifiableList(Arrays.asList("Alice", "Bob", "Charlie"));
```
With Java 9+, this became much cleaner:
```java
List<String> names = List.of("Alice", "Bob", "Charlie");
```
**Use Cases:**
1.  **Test Data:** In unit tests, when setting up mock objects or expected results, these methods were perfect for creating small, fixed collections:
    `when(mockService.getAllowedRoles()).thenReturn(Set.of("ADMIN", "USER"));`
2.  **Configuration Constants:** For defining small sets of constant values within a class:
    `private static final Set<String> SUPPORTED_CURRENCIES = Set.of("USD", "EUR", "GBP");`
3.  **Returning Immutable Copies:** When a method needed to return an immutable view of an internal collection, though careful consideration is needed if the internal collection changes. More often, it was for constructing new immutable collections.
4.  **Map Initialization:** `Map.of("key1", "value1", "key2", "value2")` is much nicer than the older ways of initializing small maps.

The key characteristics we had to remember were their immutability (which is often a good thing, promoting safer code) and that they don't allow `null` elements/keys/values. This actually helped catch potential issues earlier in some cases. For AEM components at Herc Rentals, we might use them to define a fixed set of allowed child component resource types, or for default configuration maps."

### New `Optional` methods (Java 9, 11)
*   **Java 9:**
    *   **`ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`:** If a value is present, performs the given action with the value, otherwise performs the given empty-based action.
    *   **`or(Supplier<? extends Optional<? extends T>> supplier)`:** If a value is present, returns an `Optional` describing the value, otherwise returns an `Optional` produced by the supplying function.
    *   **`stream()`:** If a value is present, returns a sequential `Stream` containing only that value, otherwise returns an empty `Stream`.
*   **Java 11:**
    *   **`isEmpty()`:** Returns `true` if no value is present, `false` otherwise. (Already mentioned under Java 8 `Optional` as it's commonly used).

### `HttpClient` (Standardized in Java 11)
*   **Overview:** A new, modern HTTP client API (`java.net.http.HttpClient`) to replace the legacy `HttpURLConnection`.
*   Supports HTTP/1.1, HTTP/2, and WebSockets.
*   **Asynchronous and Synchronous Capabilities:**
    *   `send(HttpRequest, BodyHandler)`: Synchronous. Blocks until the response is available.
    *   `sendAsync(HttpRequest, BodyHandler)`: Asynchronous. Returns a `CompletableFuture<HttpResponse<T>>`.
*   **Creating Requests:**
    *   `HttpRequest.newBuilder()`
    *   `.uri(URI.create("..."))`
    *   `.GET()`, `.POST(BodyPublishers.ofString("..."))`, `.PUT()`, `.DELETE()`
    *   `.header("Content-Type", "application/json")`
    *   `.timeout(Duration.ofSeconds(10))`
*   **Handling Responses:**
    *   `HttpResponse.BodyHandlers` provide common handlers:
        *   `BodyHandlers.ofString()`: Handles response body as a String.
        *   `BodyHandlers.ofByteArray()`: Handles response body as byte array.
        *   `BodyHandlers.ofInputStream()`: Handles response body as an InputStream.
        *   `BodyHandlers.discarding()`: Discards the response body.
    *   `HttpResponse` object contains status code, headers, and body.

> **TODO:** Did you get a chance to use the new Java 11 `HttpClient` in any of your projects, perhaps for inter-service communication in a non-Spring context or for specific needs in Vert.x at Intralinks? How did it compare to older clients like Apache HttpClient or Spring's `RestTemplate`?
**Sample Answer/Experience:**
"While many of our existing microservices at Intralinks and TOPS relied on Spring's `RestTemplate` or `WebClient` (which is reactive and non-blocking, similar in spirit to Vert.x's client), for some newer, lightweight utility services or standalone Java 11+ applications, we did start using the new standard `java.net.http.HttpClient`.

**Advantages we noted:**
1.  **Modern API:** It has a fluent builder API and natively supports asynchronous operations with `CompletableFuture`, which integrates very well with other Java 8+ asynchronous code. This was a big plus compared to the older `HttpURLConnection`.
2.  **HTTP/2 Support:** Built-in HTTP/2 support without needing extra dependencies was beneficial for performance with services that supported it.
3.  **No External Dependencies:** For smaller utilities or lambdas where we wanted to minimize dependencies, using a standard JDK HTTP client was appealing.

**Comparison:**
*   **vs. `HttpURLConnection`:** Vastly superior. `HttpURLConnection` is clunky, hard to use, and lacks modern features.
*   **vs. Apache HttpClient:** Apache HttpClient is very mature and feature-rich, offering fine-grained control over many aspects of HTTP communication. The Java 11 HttpClient is simpler and might not have all the advanced configuration options of Apache's, but for most common use cases, it's sufficient and easier to use.
*   **vs. Spring's `RestTemplate`/`WebClient`:**
    *   `RestTemplate` is synchronous and widely used in Spring MVC. The Java 11 client's synchronous API is comparable but `RestTemplate` has better integration with Spring's ecosystem (message converters, error handlers).
    *   `WebClient` is Spring's reactive HTTP client. If we were already in a reactive Spring Boot (WebFlux) application, `WebClient` would be the natural choice. The Java 11 HttpClient's async capabilities via `CompletableFuture` are good, but `WebClient` provides a richer reactive programming model with Project Reactor.
    *   In a Vert.x application at Intralinks, we'd primarily use Vert.x's own `WebClient` because it's deeply integrated into the Vert.x event loop model and provides the best performance in that specific non-blocking environment. We wouldn't typically introduce the Java 11 HttpClient there unless there was a very specific reason (e.g., a blocking worker verticle needing a simple HTTP call without full Vert.x client setup).

So, for new, non-Spring-heavy Java 11+ projects or utilities needing straightforward HTTP calls (sync or async), the standard `HttpClient` became a good option. For existing Spring projects, we usually stuck with `RestTemplate` or `WebClient` for their ecosystem benefits."

### Flight Recorder (JFR) and Mission Control
*   **Java Flight Recorder (JFR):** A low-overhead data collection framework for troubleshooting Java applications and the JVM. Collects events (diagnostic and profiling data) about JVM internals and application execution.
*   **JDK Mission Control (JMC):** A tool for analyzing data collected by JFR. Provides graphical views of JVM performance, memory usage, thread activity, etc.
*   Previously commercial features in Oracle JDK, open-sourced and included in OpenJDK starting with Java 11.
*   Extremely useful for diagnosing performance issues, memory leaks, and other runtime problems in production environments with minimal performance impact.

### Single-File Source-Code Execution (Java 11)
*   Allows executing a single Java source file directly using the `java` launcher without explicit compilation (`javac`).
*   Syntax: `java MyProgram.java arg1 arg2 ...`
*   The source file is compiled in memory and then executed.
*   Useful for small utility programs, scripts, or learning Java.
*   The file must contain a `main` method. All classes defined in the file are compiled.
*   Limitations: Cannot use external dependencies not part of JDK unless specified via `--class-path` or other means. More complex builds still require `javac` and build tools.
